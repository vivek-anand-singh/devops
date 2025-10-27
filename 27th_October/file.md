Syllabus:-

![alt text](image-1.png)

Marks Distribution:- 

Microservice Architecture 

What is Monolithic ? : 
Big 

So Microservice:

DNS 

Load Balancer:-

Path Based
Identifying Instance and then distributing

Authentication / Authorization :-
Authentication: Are you a valid user ?  / Authorization: Are you authorized to do the task ?
There are multiple instance of this

3 axes of scalibility Book
All microservices should have single database
So we store this type of data in databases like Redis(Cache Service)

Account Service

Polyglot: The term Polyglot DevOps refers to DevOps environments or engineers who work with multiple programming languages and technologies across the software delivery pipeline.

If there are multiple instances of microservices then all should have load balancer in between . 

All load balancer should have service discovery: 
Service Discovery: LB should be capable to know whne a knew instance comes up or existing goes down

Microservice to Microserice communication
Synchronous : Directly talking using Load Balancer
Asynchronous : Eventual Consistency For this we use Rabbit Mq Kafka and all
Methods of Asynchronous:-
Publisher Subscriber: Pub sub you get notified . Publisher puts the message and subscriber takes it
Queue: One of the instance takes it's message and then deletes that message

Event Syncing: How much time did it took and which instace got called ?
Logs: System issue logged

For microservice we log in some one external service
Tools Splunk ELK DataDog 

The Twelve-Factor App is a set of 12 best practices for building scalable, maintainable, and portable software-as-a-service (SaaS) applications.

API Gateway

https://12factor.net/


Dependencies:
Service Dependency: Microservice dependency , Data Dependency
Component Dependency: Using some sort of SDK 
