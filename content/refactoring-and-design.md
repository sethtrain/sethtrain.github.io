Title: Refactoring this Flask View
Date: 2019-06-15 12:00
Category: Development
Tags: python,refactoring,design

Recently, [at my day job](http://myemma.com), I ran across a situation that I wondered if I could make better.  _"Better"_ to a software
engineer is definitely subjective.  Hopefully I make the case as to why I think it is better.

Change
------

If there is one thing I've learned as a software engineer it is that change is inevitable.  There are [smart](https://www.amazon.com/Working-Effectively-Legacy-Michael-Feathers/dp/0131177052)
[people](https://www.youtube.com/watch?v=rI8tNMsozo0) who have made it a part of their books and conference talks.  So my main focus
is not truly to implement some design pattern as much as it is to get a handle on dependencies and optimize for the _potential_ ability
to change. _Potential_ is important here.  My changes could be a waste of time, I don't think they are as you will see but, there is
always room for "good enough" and move on.

*Note* - While I don't go into the details of actually refactoring this view I did have to ensure that this functionality was
tested so that I had the confidence to be able to [make the simple change](https://twitter.com/kentbeck/status/250733358307500032?lang=en).

The view
--------

```python
@blueprint.route("", methods=["POST"])
def create_preferences(account_id):
    # ... some general setup

    subscriptions = payload["subscriptions"]
    optouts = payload["optouts"]

    # ... the core functionality

    for subscription_id in subscriptions:
        try:
            message = make_member_subscribed(account_id, member_id, subscription_id)
            subscriptions_topic.send(message)
        except Exception as exc:
            failure = "Failed to send subscription message for member %s: %s" % (member_id, exc)
            logger.warning(failure) 

    for subscription_id in optouts:
        try:
            message = make_member_opted_out(account_id, member_id, subscription_id)
            subscriptions_topic.send(message)
        except Exception as exc:
            failure = "Failed to send optout message for member %s: %s" % (member_id, exc)
            logger.warning(failure)

    # ... return successful response
```
This view function is already quite long but my main focus was to be able to test this view without having to test the message
dispatching, which caused me to think _"maybe this is doing too much"_.

The Dispatcher
--------------

The first step I took was to pull out this functionality into a class.  I wanted to try and keep this consistent with the [Single
Responsibility principle](https://en.wikipedia.org/wiki/Single_responsibility_principle) and, as [Sandi Metz](https://www.deconstructconf.com/2018/sandi-metz-polly-want-a-message) teaches,
be able to explain it's role with one sentence.

```python
class Dispatcher(object):
    def __init__(self, pipe, logger):
        self.pipe = pipe
        self.logger = logger

    def send(self, data):
        raise NotImplementedError("dispatchers must implement send")
```

I know _pipe_ isn't potentially the best name but it is good enough for now. This "interface" class allowed me to begin my core
implementation for the functionality in the view.

```python
class MemberSubscribedDispatcher(Dispatcher):
    def send(self, data):
        try:
            self.pipe.send({
                "type": "subscriptions-member-subscribed",
                "account_id": data["account_id"],
                "member_id": data["member_id"],
                "subscription_id": data["subscription_id"],
            })
        except Exception as exc:
            failure = "Failed to send subscribed message for member %s: %s" % (data["member_id"], exc)
            self.logger.warning(failure)
```

By [injecting](https://en.wikipedia.org/wiki/Dependency_injection) the _pipe_ and _logger_ above I now have a class that is only 
dependent on the objects passed to it.  I have also defined the contract for how dispatching works.  As long as the _pipe_ implements
`send` and the _logger_ implements `warning` it can successfully be injecting into this implementation.  I have also remove the
factory function that generated the message format previously and brought that message format closer to the behavior.

I submit that this class is quite simple and does just one thing (two if you count handling the error of a pipe),
_"dispatch a member subscribed message to a given pipe"_.

This view functionality can now change.  We could change the AWS topic we are using to be a Kinesis Firehose as long as we implement
the functionality in a manner that meets this "interface".

The refactored view
-------------------
After all that, here is where we landed.

```python
@blueprint.route("", methods=["POST"])
def create_preferences(account_id):
    # ... some general setup

    subscriptions = payload["subscriptions"]
    optouts = payload["optouts"]

    # ... the core functionality
    
    subscribed_dispatcher = MemberSubscribedDispatcher(subscription_topic, logger)
    for subscription_id in subscriptions:
        subscribed_dispatcher.send({"account_id": account_id,
                                    "member_id": member_id,
                                    "subscription_id": subscription_id})
    for subscription_id in optouts:
        # ... MemberOptOutDispatcher

    # ... return successful response
```
This view is now easier to test.  If I feel so inclined I could test that these functions are called within the test.  All the
functionality is tested in the `MemberSubscribedDispatcherTestCase` that I implemented as a part of this refactor so I only focused
on ensuring that these dispatchers actually dispatched the amount of times I expected given the rest of the view functionality.

Better
------

So why is this better?  I propose that this implementation is better for a couple of reasons.  

* I have pulled the implementation into a class that is only dependent on what is sent to it.  We could go so far as to change how we log and still not have to change this implementation.  Sure, we might have to implement some form of adapter if we ever decided to do that but I'm okay with that.
* It is now easier to test this functionality.  I no longer have to depend on mocking a request or anything like that.  I definitely could get this benefit from a simple function.  I just chose not to go that direction here.
