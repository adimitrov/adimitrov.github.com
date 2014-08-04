---
layout: post
title: "SSH: Attack of the clones"
category: posts
---

Yesterday I was browsing trough logstash and noticed some failed ssh login attempts, so I decided to check the logs for the last month to see whats going on.
I discovered that one of the servers that has ssh open has a very large amount of failed attempts.
This server is the only one (of loggators servers) that has ssh open (not firewalled). Thats probably why its the only one that was targeted.
I decided to analyse the logs to exercise my bash skills and out of curiosity.

First wanted to know how many failed attempts there were. Which is pretty straightforward using grep and wc.

{% highlight bash linenos cssclass=hl %}
% grep -o "Failed password" auth.log* | wc -l
   87414
% wc -l auth.log*
    ...
   242027 total
{% endhighlight %}



Then I wanted to find out from where the attempts were coming.
I filtered the IPs and ran them through geoiplookup and uniq to find this list:

{% highlight bash cssclass=hl %}
grep -o -h "Failed password for \(invalid user \)\?[a-z]* from [0-9\.]*" auth.log* | \
awk '{print $(NF)}' | \
sort | \
uniq | \
xargs -n 1 geoiplookup | \
sort | \
uniq -c | \
sort -r
{% endhighlight %}



{% gist adimitrov/0816c6a5d67b7ba299b1 countries.txt %}



Most of the IPs are from China, or more precisely "China Telecom Huzhou".

So far so good. Next I wanted to know what users they were attempting to login as. The list surprised me as it was very big, which is why I made it as a [gist](https://gist.github.com/adimitrov/0816c6a5d67b7ba299b1).

{% highlight bash cssclass=hl %}
grep -o -h "Failed password for \(invalid user \)\?[a-z]* from [0-9\.]*" auth.log* | \
awk '{print $(NF-2)}' | \
sort | \
uniq -c | \
sort -r
{% endhighlight %}

This is a list of the top attempted users:

{% gist adimitrov/0816c6a5d67b7ba299b1 users_head.txt %}

The top results here is no surprise, but the fact that there are so many others is. Some of them don't even make sense.


After all this I disabled ssh password login, because like everyone else I use a public key to login to the server.
To do this changed a few settings as suggested by some random articles around the internet :)

{% highlight bash cssclass=hl %}
PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM no
{% endhighlight %}

If you are wondering what is the purpose of this kind of brute force attacks, here is a [good article](http://blog.sucuri.net/2013/07/ssh-brute-force-the-10-year-old-attack-that-still-persists.html) that explains everyting.
It even goes as far as showing what the attacker does when they log in.

Also you should check out [Sysadmincasts](http://sysadmincasts.com). They have some very useful casts on the topic.
