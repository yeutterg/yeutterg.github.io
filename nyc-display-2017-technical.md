# NYC Window Display: Technical Details

This page details the technical implementation of the [2017 holiday window display](./nyc-display-2017) at Rockefeller Center.

## Table of Contents
* [Overview](#overview)
* [Hardware](#hardware)
* [Backend](#backend)

## <a name="overview"></a> Overview

The client requested a lighting overlay to their chocolate window display. The lighting was to change color via input from Twitter. Changes could be viewed online via a YouTube Live stream.

This project involved hardware design and specification, as well as programming in two languages and several APIs. 

A specific type of LED lighting was selected that enabled a wide range of effects via computer control. The LED lighting strands were durable and did not generate enough heat to melt chocolate.

## <a name="hardware"></a> Hardware

The LEDs used in this project are commonly known as [pixels](https://www.doityourselfchristmas.com/wiki/index.php?title=Different_Styles_of_Pixels). Every pixel node (or light on a strand of lights) can create a rainbow of colors independently from its nearest neighbor. 

Each LED node contains an RGB (short for red, green, blue) LED. The three colors can be mixed to create virtually any other color, such as teal, white, pink, or yellow. 8 strands of 50 LED pixels were used in the final implementation, for a total of 400 LED pixel nodes.

Each node contains a controller that can modulate the color of the LED. The controller found in each pixel node is the [WS2811 from WorldSemi](https://cdn-shop.adafruit.com/datasheets/WS2811.pdf) (or equivalent). The WS2811 has its own "language" of sorts, so the control gear must be able to output the signal the WS2811 chip understands. Thankfully, there is a library for the [Arduino](https://www.arduino.cc/) called [FastLED](http://fastled.io/) that can control WS2811 pixels.

Most Arduinos will struggle to control more than 200 or so WS2811 pixels. At a higher quantity, the effects will look choppy. Thankfully, there is one Arduino, the [Due](https://www.arduino.cc/en/Guide/ArduinoDue), that is much faster and also allows parallel output on up to 8 channels. (I actually used a [Due clone](https://www.amazon.com/OSOYOO-Compatible-Shield-Module-Arduino/dp/B010SCWGE2/ref=sr_1_3?ie=UTF8&qid=1515126819&sr=8-3&keywords=arduino+due) that is practically identical.)

However, the Arduino Due can only output 3.3 volts on its data channels, while the level required by the pixels is 5 volts. To get around this issue, a daughterboard was built for the Due. It contains [level shifters](https://en.wikipedia.org/wiki/Level_shifter) that change the signals from 3.3 volts to 5 volts. This header also contains screw terminals for all data connections. Screw terminals are more robust than the female headers found on most Arduinos.

The code on the Arduino listens for color commands sent over the computer's serial port. When a new color is received, the color palette defined in the Arduino code changes. All effects, including twinkle, snowfall, and fades, are generated on the Arduino.

You can see the code used on the Arduino [here](https://gist.github.com/yeutterg/2909ab05c726c4f8a4ab3f417c720d17). Warning: it's hacky.

Pixel nodes are not the easiest to wire. Because power drops over the length of a strand, power needs to be ["injected"](https://www.doityourselfchristmas.com/wiki/index.php?title=Power_Injection) at multiple places along the strand. 

## <a name="backend"></a> Backend

The backend was implemented in [Python](https://www.python.org/). It interacted with the Twitter API and an access control document in Google Sheets. New color commands were output over the serial port to the Arduino.

Every hour, a new user was granted tree control. Administrators could adjust the queue on a private document on Google Sheets. When a new timeslot started, a message was sent to the user on Twitter that it was their turn. 

The active user could send any message back to the tree. The first word in the message that matched one of the valid colors would trigger the color output. A message containing no color would be ignored.

The user could also send the commands "help" and "colors." The former responded with some instructions, while the latter responded with a list of allowed colors.

At the completion of the session, the user was sent a thank you message.

The backend also generated a random color every minute, so passersby could experiance a constantly changing display.

Originally, the Twitter streaming data API was used. However, the project site had signficiant internet connectivity issues, so that part of the code was rewritten to support the Twitter REST API. The advantages are much lower bandwidth and better reliability on a finicky connection. The disadvantage is that the response time falls to intervals of several seconds. The tradeoff was deemed acceptable.

The backend wrote color commands over serial to the microcontroller. The colors were sent as single chars. For example, if 'o' was sent, the microcontroller would interpret that as orange.

The following Python libraries were utilized: 
* [pySerial](https://pythonhosted.org/pyserial/) for communication over the serial port
* [Tweepy](http://www.tweepy.org/) for bidirectional communication with the Twitter API
* [gspread](https://github.com/burnash/gspread) for reading and writing to Google Sheets 