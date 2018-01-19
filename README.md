# AWS-Notes

Simple Queue Service 

Amazon SQS is a web service that gives you access to a message queue that can be used to store messages while waiting for a computer to process them. (Sounds a lot like Kafka!!) 
* Distributed queue system, one of the first AWS services launched 
* Queue: temporary repo for messages that are awaiting processing 
* Enables web service applications to quickly and reliably queue messages that one component in the application generates to be consumed by another component 
* Can decouple the components of an application so they run independently 
* Eases message management between components 
* Any component of a distributed application can store messages in a fail-safe queue >> any component can later retrieve the messages programmatically using the SQS API 
* Messages <= 256 KB of text in any format 
    * Billed at 64KB chunks (old size), so we get 4 chunks per message size  
* Example: can set up EC2 instances to poll queue to look for job, process it, send it back to queue 
* Queue acts as a buffer between the component producing and saving data, and the component retrieving data for processing 
    * Resolves issues that arise if the producer is producing work faster than the consumer can process it or if the producer or consumer are only intermittently connected to the network 
    * i.e. handles autoscaling or failover 
* Ensures delivery of each message at least once 
* Supports multiple readers and writers interacting with the same queue 
* Single queue can be used simultaneously by many distributed application components, with no need for those components to coordinate with each other to share the queue 
    * Hence, great way of scaling out applications 
* Engineered to always be available and deliver messages: a tradeoff is that no guarantee of “FIRST IN, FIRST OUT/ FIFO” delivery of messages 
    * For many distributed applications, each message can stand on its own, and as long as all messages are delivered, the order is not important 
    * If your system requires that order be preserved, you can place sequencing information in each message, so that you can reorder the messages when the queue returns them 
* SQS always pulls (polls for) messages from the queue; the queue never pushes out 
*  Consumer: 
    * 1. Asynchronously pulls the task messages from the queue 
    * 2. Processing 
    * 3. Writes a task complete message to another queue 
    * 4. Deletes the original task message 
    * 5. Checks for more messages in the worker queue 
* General 
    * 1. Component 1 (our web server) sends Message A to the queue; there is a visibility timeout clock as to how long that message is going to be available in the queue (visibility timeout is 30s default, 12h max) 
        * ChangeMessageVisibility : action to specify a new timeout value 
            * Use if the visibility timeout for the queue is insufficient to fully process and delete that message 
            * SQS restarts the timeout period using the new value 
    * 2. Component 2 (our application server) retrieves Message A from the queue, and the visibility timeout period starts (timeout starts only when message picked up, which guarantees at least once delivery) 
    * 3. Component 2 processes Message A and then deletes it from the queue during the visibility timeout period 
        * If the timeout expires and component 2 does not finish, message will stay in queue and won’t be deleted, and another application server will poll queue and see timeout has expired and will then assume there has been a fail somewhere and will pull the message itself to process  
        * Only when the message is deleted from the message queue is when it’s finished 
* We can also configure autoscaling so if the queue is getting a certain size, it will start spinning up more application servers (and reducing them once once no longer needed) 
* Although most of the time each message will be delivered to your application exactly once, you should design your system so that processing a message more than once does not create any errors or inconsistencies 
* Pricing 
    * First 1M SQS requests per month are free
    * 50 cents per 1M requests thereafter per month 
    * Single request: 1-10 messages, up to maximum total payload of 256 KB
        * each 64KB chunk is billed as one request (i.e. the max payload will be billed as four requests)  
* SQS messages can be delivered multiple times and in any order 
    * if you need to prioritize particular messages, set up two different queues with different priorities 
* Traditional is SQS short polling
    * Returns immediately, even if the queue being polled is empty 
* SQS Long Polling 
    * Does not return a response until a message arrives in the queue OR the long poll times out 
    * Makes it east and inexpensive to return messages from your SQS queue as soon as they are available 
        * Away to avoid burning through CPU cycles which costs extra money 
    * Max long poll timeout : 20s 
￼
* SQS Fanning Out 
    * Create SNS topic with SNS, then create and subscribe multiple SQS queues to the SNS topic 
        * Whenever a message is sent to the SNS topic, the message will be fanned out to the SQS queues (message delivered to all subscribed queues)

￼
￼
￼Simple Notification Service 

