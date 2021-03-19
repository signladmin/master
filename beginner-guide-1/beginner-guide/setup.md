---
description: In this tutorial we walk through basic Raspberry Pi and Linux Set Up
---

# Setting up the Raspberry Pi

## Summary <a id="h.vrhvb96nxxe9"></a>

1. Download an Operating system \(OS\). For this tutorial, we will be using the Raspberry Pi organization's Debian “buster” image.
2. Install Raspberry Pi OS using Raspberry Pi Imager 
3. Flash the OS onto the SD card
4. Boot up the Pi and configure the settings 
5. Insert external SSD and copy the SD card to it
6. Shutdown and reboot from SSD



{% hint style="info" %}
### DON’T SKIP STEPS 😁
{% endhint %}

### _**Part One:**_

###  🍓Installing the Raspberry Pi Debian "buster" OS on to the SD CARD 🥧 <a id="h.lpv6ciisjqp3"></a>

We are going to now download the latest official release of Raspberry Pi 64bit Debian OS. This is the official Linux 64bit OS distribution that is designed for the Raspberry Pi and its arm64 CPU. This makes it stable and very easy to get started with the Raspberry Pi 

**1. Download the Debian “buster” Raspberry Pi 64bit OS image right -&gt;** [**here**](https://downloads.raspberrypi.org/raspios_arm64/images/raspios_arm64-2020-08-24/2020-08-20-raspios-buster-arm64.zip) **and save it in an accessible location for now on your computer.**  


**2. Next, download the raspberry pi imager software that we will use in order to install the OS onto our  Raspberry Pi. This software is located on the** [**Raspberry Pi website**](https://www.raspberrypi.org/software/) **download the correct version for your computer.** 

![](../../.gitbook/assets/screen-shot-2021-03-12-at-5.36.30-pm.png)

 **3. Insert the SD card into your computer and open the "Raspberry Pi imager".**

*  **Click on "CHOOSE OS"  then find the "2020-08-20-raspios-buster-arm64.zip" file you have downloaded in step \(1\) of this tutorial and select it.** 
* **Next, click on the "CHOOSE SD" and find the SD card you inserted into the computer** 
* **Now, the "WRITE" button will appear and you can click on it to begin writing/verifying the OS onto the SD card.**  
* **Finally, once it has finished the writing/verifying process, you will see a pop-up window saying that the OS was successfully written to the SD card, click "CONTINUE" and remove your SD card from the computer.** 

{% hint style="info" %}
#### **If you still have issues following the written instructions** [**here**](https://www.youtube.com/watch?v=J024soVgEeM) **is a short video of this process 😎**
{% endhint %}

### Part 2:

### Setting Up the Raspberry Pi for USB-Boot and SSH \(headless\) login

The first thing that we want to do is get the Raspberry Pi booted up to the RaspiOS desktop screen. To do this we will need to plug the mini HDMI to HDMI cable into the Raspberry Pi 4 \(make sure to plug into the mini HDMI port nearest to the USB-C power supply port on the RPi4\). Then we need to insert the SD Card into the slot on the bottom of the Raspberry Pi, then we should plugin our keyboard and mouse into the Pi. Lastly, we can now plug in our USB-C Raspberry Pi 4 power adapter into the Pi and it will begin to boot up.







### 



