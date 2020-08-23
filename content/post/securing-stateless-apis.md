---
date: 2020-07-24
description: "Gather insights on securing your simple stateless APIs"
featured_image: "https://images.unsplash.com/photo-1585907448410-582aac876c43?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1478&q=80"
tags: []
title: "Securing Stateless REST APIs"
disable_share: false
---
In the [previous post](/post/rest-api-safety/) we had discussed elaborately about the
different threats that a REST API can face. We had gone over what each attack means and
some of its widely accepted solutions. It is always better to lead with example rather
than to preach merely by words, this article does the same. We will share our experience,
the way we secured our REST APIs. As we had promised we are back with an illustration
based on our experience with a sample REST API, that will help you develop an approach to
secure them in your project.

We will be sharing our experience that we had with a purely stateless API. We did not use
any user model or authentication for our REST API. Consider the following sample API
specification for all the solutions described in the post. This API is used to generate
salary slips for the employees in the system.

```
URL - /generate/salaryslip
Method - POST
Request header -
{
  "Content-Type": "application/json"
}

Request payload -
{
  "emp_id": "01234",
  "emp_code": "ENG",
  "emp_name": "John Doe",
  "emp_grade": "7.1",
  "working_days": 24
}
```

With the baselines set we would now dive deep into the threats and an approach for
remediation. As we have discussed earlier it would be very difficult to put up a single
solution that can remediate all the attacks and thus we have broken down the approach into
sections that will follow.

## Injection attacks

The main intention of an attacker when he is trying an injection attack is inserting a
malicious piece of code that can be executed on the server. Thus, data integrity is what
needs to be maintained. In our case to check for the payload integrity we calculate the
hash of the entire request payload by some secret which is a combination of a secret key
and current timestamp. The key and the hash function are pre-decided and known to both the
client and the server. The hash value and the timestamp are sent over as request headers
of the REST API. On receiving the request on the server, the server calculates the hash of
the received request payload and matches it with the hash received in the request header.
If the hash values do not match the request should be dropped without processing further.

Below are the steps to generate hash:
- Since we have the request payload in JSON format we convert it to a string.
- Generate a hash value of the request payload string say `hash(client)` with the help of
  a secret and a pre-decided hash function (HMAC/ECDSA), known to both server and client.
- Add this hash value as one of your request headers. On receiving the request at the
  server you can follow the similar steps as were followed at the client to create a hash
  value say `hash(server)`.
- Compare `hash(client)` and `hash(server)`. If both the hash values match the server
  proceeds with the request else drop the request.

Now that we have saved our servers form the attackers who might try to run a piece of
malicious code to gain access to the servers, a logical extension to this will be stopping
any request which seems suspicious with respect to the time it takes to reach the server
and that is what we talk about in our next section.

## Replay Attacks

A [replay attack](https://en.wikipedia.org/wiki/Replay_attack) is a broad term, but what
we consider here is that if a request does not reach the server in a specified time limit
it may contain malicious content. And it would be pretty logical to drop such suspicious
requests rather than compromising the server. To avoid such attacks we keep a request
acceptance timeout the requests at our server, say around 300 seconds which means that any
request that takes more than 300 seconds to reach the server should be dropped. Obviously,
to select a value of request acceptance timeout some factors such as network latency,
processing capabilities should be considered well otherwise one can end up dropping the
valid requests.

Following are the steps we used to save our server from these attacks:
- At the client-side, we add UTC time in the request header say `time(client)`.
- On receiving the request, the server will check the difference between `time(client)`
  with its own time.
- If the difference is greater than the request acceptance timeout (in our case we chose
  120 seconds), the server drops the request otherwise it must proceed with the request
  further.

With the server safe from any type of injection or replay attack next most important thing
that falls in line is the safety of data travelling over the wire. The following section
will briefly take you over securing the data travelling over the network.

## Man in the Middle attacks

Since we are talking about the data in motion to secure the data the channel it travels
over needs to be reliant. So just think of a situation when you are travelling through a
trail in a dark and dangerous forest and you get the invisibility cloak of Harry Potter,
wouldn’t that make you travel safely. A similar concept is used to overcome the attacks on
the data travelling over the network. Similar to the invisibility cloak SSL/TLS protocols
become your saviour. It is very easy to employ this with your product. You may need to
install a TLS certificate signed by a recognized Certification Authority on the server.
The only mandatory step is to configure your server to serve the requests over HTTPS
channel rather than HTTP.

If you notice, we have secured our servers from malicious codes, we have also secured our
the data traveling over the wire. An even more secure system would be the one that allows
only authorized entities to make a request. In the succeeding section let us go over some
interesting discussion about authentication and authorization in a system lacking
databases.


## Authentication and Authorization

Can you imagine an authentication/authorization mechanism without a database or a user
model?

The answer is yes, and a little amount of smartness will save you from the trouble and
overheads of maintaining large databases or user models. Being lazy engineers we always
try to reduce the workload of development and the overheads of maintenance. In our case,
luckily there was another server say `server(user)` which hosted all the user data. This
`server(user)` was capable of creating unique tokens for every user with some expiry time
associated with it which could be used for authentication and authorization. We leveraged
this server to generate tokens for us and we consumed these tokens with each request from
the client. For validation, the received token was sent to the `server(user)`. If the
token sent in the request was valid the user was considered authenticated and authorized
for the requested operation. If the token received had expired our client prompted the
user to generate a new token. This new token will be passed with subsequent requests.

Similarly, if you are working on a project and are able to leverage similar service within
your organization, or if you are able to integrate with some third-party services that can
help you with authentication and authorization for your system, you will save a lot of
time, money, maintenance overheads and also you will be able to provide secure access to
only authorized entities. Okay, now that the data is safe, servers are safe, only
authenticated users are allowed do think this would be enough? Before answering the
question just imagine a scenario that you are the owner of an electronics shop and you
have just got a great television and you have put it on display and it is the day of
India-Pakistan final match. Since the match has got to a crucial juncture a lot of
pedestrians, cricket lovers create a crowd outside your shop. At the same time, a
person(may not be a cricket fan who knows) comes near your shop and wants to buy a
television but due to the crowd, he is not able to get to the shop. What if this happens
with your service? If some attacker attacks with a large amount of traffic to the
endpoints serving your service and since the system will be unable to process such
humongous bogus requests, real requests will be denied the service. This brings us to the
last section for this post.

## Denial of Service (DoS and DDoS)

In our case, we used the request rate-limiting technique. As our server was written in
Python using the Django framework, we used the Django ratelimiter utility. This
ratelimiter is applied for every API. The ratelimiter checks each request for parameters
like request IP, user requesting the service, etc. and keeps a count of it. If it is
identified that some user or IP is requesting the service more than the expected number(in
our case 5 requests per min), the requests from that user or IP would be blocked for a
time period. When you design your rate-limiting consider your use-case, your expected
traffic, your serving infrastructure, and processing time and capabilities. You can run
specific performance tests to get values of each of the aforementioned parameters and
other parameters endemic to your use case and proceed with the math-based to the results.

Most frameworks and web servers have similar libraries to achieve this functionality. If
you work with cloud service providers, a large number of DoS/DDoS safety tools have been
designed to be applied at the loadbalancer level. These will stop the requests identified
under DoS/DDoS category to even hit your workload processing systems.

_This was our approach to secure all our stateless REST APIs to avoid and minimize the
threats. We hope this will act as a guide to all of you for designing the
approaches/techniques to secure their REST API. In the future, we will be diving deep into
some of the concepts explained here._
