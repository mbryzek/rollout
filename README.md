rollout
=======

Design considerations for incremental feature rollout

Goals:
  - Feature deployment - would like the ability to launch a feature into production, but to have the feature dark or to enable that feature only for myself, my team, or my company
  - Incremental rollout - feature is ready for users to interact with. Rollout the feature to 1% of traffic, then scale incrementally to 100%
  - A/B testing - define unlimited number of test and control groups to gather data on a particular features

One of the constant challenges with these variations is the impact on reporting; e.g. if we cannot reliably report on the data, then we lose much of the value. In many cases we have observed distinct testing practices to access the information based on population segmentation.

api
===
See http://www.apidoc.me/bryzek/rollout/latest

notes
=====

Two major features for AB Testing:

  - Assignment: UI - activity driven. Only decide what variants a user belongs to on demand
Population split: figure out a way to divide the population into buckets of some sort
downside is that you may take longer to reach significance, esp for features that are only visible to specific users.
  -- Reporting: can access the AB testing service

Every new test is assigned a random key ( a string )
apply murmur3 to the user guid, using the test key as the seed
Hash to an integer: 0..100 or 0...1000
Then you can choose the split, e.g. 0-30 is A, 31-60 is B, 61 - 100 is C
For a rollout, you can do 0-10 for test, 11-100 is control. Gradually increase the test
Implement as copy on edit so you can see the change in percentages

Reshuffling has to be random and equally distributed - used MurMur3 - back testing to verify distribution

Key decision is what to use as the identifier - should be a string - device ID? User guid? other?
