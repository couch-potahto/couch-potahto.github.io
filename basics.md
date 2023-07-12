layout: page
title: "On being a basic b***h"
permalink: /on-being-a-basic-b

What's the point of knowing the basics so well? 99% of the time, you'll never make use of it.
This was a sentiment I kind-of-agreed-with-but-not-really, and I didn't know how to raise a good counterargument for this mostly due to how I lacked personable evidence for the contrary. Like yeah, sure, we all *know* that Data Structures and Algorithms are the cornerstones of software engineering and I can reverse any trees any day babay~. But when was the last time you reversed a tree at work? You're probably more likely to run into a tree than to implement one.

And don't get me started on learning the ins and outs of mature technologies. They're _mature_! Just use it! And now everything's on the cloud so if any of these said technologies go down, simply make a call to your friendly AWS/GCP/Azure support engineer and they'll perform some black magic to make your problems disappear. Software engineering has never been simpler!

Well, I guess today I managed to run into a problem that allowed me to get some decent evidence arguing the contrary. The problem goes something like this:

So our team, like many other teams across tech companies, have oncall rotations. The duty of the personnel oncall is to

1. monitor system-level metrics
2. handle/delegate live issues
3. pray really hard your server doesn't go on strike

and so during my rotation, the powers that be decided to present me with a critical warning detailing that one of our DB instances was reaching > 85% CPU usage. 

My first thoughts were: "welp, I have no idea where to start..."

So I had to consult the more senior engineers and they were able to help + teach me how to get the issue resolved. 

Upon reflection it really struck me how spoilt I've been in my first two years as a software engineer. Like sure, I know concepts like "High Availability", "Replication", "Eventual Consistency" yada yada yada but once you dive just a little deeper, I'm totally lost. And I suppose this is where basics come in right? Understanding how the DB works on a deeper level would have given me a very good start point to start debugging. And a very good start point is indeed all you need to begin debugging because at least you know where to start looking and how to eliminate possible points of failures. 

Getting gud at your basics means understanding the basic facts that a certain piece of tech runs on, and through that you're able to quickly filter out all non-possibilities when dealing with issues or coming up with solutions.