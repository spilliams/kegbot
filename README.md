# kegbot
all code relating to RPi-controlled kegerator monitoring system

##Other Sources

Check out [Kegomatic](https://learn.adafruit.com/adafruit-keg-bot/overview) from Adafruit for flow metering ([repo](https://github.com/adafruit/Kegomatic)).

Install a copy of [Raspbian](https://www.raspberrypi.org/downloads/).

Learn about [OLED displaying](https://learn.adafruit.com/adafruit-1-5-color-oled-breakout-board) from Adafruit.

##Purpose

This installation is meant to be run on a [Raspberry Pi 2 Model B](https://www.adafruit.com/products/2358), with N [flow meters](https://www.adafruit.com/product/828) and N [OLED displays](https://www.adafruit.com/product/1431) attached. The intent is to provide both a web-based API and analytics package and a clear display for the OLED panels (to be installed on or near the tap handles).

OLEDs may contain the following information (in order of importance)

- the beverage on tap
- Quantity remaining in the keg
- ABV
- IBU
- SRM

And the web-based API may collect and store some information too:

- beverages on tap
- how quickly each beverage is depleted
- beverage depletion as a function of type (beer, seltzer, cold brew) or style within that type (hefeweizen, IPA, porter)
- Estimated gas depletion (based on N volume carbonated/nitrogenated and M volume dispensed)

##Development

has yet to start, but when it does I intend to get the RPi working in probably the following order

- connect to flow meter, read inputs
- set up web server with simple API endpoint (or even perhaps pages) for data entry
- ability to tell the RPi what was just put on tap
- RPi to deduct and log amounts poured
- connect RPi to OLED displays
- update displays periodically to show relevant beverage data
- Slack integration that pulls from the API ("@kegbot what's on tap?")
- pull more beverage data from RateBeer and/or BeerAdvocate (stuff like IBU, SRM. Still allow for manual input though, in case of homebrew)
- data analytics presented by a web page that pulls from the API
