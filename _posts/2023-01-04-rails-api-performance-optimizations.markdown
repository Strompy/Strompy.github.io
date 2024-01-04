---
layout: post
title: Rails API Performance
date: 2023-01-04
categories: Rails Performance Tips
---

_This post was first published on the [Brewer Digital Engineering Blog](https://brewerdigital.com/engineering/rails-api-performance-optimizations/)_


<h1>Rails API Performance Optimizations</h1>

Ruby on Rails is a dynamic and approachable ecosystem with many opinions on how it should be used, structured, built upon, improved, and maintained. But what do you do when your app begins to age and the accumulated data of years of requests causes your API to slow down? You have a few options.

<h2>Benchmarking and Automated Code Analysis</h2>

You may think that diving into code changes first, especially the obvious ones, might be the answer here but in the words of Treebeard: don’t be hasty. Not all potential optimizations can or will be helpful. Say, for instance, that you’ve come across a tip to use `Model.includes(:association)` to speed up your API calls; this would work in some cases, and not in others. If your app serializes that relation, it’s wise to include it in your initial database call in order to prevent making more database calls just to finish the initial request. On the other hand, including a relation that isn’t used can add unnecessary complexity to the original database call. Keep in mind that you should only fetch what you need.

When it comes to figuring out what methods or parts of methods take the most time to run, your best option is to use a benchmarking tool. There are numerous methods for benchmarking your app, but I’ll name a few in distinct categories.

Production monitoring:
- Datadog
- Newrelic

These monitoring services exist to monitor production data, largely. They report on nearly anything you could need to know about the database that your customers care about. This is a huge boon when you’re looking to find out which requests are the slowest, which errors occurred when,  and also makes these services indispensable for uncovering slow or error-prone methods. However, these are paid services, and are only good for finding more information on issues that are already affecting production. 

Development monitoring:
- [Bullet gem](https://github.com/flyerhzm/bullet)

Bullet is a gem that reports on any n+1 queries in your app, which are logged and announced as you click around your development (or production) app. 

Test benchmarking:
- Rspec: [rspec-benchmarking gem](https://github.com/piotrmurach/rspec-benchmark)
- [ActionTest](https://guides.rubyonrails.org/v3.2/performance_testing.html)
- Ruby’s [Benchmark module](https://ruby-doc.org/stdlib-2.5.3/libdoc/benchmark/rdoc/Benchmark.html)

Tests have the distinction of being the closest you can get to checking only your code. Benchmarking performance in your test spec can aid in catching or diagnosing speed issues before the code can become a pull review. You should be testing with large amounts and/or complex objects. 

<h2>Code Optimizations</h2>

You should do as many of the following as you are able. Always aim to prevent n+1 queries. Database queries themselves aren’t slow by any means, but you can easily write code that triggers a database call for each object in a collection, known as an “n+1 query.” These queries are likely where your app begins to chug, as this issue scales with complexity.

**Remove unnecessary code**. This is likely the most obvious suggestion; that’s why I put it first. Apps take many forms during their lifetimes, and some parts of the path through any single request could be bloated with unnecessary database calls, logs, API calls, or methods that no longer serve a purpose. If any of the removed code accesses the database, you might see some marked improvements.

**Eliminate loops where possible**. This will almost certainly require a rewrite, but you may find cases where iterating over a collection to grab attributes from each item might be less performant than plucking those columns from the association instead. 

**Clean up your serializer(s)**.  Maybe your serializer serializes something that’s no longer needed on the front end. You might be able to use the front end’s code to check for yourself or ask a coworker for help. You could also try implementing a new type of serializer. ActiveModelSerializer and JSONAPI-Serializer are strong options. 

**“Include” relations when you call upon those relations**. I mentioned this at the start, but it’s faster to ensure that all objects that are needed are grabbed from the database at once. For instance, let’s say that your serializer returns an “artist_name” attribute that calls the “name” field from the Artist table. Including the artist before serialization allows ActiveRecord to get the artist in the original call instead of needing to issue another to find that information. This makes little difference for a single song, but saves time for multiple Song records. You would see similar results for instances where you might also call upon an associated Genre,  Album, Movie, or TrackList.

**Do it later**. Anything that doesn’t need doing immediately, such as API calls, sending emails, updating records, or creating logs could be done at some later time. Don’t queue up anything that you need the results of within the same request. Update routes can take particular advantage of jobs if the response isn’t used on the front end. Sidekiq, Resque, and Delayed Job are your likely candidates if you don’t use the built-in ActiveJob.

**Cache your data**. If you can’t make the first call any faster, you can at least optimize the following requests. This is especially useful for any index route. If you paginate the data, be sure not to include the page number in the cache name. Be aware that caching in multiple places might obscure issues. Be thorough, and be sure to pick a reasonable cache time for the route.

**Ruby is slow**. ActiveRecord instantiates Model objects for each table row.  You can skip this by going straight to the database, using SQL. This requires more technical knowledge of SQL, but can cut down on unneeded objects.

**Use gems to replace your app’s logic**. It’s likely that your app’s current solution may have been overly-engineered. Just trust that someone else has already tackled the same problem you are currently addressing.  I don’t have any specific recommendations for you, as the gems you might want will depend on the app’s purpose, company policy, the time required to implement and test it, etc. You may wish to use a gem to handle money conversions, time zones, transactions, advanced mathematics, or any manner of high-concept or very complex interaction.
