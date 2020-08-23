---
date: 2020-09-01
description: "Diving into Django's SECRET_KEY"
featured_image: "https://images.unsplash.com/photo-1537111493080-d1bfb2e415ad?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1950&q=80"
tags: []
title: "Purpose of SECRET_KEY in Django"
disable_share: false
featured: "true"
---

Have you ever wondered why Faizal from our famous cult film [Gangs of Wasseypur](https://en.wikipedia.org/wiki/Gangs_of_Wasseypur) is featured in first part of the series. His character is given no importance in the first part, but he is still there. It is the ultimate part of the series where he emerges as a HERO. Now you would be wondering why this idiot is talking about films in a technical article? This is just to introduce you to a similar character in Django applications, the `SECRET_KEY`.

While developing Django applications it is often a question what is this `SECRET_KEY` and why is it here? Is it that important? The official documentation even goes on to say that one of the first things you need to worry about is keeping the infamous `SECRET_KEY` setting secret. In fact, it's one of the first things mentioned in the [Official Deployment Checklist](https://docs.djangoproject.com/en/3.1/howto/deployment/checklist/#deployment-checklist). But what is the actual purpose of `SECRET_KEY`? We know it needs to be kept secret, but why?

Integrity of the data that travels over the network is of utmost importance. Use of unsigned data over the network is can always be at a risk of MITM(Man in the middle) attack. We have discussed about this in our [previous articles](https://awsomsauce.tech/post/securing-stateless-apis/). Thus using a message authentication mechansim becomes a necessity. It is here that Faizal gets on the forefront to handle the security and integrity of the travelling data. HMACs (Hash-based Message Authentication Codes) are used to generate hash values of the travelling data. These hash values are used as authentication codes.

`SECRET_KEY` is used exactly for what it stands, for keeping something secret and maintaining its integrity. `SECRET_KEY` is passed on to the hmac functions to compute the hashes. It is also used as a secret in [`salted_hmac()`](https://github.com/django/django/blob/master/django/utils/crypto.py#L19), but what does [`salted_hmac()`] even mean? Looking at the signature of the function in Django's source code:

```python
def salted_hmac(key_salt, value, secret=None, *, algorithm='sha1'):
    """
    Return the HMAC of 'value', using a key generated from key_salt and a
    secret (which defaults to settings.SECRET_KEY). Default algorithm is SHA1,
    but any algorithm name supported by hashlib.new() can be passed.

    A different key_salt should be passed in for every application of HMAC.
    """
```

`salted_hmac()` is just like the salted packet of Lays. The chips are the same, made from potato, fried in oil but SALT is added to it to enhance its taste. Similarly, `salted_hmac()` is just a slight variation of hmac, where we use a randomly generated value the salt, with our actual key to increase the entropy of the actual key to be used in hmac creation. The salt is just an additive with the `SECRET_
KEY` to enhance the strength of the hash functions.


These functions are used at several places in Django:

- To generate the session key for an authenticated user. This is done by using the user password as the HMAC value.
- To generate the password reset token for a user who has requested to reset the password. The user and the timestamp are used as values in this HMAC.
- For cryptographic signing. See [Cryptographic signing | Django documentation | Django](https://docs.djangoproject.com/en/3.1/topics/signing/)

---

In Django 4.0, a lot of the signing and cryptographic methods will be upgraded to use SHA-256
and hence none of the hashes that were generated pre 3.1 will work.
There was also a lot of code involving the SECRET_KEY refactored to re-use functions that use it.

See [Django Deprecation Timeline | Django documentation | Django](https://docs.djangoproject.com/en/dev/internals/deprecation/#deprecation-removed-in-4-0)

---

___Bonus Tip:___ Lot of Application Developers keep the `SECRET_KEY` hard coded in `settings.py`, but if you are developing a production level application and your application goes for security scans, you will encounter a vulnerability from your security team saying "Hardcoded sensitive information: Insecure cryptographic storage". So instead you can configure your application to read the `SECRET_KEY` from system environment variables. This will allow you to use the features of the `SECRET_KEY` and you will also be saved from the troubles of security assessment issues.






