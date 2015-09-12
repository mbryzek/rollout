# rollout

Design considerations for incremental feature rollout

## Goals

  - Feature deployment - would like the ability to launch a feature into production, but to have the feature dark or to enable that feature only for myself, my team, or my company

  - Incremental rollout - feature is ready for users to interact with. Rollout the feature to 1% of traffic, then scale incrementally to 100%

  - A/B testing - define unlimited number of test and control groups to gather data on a particular features

One of the constant challenges with these variations is the impact on reporting; e.g. if we cannot reliably report on the data, then we lose much of the value. In many cases we have observed distinct testing practices to access the information based on population segmentation.

## api

Complete documentation online at [apidoc](http://www.apidoc.me/bryzek/rollout/latest)

**Create a test**

    curl -X POST \
      -d {
        "name": "my test",
        "variants": [
 		  { "name": "test", "percentage": 50 },
		  { "name": "control", "percentage": 100 }
        ]
      } \      
      http://local	host/tests

**Update variants for a test**

    curl -X PATCH \
      -d {
        "variants": [
		  { "name": "a", "percentage": 10 },
		  { "name": "b", "percentage": 30 },
		  { "name": "c", "percentage": 60 }
        ]
      } \
      http://localhost/tests/<guid>

Note that the updates are implemented as copy on edit. The API exposes allocations as a way to see all of the variants and when each set of variants went live (or was ended).

**Get variants**

  - The default case will fetch all variants that are currently live for a particular ID
 
        curl http://localhost/variants?id=<your id>

    The id that you provide can be any string. Most common examples will be a session ID or a user ID. Be consistent and ensure that you will have the ability to access this ID later in reporting databases.


  - Or you can provide your own timestamp to fetch the variants that were live for this ID at a specific point in time. This is most useful for reporting applications that would like to query what specific variants were live at a particular point in time (e.g. when an order was placed, when a user registered, etc.)
 
        curl http://localhost/variants?id=<your id>&time=<timestamp>
    
    The timestamp will be iso8601 format.

The call to variants will return a paginated list of all of the variants that currently match the filter criteria. Each variant will contain a reference to the test (the guid) and the name of the specific variant.


## Miscellaneous Notes

Two major features for AB Testing:

  - Assignment: UI - activity driven. Only decide what variants a user belongs to on demand
Population split: figure out a way to divide the population into buckets of some sort
downside is that you may take longer to reach significance, esp for features that are only visible to specific users.

  - Reporting: can access the AB testing service

### General background for designing the test service

  - Every new test is assigned a random key ( a string )
  - apply murmur3 to the user guid, using the test key as the seed
  - Hash to an integer: 0..100 or 0...1000
  - Then you can choose the split, e.g. 0-30 is A, 31-60 is B, 61 - 100 is C
  - For a rollout, you can do 0-10 for test, 11-100 is control. Gradually increase the test
  - Implement as copy on edit so you can see the change in percentages

Reshuffling has to be random and equally distributed - used MurMur3 - back testing to verify distribution

Key decision is what to use as the identifier - should be a string - device ID? User guid? other? From an API standpoint - just accept a string. The caller will determine what string to pass in.

## Acknowledgements

Many thanks to Young Moon who shared his experience building this sort
of system for gilt. Young's work went live in 2014/2015 and was met
with enthusiasm by the other engineers at gilt - i.e. he built a
system that people loved to use. A lot of the API and work here is
thanks to him.