SNS 
* Web service that makes it easy to set up, operate, and send notifications from the cloud 
* e.g. can have Cloud-Watch tied, autoscaling triggers, emails / texts 
* Scalable, flexible, cost-effective capability to publish messages from an application and immediately deliver them to subscribers of other applications 
    * Simple APIS 
    * Minimal upfront development 
    * No maintenance or management overhead 
    * Pay-as-you-go pricing 
* Publish-subscribe (pub-sub) paradigm 
    * Notifications being delivered to clients using a “push” mechanism 
        * Eliminates the need to periodically check or “poll” for new information and updates 
* Delivery Protocols 
    * Cloud notifications directly to mobile devices: Push notifications to Apple, Google, Fire OS, Windows devices, Android devices in China Baidu Cloud push 
    * SMS text message or email 
        * Email
        * Email-JSON
        * Note: Email subscriptions will expire after 3 days if not confirmed 
    * Amazon SQS (queues)
    * HTTP(S) endpoints 
    * Application 
    * Note: TTL is the time to live, and is the number of seconds since the message was published. When you use TTL, messages that remain undelivered for the specified time will expire. 
    * Note: Messages can be customized for each protocol 
* To prevent messages from being lost, all messages published to Amazon SNS are stored redundantly across multiple availability zones 
* Topics : access point of allowing recipients to dynamically subscribe for identical copies of the same notification 
    * What SNS uses to group multiple recipients 
    * Single topic can support delivery to multiple endpoint types (e.g. group together iOS, Android, and SMS recipients) 
    * When you publish once to a topic, SNS delivers appropriate formatted copies of your message to each subscriber 
* Benefits 
    * Instantaneous, push-based delivery (no polling) 
    * Simple APIs and easy integration with applications 
    * Flexible message delivery over multiple transport protocols 
    * Inexpensive, pay-as-you-go model with no upfront costs 
    * Web-based AWS management console offers the simplicity of a point-and-click interface 
* Pricing 
    * 50 cents per 1M SNS requests 
    * 6 cents per 100K notification deliveries over HTTP 
    * 75 cents per 100 notifications over SMS 
    * $2 per 100K notifications over email 
* Data format of SNS is JSON 


SNS v SQS 
* Both messaging services in AWS 
* SNS: push
* SQS: polls (pulls) 
* Fanning with SQS: SQS will send a notification to SNS and that will push it out to other SQS queues 



SWF 
* Web service that makes it easy to coordinate work across distributed application components 
* Enables applications for a range of use cases to be designed as a coordination of tasks
    * Media processing 
    * Web-application backends 
    * Business process workflows 
    * Analytics pipelines 
* Tasks represent invocations of various processing steps in an application
    * Step can be performed by 
        * Executable code 
        * Web service calls 
        * Human actions 
        * Scripts 
* SWF workers : programs that interact with SWF to get tasks, process received tasks, and return the results 
* SWF deciders : program that controls the coordination of tasks, i.e. their ordering, concurrency, and scheduling according the application logic 
* Workers and deciders
    * Can run on cloud infrastructure, eg. EC2 or on machines behind firewalls 
    * SWF brokers the interactions between workers and the decider 
        * Allows the decider to get consistent views into the progress of tasks and to initiate new tasks in an ongoing manner 
        * Stores tasks, assigns them to workers when they are ready, and monitors their progress 
        * Ensures that a task is assigned only once and never duplicated 
        * Maintains the application’s state durably so workers and deciders do not have to keep track of execution state and can run independently and scale quickly 
* SWF domain : isolate a set of types, executions, and task lists from other within the same account 
    * To which workflow, activity types, and the workflow execution itself are scoped 
    * Can register using the AWS management console OR using the RegisterDomain action in SWF API 
    * maximum workflow is 1 year (value specified in seconds) 
    * Parameters are specified in JSON format 
￼

SWF v SQS 
* SWF: task-oriented API
    * SQS: message-oriented API 
* SWF: ensures a task is assigned only once and never duplicated 
    * SQS: need to handle duplicated messages; may also need to ensure that a message is processed only once 
* SWF: keeps track of all the tasks and events in an application 
    * SQS: need to implement your own application-level tracking, especially if your application uses multiple queues 

Exam tips: 
- Look for human interaction, look for delivery windows (hours as in SQS v days/months as in SWF)
- Language: domains, deciders, workers 

￼
￼
