---
title: "Building a smart device without writing code"
date: 2022-12-22T09:59:07-05:00
draft: false
categories: ["Build Log"]
tags: ["esp8266", "homeassistant", "homelab", "iot", "smarthome"]
description: "A while ago (2019 according to [the repo](https://github.com/buzzsurfr/color-torch-device-esp8266)), I was learning about the Internet of Things (IoT) and we..."
image: ""
---

A while ago (2019 according to [the repo](https://github.com/buzzsurfr/color-torch-device-esp8266)), I was learning about the Internet of Things (IoT) and went through the process to prototype a *smart indicator light*. I made it communicate with AWS IoT where I could both change the color of the light on the device (and it would report to the cloud) as well as change the color in the cloud (and have the indicator light change).

Separately, I've also been spending more time on home automation. I have Home Assistant setup in my house and had been reading on ESPHome but haven't come up with a good test project--so I decided to repurpose my indicator light to work with ESPHome!

## Original Hardware

I wanted to see whether I could use the original hardware without modifications--mostly because instead of recreating the board, I just dusted it off...

![](https://theodorejsalvo.com/wp-content/uploads/2022/12/color-torch-esp8266-bb.png)Original prototype on breadboard

As the name may imply, ESPHome must be used by hardware supporting the ESP32 or ESP8266 (or RP2040, but that's for another time) chipset. I originally made this prototype on a Raspberry Pi but wanted to use a smaller form factor for portability and yet retain the ability to connect using Wi-Fi. I've been using the [Adafruit Feather HUZZAH](https://www.adafruit.com/product/2821) on a few projects, and had back then, so I stuck with it.

## Removing the Software

This project has been through a few iterations of software. I first started with python on the Raspberry Pi [[code]](https://github.com/buzzsurfr/color-torch-device-esp8266-old/blob/3980614c3175de0711e3808dc3abb226ea815dea/rgb_led.py). This would use the GPIO to control each leg of the RGB LED and would communicate with an IoT shadow in the cloud for the status. This meant AWS IoT was my *interfacing layer* and I could build a web-based GUI, Alexa skill, or mobile app to control this light.

When I switched to the Feather, it meant I needed to change programming languages. While MicroPython was an option, it still required the interpreter at runtime and didn't have the benefit of compiled code. I also wrote the program in C (using Arduino) but never bothered to become proficient with C's syntax, and ended up using JavaScript (using Mongoose OS). The trouble with all of these approaches is that the intent is simple (power to pin when condition) but writing the code becomes more difficult.

[ESPHome](https://esphome.io) has a different approach--you declare which components to use and provide the configuration for those components, then ESPHome compiles the modules and configuration together and produces an artifact that can be loaded onto the device. Anyone familiar with kubernetes will recognize this pattern: declare your intent in a resource file and let kubernetes build it. With ESPHome, I declare the light and which pins to use for output, and it builds the rest of it for me.

This is the configuration section for the LED in ESPHome:

```
`light:
  - platform: rgb
    id: torch_led
    name: "torch_light"
    red: led_red
    green: led_green
    blue: led_blue

output:
  - id: led_red
    platform: esp8266_pwm
    pin: GPIO14
    inverted: true
  - id: led_green
    platform: esp8266_pwm
    pin: GPIO12
    inverted: true
  - id: led_blue
    platform: esp8266_pwm
    pin: GPIO13
    inverted: true`
```

While this appears simple enough, I still did have to spend time learning the different values, but was able to piece it together by looking at the examples on ESPHome's website. 

There's an added benefit of "no code" solutions like ESPHome like the included features regarding Wi-Fi, Over The Air updates (OTA) and API integration. For every programming language, adding these features meant extra lines of code and setup, but ESPHome packages them as part of the configuration and build process. Much of the code in the earlier revisions was dedicated to Wi-Fi and API connectivity, with only a small section actually controlling the physical hardware. I added OTA when moving to ESPHome, and wrote less lines as a result of the switch!

## Migrating from AWS IoT to Home Assistant

While I was *able* to build interfaces that worked with IoT and the cloud, I wanted something that was already interconnected and didn't require me to build the integrations. I also prefer for control traffic to be kept as local as possible. While the cloud rarely goes offline, my internet connection is much more susceptible to outages which would render the light inoperable. With a local *brain*, I can save on both.

Like this project, I've built [Home Assistant](https://www.home-assistant.io) a few times over the years and have been slowly expanding it to incorporate the features I need. I have the [Home Assistant Podcast](https://hasspodcast.io) on my feed and it seems like everyone kept mentioning ESPHome and how it integrates with Home Assistant. Plus, ESPHome is easily run as an addon for Home Assistant. However, the best part is the interfacing is done for me! When I create the light in ESPHome, the device and entity show up in Home Assistant and include the interfacing.

![](https://theodorejsalvo.com/wp-content/uploads/2022/12/torch-light-homeassistant.png)The color and brightness controls come automatically in Home Assistant since I selected a RGB light as the platform in ESPHome.

Because I've integrated Home Assistant with Alexa, I also automatically get an Alexa interface through the Alexa app as well as voice control!

![](https://theodorejsalvo.com/wp-content/uploads/2022/12/IMG_4680-853x1024.jpeg)Alexa app also automatically can control the light.

## Okay, now what?

What's the point of a RGB light that's "smart-controlled"? The device isn't practical--but it's one of the first small board projects I built and have spent a lot of time with. I'd already completed this project, but was able to repurpose it and find out something new. So the point--is discovery.

> 

That's because great achievement has no road map. The X-Ray is pretty good, and so is penicillin, and neither were discovered with a practical objective in mind. I mean, when the electron was discovered in 1897, it was useless. And now we have an entire world run by electronics. Haydn and Mozart never studied the classics. They couldn't. They invented them.

Dr. Dalton Milgate, exerpt from the fictional series The West Wing S3E16

The project itself is a learning tool--now that I've made this work, I've also been able to add smart controls to a LEGO set with lights. Now as I'm automating my house, if I need a random motion sensor that communicates with MQTT then I can build it and integrate it quickly!

ESPHome also makes home automation more available to everyone. I speak more programming languages than languages--but not everyone does. Writing a config file is much easier than writing code and it cuts down on development time. Less time on software means that I also get more time on hardware!

![](https://theodorejsalvo.com/wp-content/uploads/2022/12/92d44ab4-5276-4592-96ba-01a294a170cc_text.gif)Image from [Yarn](https://getyarn.io/yarn-clip/92d44ab4-5276-4592-96ba-01a294a170cc)