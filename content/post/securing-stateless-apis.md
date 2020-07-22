---
date: 2020-07-21
description: "How would you secure simple stateless APIs?"
featured_image: "https://images.unsplash.com/photo-1585907448410-582aac876c43?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1478&q=80"
tags: []
title: "Securing Stateless REST APIs"
disable_share: false
featured: "true"
---
In the [previous post](http://localhost:1313/post/rest-api-safety/) we had discussed
elaborately about the different threats that a REST API can face. We had gone over what
each attack means and some of its widely accepted solutions. It is always better to lead
with example rather than to preach merely by words, this article does the same. We will
share our experience, the way we secured our REST APIs. As we had promised we are back
with an illustration based on our experience with a sample REST API, that will help you
develop an approach to secure them in your project.

We will be sharing our experience that we had with a purely stateless API. We did not
use any user model or authentication for our REST API. Consider the following sample API
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

The main intention of any attacker when he is trying an injection attack is inserting a
malicious piece of code that can be executed on the server. Thus, data integrity is what
needs to be maintained. In our case to check for the payload integrity we calculate the
hash of the entire request payload by some secret which is a combination of a secret key
and current timestamp.The key and the hash function is pre decided and known to both the
client and the server. The hash value and the timestamp are sent over as request headers
of the REST API. On receiving the request on the server, the server calculates the hash of
the received request payload and matches it with the hash received in the request header.
If the hash values do not match the request should be dropped without processing further.

_Below are the steps to generate hash_:
- Since we have the request payload in JSON format we convert it to a string.
- Generate a hash value of the request payload string say `hash(client)` with the help of a
  secret and a pre-decided hash function (HMAC/ECDSA), known to both server and client.
- Add this hash value as one of your request headers. On receiving the request at the
  server you can follow the similar steps as were followed at the client to create a hash
  value say `hash(server)`.
- Compare `hash(client)` and `hash(server)`. If both the hash values match let the server proceed
  with the request else drop the request.

## Broken Authentication and Authorization

_Can you imagine an authentication/authorization mechanism without a database or an user
model?_

The answer is yes, but as we said we do not have any user model in our API.
But in our case there is another server say `server(user)` which is hosting all the user data.
This `server(user)` creates unique tokens for every user with some expiry time associated with
it. So our server is accepting this token with each request from client and asks
`server(user)` to validate it. If `server(user)` says the user is authenticated and authorized for
the operation we proceed with the request. Also if token is expired our client will prompt
user and ask for valid credentials to generate a new token. This new token will be passed
with subsequent requests.

## Man In The Middle (MITM) attacks

Here data at rest was not our concern since we dont store any users data. But to secure
data in motion we ensured data tansfer between our client and server along with to and
from our server to ABC server if through secure network. This is achieved by using HTTPS
channel secured by TLS protocol instead of simple HTTP. needs rephrasing

## Replay attacks

[Replay attacks](https://en.wikipedia.org/wiki/Replay_attack) are a broad term, but here
it means to re-send any API request and get back the same request. To avoid such attacks
we keep a request acceptance timeout for certain requests at our server, say 300 seconds
which means that such requests will be blocked by the server if they are received by the
server after 300 seconds.

Steps to acheive this are:
- Now we will add client utc time in request header say `time(client)`.
- Server will check the difference between `time(client)` with its own time.
- If the difference is greater than the time delay constant then server will drop the
request otherwise proceed the request further.

To set time delay constant we should consider some factors such as network latency,
processing capabilities otherwise we can end up with dropping valid requests.

## Denial of Service (DoS and DDoS)

For this type of attack we used request ratelimit technique. As our server is written in
python using Django framework, we used django ratelimiter utility. This ratelimiter is
applied for every API which checks how many times request being hit and limits its use
based on users identity and ip address of the machine being used for request.

Most frameworks and web servers have similar libraries to achieve this functionality.

_This is our approach to secure all our stateless REST APIs to avoid and minimize the
threats. We hope this will act as a guide to readers of this article for designing
approaches/techniques to secure their REST API. In the future, we will be diving deep into
some of the concepts explained here._
