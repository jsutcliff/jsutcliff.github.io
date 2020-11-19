---
layout: post
title: Solar Powered Ground Sensor System
subtitle: Some pictures and details from my moped restoration project
thumbnail-img: /assets/img/Ground Sensor/thumbnail.jpg
share-img: /assets/img/Ground Sensor/thumbnail.jpg
tags: [Electronics]
---

One of the first projects I worked on as a college student was a system of sensors for measuring temperature and humidity for agriculture applications. The main goals of the project were to: 

Accurately measure temperature and humidity
Routinely transmit data over long distances to a ground station
Reliably operate off of solar power in harsh conditions
Keep manufacturing costs low in order to implement more sensors
My design for the sensor incorporates an ATmega88 as the main MCU. This was chosen over the more popular ATmega328 because of price. The simplicity of the project means that I would not need the additional flash memory of the 328. For temperature and humidity sensing, I chose to use Texas Instruments’ HDC1080. I designed the device to run off of a small AA sized lithium battery. A 6v solar panel was chosen to maintain battery charge. The MCP7383 charge controller was chosen to regulate the power flowing from solar panel to lithium battery in order to prevent over charging. Finally, the STX882 was chosen as a low cost, long range 433MHz transmitter module.
 
The schematic for the device can be seen below. Please note, some components part numbers are incorrectly labeled

<img src="/assets/img/Ground Sensor/Screen-Shot-2018-09-08-at-12.39.52-PM.png" class="rounded mx-auto d-block my-2">

From here I designed the PCB to be as compact as possible while remaining easy to assemble quickly. I also included headers for fast ISP programming. The final board design is below:

<img src="/assets/img/Ground Sensor/Screen-Shot-2018-09-08-at-12.22.07-PM.png" class="rounded mx-auto d-block my-2">

About 50 of these sensors were hand soldered and assembled for testing. They were placed throughout a cherry orchard in Northern Michigan. After over a year of testing 95% of the sensors were still functioning. Here is a picture of one 

<img src="/assets/img/Ground Sensor/IMG_1661-e1536425743492-350x467.jpg" class="rounded mx-auto d-block my-2">

Here is a small section of the data that was collected by one of the sensors:

<img src="/assets/img/Ground Sensor/Screen-Shot-2018-09-08-at-3.02.06-PM-1024x399.png" class="rounded mx-auto d-block my-2">

One drawback to my design is that if two sensors transmit data at the same time, the receiver often picks up corrupted data. In order to solve this, I implemented a simple random sleep timer algorithm. Most of the time, the sensors are in a low power sleep mode. The sensors sleep for a random amount of time so that the chances of two or more transmitting at the same time are slim. Here’s the code running on the ATmega88:

~~~ c
#include <VirtualWire.h>
#include <Wire.h>
#include "Vcc.h"
#include <avr/sleep.h>
#include <avr/power.h>
#include <avr/wdt.h>

#define ID 250
#define SLEEP 1

Vcc vcc(1.0);

int cycle = 0;

ISR(WDT_vect) {
}

void enterSleep(void) {
  set_sleep_mode(SLEEP_MODE_PWR_DOWN);
  sleep_enable();
  sleep_mode();

  sleep_disable();
  power_all_enable();
}

void setup() {
  Wire.begin();
  Wire.beginTransmission(0x40);
  Wire.write(0x02);
  Wire.write(0x0);
  Wire.write(0x0);
  Wire.endTransmission();

  vw_set_tx_pin(9);
  vw_set_rx_pin(2);
  vw_set_ptt_pin(3);
  vw_set_ptt_inverted(true);
  vw_setup(1000);

  MCUSR &= ~(1 << WDRF);
  WDTCSR |= (1 << WDCE) | (1 << WDE);
  WDTCSR = 1 << WDP0 | 1 << WDP3; /* 8.0 seconds */
  WDTCSR |= _BV(WDIE);

  cycle = 10;
}

void loop() {
  if (cycle >= SLEEP + random(2) - 1) {
    delay(random(5) * 200);

    cycle = 0;
    uint16_t temp = readData(0x00);
    uint16_t humd = readData(0x01);
    uint16_t batt = vcc.Read_Volts() * 10000;
    uint8_t packet[7];

    packet[0] = ID;
    packet[1] = highByte(temp);
    packet[2] = lowByte(temp);
    packet[3] = highByte(humd);
    packet[4] = lowByte(humd);
    packet[5] = highByte(batt);
    packet[6] = lowByte(batt);

    vw_send((uint8_t *)packet, 7);
    vw_wait_tx();
  }

  cycle++;
  enterSleep();
}

float getTemp() {
  uint16_t rawT = readData(0x00);
  return (rawT / pow(2, 16)) * 165 - 40;
}

float getHumd() {
  uint16_t rawH = readData(0x01);
  return (rawH / pow(2, 16)) * 100;
}

uint16_t readData(uint8_t pointer) {
  Wire.beginTransmission(0x40);
  Wire.write(pointer);
  Wire.endTransmission();

  delay(10);
  Wire.requestFrom(0x40, 2);

  byte msb = Wire.read();
  byte lsb = Wire.read();

  return msb << 8 | lsb;
}
~~~