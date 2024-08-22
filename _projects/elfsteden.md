---
layout: page
title: From Datamining to a Raceplan
description: How to become faster at rowing with Python
img: assets/img/elfsteden.jpeg
importance: 8
category: fun
related_publications: false
---

## Synopsis
Here I present my project that uses public GPS data points of a rowing regatta. The dataset can be used to extract useful positions and spots along the track from the winning team.

## The Event
The event we will be talking about is the <a href="https://elfstedenroeimarathon.nl/en/">elfsteden roeimarathon</a> in the northern parts of the Netherlands. Once upon a time, when the canals that are found everywhere in the Netherlands still froze over in the winter, there was a ice skating event. The race span almost 200 km and went through eleven cities, hence the name.Nowadays,s that the canals do not freeze anymore, the event takes other shapes such as a bicycle race and a rowing regatta. The latter is the event we will analyze here.
<div class="row justify-content-sm-center">
  <div class="col-sm-6 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/elfsteden.jpeg" title="Elfsteden panorama" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
The course is around 210 km long and the majority of teams are expected to take between 16 and 20 hours to arrive at the finish line. The only boat-class that is allowed to start is the coxed double, i.e. two rowers and one cox. The three team sizes are either just the three, six or up to twelve rowers. The six and twelve rowers teams follow their team boat by car, waiting to swap the athletes at fixed points. Each team finds suitable points along the route to swap athletes either by analyzing satellite images or by asking around. Not every team gives up its best swapping points willingly, as congestion would significantly slow down the swap.

## Getting the Data
During the race all boats carry a GPS tracker that transmits the position in real-time, such that the complete race can be followed by the  <a href="https://tourmonitor.eu/elfstedenroeimarathon/esrm2023/">tour-monitor</a> website. As the race progresses, the website takes continuously longer to load and render. As any reasonable data scientist would do, I checked the source code. It turns out, it loads numerous large files, indicating that rendering of the course is done completely on the client site. The majority of the files contain thousands of entries like

    {"id":"552957","d_id":"1413","t":"1684555152000","lat":"52.881303","lng":"5.498525","s":"3.28"}

This json format can easily be read by a simple python script, making data mining a breeze.
Thankfully the software engineer that set this whole thing up did not obfuscate anything. "lat" and "lng" does look an awful lot like latitude and longitude in degrees which also fits the location of the race. The next step is to make sense of the other fields. "s" might be the current speed in m/s, but we will ignore that. "t" looks like the time given in the unix epoch. Converting this to human time: Yes we are golden. The date and time is precisely during the race. The last two fields "id" and "d_id" are yet to be uncovered. As the former is only found once in each file, this is most likely the id of the GPS point, therefore irrelevant for us. The latter is found repeatedly, hinting that this might be unique to each team. In fact, the very first data file the website pulls contains fields like

    {"id":"1536","ca_id":"1304","nm":"Speedy BONNzales","b_nm":"","c_nm":"","stp":"6","q":"0"}

"nm" is definitely our team name, but the "id" 1536 does not appear in the GPS files. Luckily there is another JSON field "device" that looks like

    {"id":"1413","tm_id":"1536","nm":"WW-006 (2021) (juliet)","mo":"","so":""}

In other words: our team carries the device with id 1413. This id is also found by the thousands in the GPS files. Let's group all GPS points belonging to a single id. Now that we understand the data, let's see what it contains!

## First sanity checks
The first thing to check is the latitude and longitude. Let's plot the values of our team:
<div class="row justify-content-sm-center">
  <div class="col-sm-6 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/course.jpg" title="Elfsteden course" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
That is exactly what the course looks like, although slightly distorted due to weird scaling. The course starts roughly at (53.2 , 5.8), traversing the small upper circle clockwise, to and from Dokkum at the top right, followed by the large circle and finishing a few kilometers from the start.

## Extracting Speed
Using some trigonometry and the radius of the Earth allows converting the angular coordinates to meters. The distance between two GPS signals combined with the timestamps allows calculating the instantaneous speed. Lastly, converting the epoch to human time allows comparing to the events that we have experienced. Let's plot this!
<div class="row justify-content-sm-center">
  <div class="col-sm-12 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/speed.jpg" title="Speed graph" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
So, what are the features we are seeing now? Clearly, the peaks of 18 km/h are noise. This happens when the GPS-signal is weak and suddenly jumps. Secondly, you can observe fatigue each single crew. For example at 22:00 a friend and me had our turn in the boat. Even though we were the fastest crew overall, you can clearly observe that we slow down until the next change at ~23:00. One hour of fast rowing tires you out!

Also, at the start of our turn there is a lot of noise, both upward and downward. The reason is quite beautiful in person: neat little bridges that we went under. Here the GPS tracker lost contact to the satellites and gave bad readings.

## Spying on the others
The most interesting part of the data are the real dips in the speed. For what reason do you have to come to a complete stop? Right, changing the crew!

This is very useful! Let us see where the fastest team of that year changed. Maybe there are some good spots for us to use. As an example, let's pick the dip in speed at 4:50 in the morning. The coordinates can be found by some more python manipulation to be (52.873161,5.540571). Let's see where this is on the <a href="https://www.openstreetmap.org/?mlat=52.87307&mlon=5.54054#map=18/52.87307/5.54054">OpenStreetMap</a>. 

<div class="row justify-content-sm-center">
  <div class="col-sm-8 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/map.png" title="Ovenstreetmap of a great spot" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
Clearly, the team changes right besides the bridge, easily reached by the team car. This was not a spot we were aware of.   

## Future plans
Certainly, the next time we start at this regatta, we will have a lot more spots at our disposal. One could even assign a congestion risk to each spot, by counting teams that seem to change at each respective spot. Another possibility to use the data would be to check which spots are easy to use. Sometimes changing spots are hard to reach, overgrown or at weird angles. All these make the change take longer. Conversely, if the dip in speed is narrow and only encompasses few GPS-pings it seems to be a very practical place to change.