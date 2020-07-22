---
date: 2020-06-24
description: "An Introduction to REST API safety"
featured_image: "https://images.unsplash.com/photo-1519575706483-221027bfbb31?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1502&q=80"
tags: []
title: "Are your REST APIs Safe?"
disable_share: false
author: "Ankit Surkar"
---
_“Don’t write the complete damn code my boy just get an API that connects with our application and get your result”_, you would have heard many of your techie friends, your product architects advising you this in your product discussions.

REST APIs (Application Programming Interfaces) have become an inseparable part of our industry today due to its ease of use, ease of integration, platform independence and most importantly time saving capabilities. You think of an use case and there is some or the other API available for that, if you create your own web service, your architects will suggest you try to write the web service as an API, so that any new consumer can easily integrate with your web service. This being said, the biggest concern that arises is the safety of these APIs.

APIs are somewhat like service integrators who take input from consumers, give it to the servers for processing and give back the response to the consumers, much like the waiters at a restaurant who take our order (which is our input) gives that to the chef (our server) who then prepares the delicacies (processing) and gets us the prepared food (response). Imagine a case where the waiter is stopped by one of your rivals, he changes your order and adds a peanut dish which you are allergic to, what will happen? Similar case can arise for our service integrators, the APIs.

In this article we will go over a few threats that the REST APIs face and some of its widely accepted remedies. Henceforth, in the article whenever API is mentioned it is REST APIs that we will be talking about.

## Injection Attack

This is very similar to the one I had mentioned in the analogy before.

- The attacker injects a potentially dangerous piece of code or a query in the API request to stage the attack.
- SQL-injection and cross site scripting are the most widely known injection attacks.
- This malicious input actually gets executed, exposing the data or giving attackers the control of your system.
- Since data integrity is under threat request payload validation is the only solution for this. For validation of the request, you can create a hash/checksum of request payload and pass it as one of the request headers. On the server side this hash/checksum value can be validated before processing the request.

This is one of the easiest ways, you can also encrypt the entire payload and then sign the payload using some key and then validate both the encrypted data and the signature at the server side. 

## Broken Authentication and Authorization

- When an API is designed with weak or no authentication in place an attacker can take control over several accounts.
- The attacker can impersonate the actual user, thus getting the privileges that are given to the users.
- Token based authentication and authorization is a widely used remedy for authentication problems.
- Usually nowadays JSON Web Tokens (JWT) are issued by the server to all the clients during authentication, to be sent over when they actually request a service.
- If you have a user model and a database in place in your API, this is easy. But if you are developing a stateless system without a database you can take help of some third party providers that can help you with the tokens.
- If some other supporting application in your organization can issue tokens for you, you can even take help of that. You will see a great example of this in coming articles where we would illustrate the ways and techniques we have used.

## Exposure of request data

- The data exposure is a potential threat for both data in motion and data at rest.
- The data in motion can be attacked in case the APIs use insecure medium to transfer the data.
- When the data is captured on the user’s machine or on the server in an exploitable state is the attack on data at rest.
- Usually efficient encryption techniques can be employed to stop the attack on the data at rest.
-  Additionally, you can also implement some name mangling, obfuscation or encoding techniques while sending data through the request payload so that it will be difficult for the attacker to decipher the data being sent over the wire.

## Man In The Middle (MITM) attack

- MITM is active eavesdropping. The attacker inserts him/herself into a conversation between two parties, impersonates both parties and gains access to information that the two parties were trying to send to each other.
- MITM attacks allow the attacker to intercept, send and receive data intended for someone else. In our analogy this is the rival who tries to read the order you are giving.
- To avoid or minimize such attacks one should always try to use secure channels for data transfer.
- The most widely used and the simplest solution for this is to transfer your data using HTTPS. HTTPS is a widely accepted Layer 7 protocol that transmits the data in encrypted form over the network.
- In terms of our analogy, if the waiter writes your order such that your enemy cannot read it and passes on to the chef who can decode it perfectly.

## Replay attack
- If a valid data transmission is maliciously delayed or repeated, the system is under Replay attack.
- The attacker is able to intercept the request payload and intentionally delay it or repeatedly hit the same payload numerous times.
- To save your system from replay attacks, you can keep your REST API idempotent and make sure that a request can only be hit once and in allowed time duration.
- This can be done by passing some timestamp headers with the requests and validating it on your application server.
- The request should be allowed only if it reaches the server in the allowed time interval. This time needs to be of course based on your application processing capabilities and network latencies so that you do not actually deny the valid requests.

## Denial of Service (DoS and DDoS)

This is one of the most widely known attacks to the community, thus lets not go deep into it. Just to understand a scenario imagine it like,  when your waiter is overburdened with a large amount of unnecessary jobs that he is unable to take your request.
- The DoS and DDoS attacks are usually minimized by limiting the access to the server based on various filtering strategies and these strategies can be applied at various stages:

    - On Infrastructure level many well known cloud providers have their own specialized tools for this, for example AWS provides AWS Shield to stop the directed attacks like this. Even some firewalls provide this security feature.

    - As a developer as well, you can ratelimit the use of your API by some filtering techniques wherein you can set a limit based on user’s identity, IP of the machine being used to request the service, you can set a direct limit on the use of API as well irrespective of any filters if you have developed some kind of an API wherein the use is limited and the limits are known.

The above mentioned categories actually cover almost all of the known attacks on the APIs and also it is pretty evident that all of these attacks are very different from each other and thus a single solution will not help us to resolve all of the issues.

Thus, **_the security of your API_** is a difficult problem to solve and there is no single full proof solution that can solve all problems. You need to cautiously select a combination of solutions for your APIs based on its use case, severity of the attack and available resources. At times it becomes difficult to use some framework based or widely used utility as a solution maybe due to the use case or availability of resources, thus it becomes very important that you understand the attacks and plan for the strategies of mitigation during the design phase itself.

We hope this would be a helpful piece of information for all the readers. We are coming up with a series of on API security wherein we will discuss various different API security issues in depth. Also In one of our upcoming pieces we will try and illustrate with an example - How can you develop your API security strategies.

Till then Happy Coding!!!
