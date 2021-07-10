---
date: 2020-11-15
description: "Improving asynchronous task throughput using Redis Cluster"
featured_image: "https://images.unsplash.com/photo-1519248200454-8f2590ed22b7?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1655&q=80"
tags: []
title: "Integrating Huey with Redis Cluster"
disable_share: false
author: "Ankit Surkar"
---

Asynchronous programming has come under spotlight in recent times. It gives the developers and the architects a flexibility of developing the application where some tasks that are not needed to be serially executed or where the responses are not expected instantly to accept the tasks and work on it as and when the resources are available. We have a large number of tools and techniques that can be used to develop applications with the aforementioned features.

For Python developers the first choice for asynchronous programming has always been Celery. Celery is a feature rich Python library and offers a wide range of capabilities to the developers. But you would agree that everytime the best is not required, becuase the stuff that makes it the best may become overheads for systems that do not need those features. There is a high chance that you do not need all the capabilities that Celery provides and also may think that the overhead it brings with it might affect your application adversely.

The saviour in situations where, you just want to execute tasks asynchronously without adding a lot of dependencies to your codebase, [Huey](https://huey.readthedocs.io/en/latest/) can be a perfect choice. Huey is extremely lightweight in terms of code, resources as well as set up.

A major requirement that may need some architectural design thinking is that Huey requires a mechanism to maintain the task queue, and out of the available options, Redis is the [most widely used](https://www.theregister.com/2020/11/23/redis_the_most_popular_db_on_aws/).
Huey works perfectly when the application is developed to use a single instance of Redis. But, having a single instance can be a cause of major catastrophy as it becomes a single point of failure. To overcome the issue of single point of failure Redis has come up with solutions like Redis Cluster and Redis Sentinel. But, there is caveat with Huey when it comes to multi-instance Redis solutions such as Redis Cluster and Redis Sentinel, and it is that huey does not innately support these Redis solutions.

This limitation is not that big a limitation that we give up using Huey. Just a few lines of code can make Huey work with the Redis Cluster without loosing advantages that Huey provides compared to other Python task managers.

Firstly, we will use [`redis-py-cluser`](https://pypi.org/project/redis-py-cluster/) as a python client for Redis Cluster. `RedisHuey` and `RedisStorage` are classes defined in Huey library. Next, we will define subclasses for `RedisHuey` and `RedisStorage` and using `redis-py-cluster` as client for Huey connection object, as shown in the following code snippet.

```python
from rediscluster import RedisCluster, connection
from huey.api import RedisHuey
from huey.storage import RedisStorage, SCHEDULE_POP_LUA

class MyRedisStorage(RedisStorage):

    def __init__(self, *args, **kwargs):
       super().__init__(*args, **kwargs)
       cluster_endpoint = [{"host": "redis-cluster-host", "port": "6379"}]
       self.pool = connection.ClusterConnectionPool(**kwargs)
       self.conn = RedisCluster(startup_nodes=cluster_endpoint, skip_full_coverage_check=True)
       self.connection_params = kwargs
       self._pop = self.conn.register_script(SCHEDULE_POP_LUA)


class MyRedisHuey(RedisHuey):

   storage_class = MyRedisStorage

   def __init__(self, name, **kwargs):
       super().__init__(name='my-task-queue', storage_class=MyRedisHuey.storage_class, **kwargs)
```

With the above code that can connect Huey with Redis Cluster, we just need to instruct Huey to use the implmentation. For that we would have to override `huey_class` property from Huey configuration object and assign it class name which we have just defined in the code snippet.

```python
HUEY = {
    # one may need to assign class name with relative path based on framework
    # being used for application development
    'huey_class': 'MyRedisHuey',
    ...
}
```

With some easy tweaks as mentioned above you can make use of a lightweight, easy to operate and easy to maintain asynchronous task manager the Huey.
Happy Coding!!!
