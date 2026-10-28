# Using Apache Kafka for Communication Between Two Applications

While working on one of our projects, we had a requirement where another team was responsible for maintaining a particular status flag, and our application needed to keep that flag updated against the corresponding records in our database.

Although the requirement sounded simple, both teams had their own applications and were working independently. We needed a reliable way to receive these updates without creating too much dependency between the systems.

One option was to expose a REST API and ask the other team to call our service whenever the flag changed. This was something we were already familiar with, and it would have been relatively straightforward to implement.

However, after discussing the requirement, we decided to use Apache Kafka for communication between the applications.

## Explaining the Decision to the Product Owner

During our discussions, I explained the difference between REST API communication and Kafka in simple terms.

With a REST API, whenever the other team wanted to update a flag, their application would need to contact our application directly. If our service was unavailable at that moment, they would need to handle the failure and retry the request.

With Kafka, the other team could publish an update as a message, and our application could consume that message and update the corresponding record in our database.

I explained it using a simple example.

Imagine two departments working in the same office. Instead of one department calling the other every time something changes, they place the update in a shared message queue. The receiving department picks it up and processes it when ready.

Kafka works on a similar principle, although technically it stores messages in topics rather than operating as a traditional queue.

This helped explain why the two applications did not need to be available at exactly the same time to exchange updates.

## The Trade-off

I also explained that Kafka was not necessarily the simplest solution.

A REST API would have been easier to implement and debug, especially for a small number of updates. It would also provide an immediate response indicating whether the request was successful.

Kafka introduced additional things that we needed to consider, such as message consumption, retries, duplicate messages and monitoring.

Another important difference was that the database update would not necessarily happen immediately after the other team published the message.

There could be a small delay between publishing the event and our application processing it.

However, our requirement did not need an immediate synchronous response. What mattered more was that updates could be processed reliably without requiring both applications to be available at the same time.

That was the main reason we preferred Kafka.

## How We Implemented It

The other team was responsible for publishing messages containing the record identifier and the updated flag value.

On our side, we developed a Kafka consumer using Spring Boot.

Whenever a message was received, our application would identify the corresponding record and update the flag in our database.

The basic flow was:

**Other Team → Kafka Topic → Our Spring Boot Consumer → Database Update**

One important consideration was ensuring that the correct record was updated. We used the identifier provided in the message to locate the record before applying the new flag value.

We also needed to think about scenarios where a message might be received more than once or where a database update could fail.

These were important considerations because Kafka consumers may process messages again after certain failures, so the database operation needed to handle repeated processing safely.

## What I Learned

The interesting part of this experience was not just implementing Kafka. It was explaining why we were introducing another technology when a REST API could also solve the problem.

From a technical perspective, Kafka gave us asynchronous communication and reduced the direct dependency between the two applications.

From the product owner's perspective, the benefit was easier to understand: one team's application could send an update without having to wait for the other team's application to process it immediately.

At the same time, I made sure the trade-off was clear. We were accepting additional implementation and operational complexity in exchange for more flexible communication between the systems.

This experience helped me understand that when explaining technical decisions to non-technical stakeholders, it is better to focus on the business impact rather than the technology itself.

The goal was not to convince the product owner that Kafka was a better technology than REST. It was to explain why Kafka was a better fit for this particular requirement.
