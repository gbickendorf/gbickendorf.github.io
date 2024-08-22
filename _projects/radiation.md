---
layout: page
title: Raspberry Pi Geiger Counter
description: Combining a Raspberry Pi with a cheap self-soldered Geiger counter
img: assets/img/rad.jpg
importance: 8
category: fun
related_publications: false
---

## Introduction
Radiation is a fascinating topic. Some forms are invisible, but the exposure is still unhealthy. In some region, the basement is prone to accumulating radon gas, which can be breathed in. When the atoms decay inside the lung, this raises the lung cancer risk significantly. Given this and the general fascination of measuring invisible effects I wanted to build a Geiger counter and maybe integrate it into my Raspberry Pi home server.

## Principle behind a Geiger Counter
The main component of our Geiger counter is the Geiger tube. This tube is cheap and readily available. The glass tube is filled with a mixture of gasses (often a noble and a halogen gas). Through the middle there is a thin metal wire forming the anode. The inner wall of the tube is thinly coated with a conducting metal which is connected to the cathode. Between the anode and cathode lies a potential difference of several hundreds of volts. When a particle of ionizing radiation enters the tube, it ionized the gas along its path. The voltage accelerates the free electrons towards the anode which ionizes other atoms along the way. This results in an avalanche that ionizes almost the entire gas. Once the electrons reach the anode, there is a small current. This current can be amplified and subsequently measured. When the pulse is fed into a speaker, it produces the well known "click"-sound.

## Open Source Circuit
The diagram for the circuit that produces the high voltage, measures the current pulse and produces the sound can be found <a href="https://github.com/SensorsIot/Geiger-Counter-RadiationD-v1.1-CAJOE-/blob/master/Geiger%20Counter%20Diagram.pdf">online</a>. There are also shops that sell the printed circuit board with a small bag of components such as the M4011 tube, resistors, capacitors, chips and transistors for soldering yourself. This set usually cost less than 30€.

After soldering everything, the set looks like the following image.
<div class="row justify-content-sm-center">
  <div class="col-sm-6 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/rad.jpg" title="Geiger counter" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
When a battery is connected, it indeed detects one particle every 10 seconds or so. This is what is expected with the normal background radiation in Bonn.

## Connecting to the Raspberry
The pins, shown in the upper left corner of the image above allow to connect the Geiger counter to other devices. Two pins are power connections, while the third sends a 5V pulse when the detector triggers. My Raspberry Pi 3 Model A+ has a bunch of general <a href="https://datasheets.raspberrypi.com/rpi3/raspberry-pi-3-a-plus-reduced-schematics.pdf">IO pins (GPIO)</a> that allow exactly this connection. Connecting the power supply to pins 1 and 6 allows the detector to be supplied by the Raspberry Pi alone. Delivering the pulse to pin 7 let the PI read the detector response.
<div class="row justify-content-sm-center">
  <div class="col-sm-6 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/raspi" title="Raspi plus Geiger counter" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
Using the `GPIO` module of the `RPi` python package supplies everything one needs to read the detector. 

    GPIO.setmode(GPIO.BCM)
    GPIO.setup(4, GPIO.IN)
    GPIO.add_event_detect(4,GPIO.FALLING, callback=decay)

The first line sets the numbering scheme by which we label the pins in the python code. `BCM` means that we refer to the internal labels, rather than the physical circuit-board numbers. The internal name for pin 7 is `GPIO04`.
The second line tells the Raspberry Pi to read from the pin, rather than setting the output voltage. The last line tells it to set an interrupt. When the Raspberry Pi detects a falling flank on our pin, it calls the function `decay`. In this function, we simply save the timestamp of each detected particle. 

Since the main body of the program has to simply wait for a decay, it is a forever-loop of `time.sleep(1)` because everything else is managed by the interrupt.
The complete python code is as simple as the following:

    import RPi.GPIO as GPIO
    import time

    def decay(channel):
        """Called every time the detector send a falling flank"""
        """Implement whatever you like"""

    GPIO.setmode(GPIO.BCM)
    GPIO.setup(4, GPIO.IN)
    GPIO.add_event_detect(4,GPIO.FALLING, callback=decay)
    while True:
            time.sleep(1)

There is also a tube specific factor to roughly translate the counts per minute (CPM) to µSv/h, which is for the M4011 tube is 153.8 CPM h/µSv.
Here is the plot of both CPM and the dose:
<div class="row justify-content-sm-center">
  <div class="col-sm-6 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/fig.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
Converting the average dose ~0.125µSv/h to the anual dose gets us an exposure of 1.095µSv/a which puts us on the low end of the <a href="https://www.bfs.de/EN/topics/ion/environment/natural-radiation/natural-radiation.html">expected range</a> between 1 and 3 µSv/a. This is not too surprising, since a large part of the average natural exposure comes from radon gas. Elevated radon levels are common where granite is the bedrock, which is not the case in Bonn.