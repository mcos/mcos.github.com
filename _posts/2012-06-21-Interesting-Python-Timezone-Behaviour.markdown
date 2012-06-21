---
layout: post
title: Interesting Python Timezone Behaviour
---

I had some interesting behaviour crop up in my code today. I was debugging a service I wrote for our application at work where users can schedule updates to be sent to their social media sites (Twitter, Facebook and LinkedIn). 

Quick note for anyone working with dates, times and timezones: you should always use UTC (<http://en.wikipedia.org/wiki/Coordinated_Universal_Time/>) as the standard timezone for all time-sensitive operations.

So, here is what I ran into: I was converting from the user's timezone to UTC. To do this, I use the very impressive pytz (<http://pytz.sourceforge.net/>) library and then using some native python datetime functionality to perform the conversion.

Here's the interesting part, when I did this:

	timestamp = datetime.strptime(input_timestamp, TIME_FORMAT)
	usertimestamp = timestamp.replace(tzinfo=usertz)
	utcusertimestamp = usertimestamp.astimezone(utc)
	return utcusertimestamp
	
I kept getting times that were off by one hour - i.e. they weren't taking into account Daylight Saving Time.

It turns out that the following line was the problem:

	usertimestamp = timestamp.replace(tzinfo=usertz)
	
Doing a replace of the timezone information on in the datetime object doesn't actually take into account Daylight Saving Time (<http://en.wikipedia.org/wiki/Daylight_saving_time/>). It turns out that I needed to do the conversion this way instead:

	usertimestamp = usertz.localize(timestamp)
		
Here's the final conversion code I used:

	timestamp = datetime.strptime(input_timestamp, TIME_FORMAT)
	usertimestamp = usertz.localize(timestamp)
	utcusertimestamp = usertimestamp.astimezone(utc)
	return utcusertimestamp

Let's hope this bug doesn't rear it's head again when the clocks go back later this year!