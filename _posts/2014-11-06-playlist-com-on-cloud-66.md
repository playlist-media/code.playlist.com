---
layout: post
title:  "Playlist.com on Cloud 66"
date:   2014-11-06 10:18:00
author: jacobwgillespie
---

Hi, my name is Jacob and I am an engineer at [Playlist](http://www.playlist.com), an internet radio streaming company.  In my role, I wear many different hats and touch many different systems encompassing backend, frontend, and devops.  Last month, we successfully migrated our Ruby on Rails primary application from Heroku onto AWS EC2 hardware using [Cloud 66](https://www.cloud66.com/), with great success.  So, I wanted to share some of our story.

Since September of last year, we were utilizing Heroku to host Playlist.com, our main application.  We had migrated from entirely custom servers with custom bash scripts for deployment to Heroku for the ease of rapid deployments, and also for the ability to quickly attach addons to our application (like Redis).  We also appreciated the ability to scale up our app serving capacity as our userbase grew.  Being able to run git push to trigger deployments made everything quick.  However, as time progressed, cost and network performance really began to hurt.

##The Problem

![costs](https://www.dropbox.com/s/p4ll7fr7arxtp9w/Screenshot%202014-11-01%2017.04.04.png?dl=1)

Running our app was expensive on Heroku.  Our hosting bill started out in the hundreds of dollars per month and steadily grew towards thousands per month.  In May of 2014, we launched a more complex backend that nearly tripled our costs.

At the peak of the problem, we were running an average of 24 2X dynos (!) per day in order to keep the request queuing time down.  Anything lower and requests would start to pile up and response times would begin to climb, and we could literally watch the number of concurrent users in Google Analytics plummet 80%.  There were multiple pain points with our backend that could have potentially been solved by a rearchitecting to use more background processes and perhaps something like websockets, but with a small team, we really could not afford to stop development on the next backend in order to patch the current one.

We began building the backend for the next major version of our products.  By using [Go](https://golang.org) and incorporating a lot of the lessons learned from the first backend, it was considerably more performant.  But in the meantime, we really needed to do something with the Rails app until we could complete and test the new backend.

##The Solution

I had heard of [Cloud 66](https://www.cloud66.com/) about two years ago on [Hacker News](https://news.ycombinator.com/news), but had never really given it a serious try.  After some research, I decided to spin up our Rails app on EC2 using Cloud 66 to see if it might be a viable alternative to Heroku.  It was painless to set up.  After it analyzed our application, I edited the environment variables with our custom values and told Cloud 66 not to manage the database (we use Amazon RDS), and Cloud 66 built our first server.  Customer support was super helpful, especially via chat, and I increased our server count to three (on c3.large instances).  Once I was confident that the application was running correctly, I switched our DNS record at CloudFlare, and instantly we were live in production.

*Side note: the [Cloud 66 toolbelt](http://help.cloud66.com/toolbelt/introduction.html) is very useful.  During the migration, it was easy to keep track of the logs with the cx tail command.*

![performance](https://www.dropbox.com/s/q8lkckx9a38qapd/Screenshot%202014-11-01%2017.16.25.png?dl=1)

The blue bar between 10/21 08:00 and 10/21 16:00 is our switch to Cloud 66, and the difference in web app performance was immediately noticeable.  Plus, this was with only 3 servers (instead of those 24 dynos).  Our number of song plays per day (an important metric to us) increased almost immediately by about 15%, demonstrating again that web app performance directly impacts key metrics.

Cost was *drastically* improved.  We increased our server count to 4, just to be safe, and with AWS reserved instances, our monthly costs are now less than 10% of what they were on Heroku.  App performance is significantly more stable and response times are lower.

Additionally, we did not lose the convenience of being able to deploy with git, and actually improved the workflow by using the Cloud 66 redeploy hook with our continuous integration service, [Circle CI](https://circleci.com/).  Now if our tests pass, deployment is automatic.  Check out the [Github Flow](https://guides.github.com/introduction/flow/index.html) for more information on our development process.

##Going Forward

We really love the experience with Cloud 66.  I was a little disappointed that we would not be able to use Cloud 66 when we switch to the non-Rails Go backend.  However, Cloud 66 is working on supporting [Docker](https://docker.com/), so we are happy we will be able to package our app into a container and continue to take advantage of the awesome features and support of Cloud 66.

Another feature I am excited about utilizing is [Elastic Addresses](http://help.cloud66.com/dns/elastic-address.html) - we can spin up a backup deployment on another cloud (Digital Ocean, Rackspace, etc.), and then if (/when) AWS us-east-1 goes down, we can failover to the other cloud.

Overall, we have been very happy with [Cloud 66](https://www.cloud66.com/) for our application deployment.  If you have not tried it out, I would highly recommend it.
