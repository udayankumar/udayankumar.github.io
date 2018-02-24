---
layout: post
title: Heart Rate monitor using Raspberry Pi and Pulse sensor
comments: true
tags: [heart rate, raspberry, pulse, ADS1015]
---

There are several heart rate monitors available in the market but I wanted to make something cheap (already owned Raspberry Pi) and one that can be wall-powered for continuous use. Currently, it is sitting on my desk and I can measure my pulse anytime by putting my finger on the sensor. I am thinking about embedding it on my keyboards palm-rest and measure my heart rate continuously while I am using computer. 

![help]({{ site.url }}assets/images/raspberryPi-full.jpg)


## Equipment/Components

- Original Raspberry Pi, running raspbian-jessie 
- Pulse Sensor, un-official version off of Ebay  (< $4, free shipping)
- Analog to Digital converter, ADS1015, from [adafruit](https://www.adafruit.com/product/1083)
- Seven Breadboard Jumper cables, female to female

## Connections

I used digital GPIO ports on Raspberry Pi to connect the ADC. Then connected the pulse sensor to the ADC on channel A0 (make sure to check the channel in the code below). 

- The 3.3v (pins 1 and 17) from Raspberry pi provide power to ADC and Pulse Sensor. 
- The ADC, ADS1015,  uses I2C communication protocol, so the SDA pin on ADC connects to pin 3 on Raspberry Pi and SCL on ADC connects to Pin 5. - For ground, I have used pins 6 and 9 on Raspberry Pi. 

![help]({{ site.url }}assets/images/Connections.jpg)

![help]({{ site.url }}assets/images/PulseSensorConnections.jpg)

![help]({{ site.url }}assets/images/ADC_RASP_CONNECTION.png)


## Configuration 

Now we need to configure Raspberry Pi to enable I2C module. The detailed 
instructions are [here](https://learn.adafruit.com/adafruits-raspberry-pi-lesson-4-gpio-setup/configuring-i2c). 
Briefly said,  run `sudo raspi-config` and from the `Advance Options` enable I2C. Reboot

Also, install https://github.com/adafruit/Adafruit_Python_ADS1x15. Run `sudo pip install adafruit-ads1x15`


## Code

I used python to read and process data from the Pulse Sensor and produce Beats per Minute (BPM) or the Heart rate. The code is available in  [here](https://github.com/indolent/heart-rate-raspberry-pi). The code draws heavily from the [Pulse Sensor Arduino](https://github.com/WorldFamousElectronics/PulseSensor_Amped_Arduino) for detecting the pulse and [Adafruit Python ADS Code](https://github.com/adafruit/Adafruit_Python_ADS1X15) for reading the ADC.


Below is a screen-shot of an actual run of the code. The initial BPM is 109 but quickly converges to 74-76 range. 
![help]({{ site.url }}assets/images/screenShot_running.jpg)

## Accuracy 

I compared the pulse reading using pulse sensor on Galaxy S6. I used the default health app to get the pulse rate and compare it with the reading that I got from my setup. The measurements were done simultaneously on both of them, assuming that pulse rate in right hand is same as the pulse rate in the left :D. Almost every time the difference between the two was less than 3 BPM. If Samsung tested their sensor well, this sensor is pretty close! 


