# Thoonk #

Thoonk is a persistent (and fast!) system for push feeds, queues, and jobs which
leverages Redis. Thoonk.py is the Python implementation of Thoonk, and is
interoperable with other versions of Thoonk (currently Thoonk.js for node.js).

# Feed Types #

## Feed ##

The core of Thoonk is the feed. A feed is a subject that you can publish items
to (string, binary, json, xml, whatever), each with a unique id (assigned or
generated). Other apps and services may subscribe to your feeds and recieve
new/update/retract notices on your feeds. Each feed persists published items
that can later be queried. Feeds may also be configured for various behaviors,
such as max number of items, default serializer, friendly title, etc.

Feeds are useful for clustering applications, delivering data from different
sources to the end user, bridged peering APIs (pubsub hubbub, XMPP Pubsub,
maintaining ATOM and RSS files, etc), persisting, application state,
passing messages between users, taking input from and serving multiple APIs
simultaneously, and generally persisting and pushing data around.

## Queue ##
Queues are stored and interacted with in similar ways to feeds, except instead
of publishes being broadcast, clients may do a "blocking get" to claim an item,
ensuring that they're the only one to get it. When an item is delivered, it is
deleted from the queue.

Queues are useful for direct message passing.

## Job ##

Jobs are like Queues in that one client claims an item, but that client is also
required to report that the item is finished or cancel execution. Failure to to
finish the job in a configured amount of time or canceling the job results in
the item being reintroduced to the available list. Unlike queues, job items are
not deleted until they are finished.

Jobs are useful for distributing load, ensuring a task is completed regardless
of outages, and keeping long running tasks away from synchronous interfaces.

# Installation #

In theory, if I publish the package correctly, then this should work.

    pip install thoonk

## Requirements ##

Thoonk requires the redis-py package. I strongly recommend installing
the hiredis package as well, since it should significantly increase performance.

Since the redis package is undergoing refactoring with backwards incompatible
changes, be sure to use version 2.2.4.

    pip install redis=2.2.4
    pip install hiredis

## Running the Tests ##

After checking out a copy of the source code, you can run

    python testall.py

from the main directory.

# Using Thoonk #

## Initializing ##

    import thoonk
    pubsub = thoonk.Thoonk(host, port, db)

## Creating a Feed ##

    thoonk.create_feed(feed_name, {"max_length": 50})

OR create an object referencing the feed, creating it if it doesn't exist,
reconfiguring it if you specify a configuration.

    test_feed = thoonk.feed(feed_name)

The same is true for Queues and Jobs
    
    test_queue = thoonk.queue(QueueName);
    test_job = thoonk.job(JobName);

## Configuring a Feed ##

    thoonk.set_config(feed_name, json_config)

### Supported Configuration Options ###

* type: feed/queue/job
* max\_length: maximum number of items to keep in a feed

## Using a Feed ##

    feed = thoonk.feed('test_feed')

### Publishing to a Feed ###

Publishing to a feed adds an item to the end of the feed, sorted by publish time.

    feed.publish('item contents', id='optional id')

Editing an existing item in the feed can be done by publishing with the same ID as
the item to replace. The edited version will be moved to the end of the feed.

### Retracting an Item ###

Removing an item is done through retraction, which simply requires the ID of the
item to remove.

    feed.retract('item id')

### Retrieving a List of Item IDs ###

Retrieving all of the IDs in the feed provides the order in which items appear.
   
    item_ids = feed.get_ids()

### Retrieve a Dictionary of All Items ###

Retrieving a dictionary of all items, keyed by item ID is doable using:

    items = feed.get_all()

#### Iterating over items in published order ####

    items = feed.get_all()
    for id in feed.get_ids():
        do_stuff_with(items[id])

### Retrieving a Specific Item ###

    item = feed.get_item('item id')

## Using a Queue ##

    queue = thoonk.queue('queue_feed')

### Publishing To a Queue ###

    queue.put('item')

### Popping a Queue ###

    item = queue.get()
    timed_item = queue.get(timeout=5)

## Using a Job Feed ##

    job = thoonk.job('job_feed')

### Publishing a Job ###

    job.put('job contents')
    job.put('priority job', priority=job.HIGH)

### Claiming a Job ###

    data = job.get()
    timed_data = job.get(timeout=5)

### Cancelling a Job Claim ###

    job.cancel('job id')

### Stalling a Job ###

    job.stall('job id')

### Retrying a Stalled Job ###

    job.retry('job id')

### Retracting a Job ###

    job.retract('job id')

### Finishing a Job ###

    job.finish('job id', 'result contents', result=True)
    job.finish('job id', 'result contents', result=True, timeout=5)

### Check Job Results ###

    query, result = job.get_result('job id', timeout=5)

# The Future of Thoonk #

# Writing Your Own Implementation or Peer #
