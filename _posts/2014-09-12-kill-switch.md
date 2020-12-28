---
layout: post
title: RC Kill Switch
subtitle: A custom solution for my first gas-powered airplane
thumbnail-img: /assets/img/Kill Switch/thumbnail.jpg
share-img: /assets/img/Kill Switch/thumbnail.jpg
tags: [RC, Electronics]
---

Thought I'd share what I've been working on lately. Recently, the club I fly at made it a rule that you
must be able to kill your engine remotely; choking the engine to kill it was deemed unreliable and dangerous. This means
that ignition to the engine must completely stop on demand. For most guys who use electronic ignition systems, this is
no problem. However, the engine that I'm planning to use has a magneto ignition. This means that a magnet attached to
the flywheel passes by a coil to generate the electricity for the spark. This ignition is particularly hard to stop
because as long as the engine is spinning, sparks will continuously be created. Luckily the magneto on my engine has a
wire that when grounded to the case of the engine will short out the sparks and effectively kill the engine. So, all I
had to do after I figured this out was create a device that shorts this wire to the case whenever it loses power or
signal. Here's the final product:

<iframe class="rounded mx-auto d-block my-2" width="560" height="315" src="https://www.youtube.com/embed/eOTCOjDk6io" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
                
<img src="/assets/img/Kill Switch/2014-04-28-16-54-28-jpg_1398718625.jpg" class="rounded mx-auto d-block my-2">
<img src="/assets/img/Kill Switch/2014-04-28-16-56-01-jpg_1398718628.jpg" class="rounded mx-auto d-block my-2">
<img src="/assets/img/Kill Switch/2014-04-29-16-45-33-jpg_1398803900.jpg" class="rounded mx-auto d-block my-2">

So how does it work? Well, let's start with the switching. The white component on the board is actually a very small, low power relay. I have it wired in so that when there's no power to the board, it shorts out the engine and stops it. This is so that if the plane loses power in flight, it won't run out of control. The other major component on the board it the microcontroller. I used a surface mount attiny85. This chip reads the radio signal coming from your receiver and switches the relay on only when it gets a valid signal. There's also a blue led on the bottom to indicate when your engine is "hot".

<img src="/assets/img/Kill Switch/screen-shot-2014-04-28-at-5-16-31-pm-png_1398719228.jpg" class="rounded mx-auto d-block my-2">
<img src="/assets/img/Kill Switch/screen-shot-2014-04-28-at-5-17-20-pm-png_1398719229.jpg" class="rounded mx-auto d-block my-2">   

Here's the code that runs on it:

~~~ c
int relayPin = 4;
int signalPin = 3;
int ledPin = 2;

void setup() {
  pinMode(relayPin, OUTPUT);
  pinMode(ledPin, OUTPUT);
}

void loop() {
  int signal = pulseIn(signalPin, HIGH, 100000);

  if(signal > 1500 || signal == 0) {
    digitalWrite(relayPin, LOW);
    digitalWrite(ledPin, LOW);
  } else {
    digitalWrite(relayPin, HIGH);
    digitalWrite(ledPin, HIGH);
  }
}
~~~