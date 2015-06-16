# kegbot

all code relating to RPi-controlled kegerator monitoring system

##Other Sources

- Check out [Kegomatic](https://learn.adafruit.com/adafruit-keg-bot/overview) from Adafruit for flow metering ([repo](https://github.com/adafruit/Kegomatic)).
- Install a copy of [Raspbian](https://www.raspberrypi.org/downloads/) (Development of prototype is based on Debian Wheezy, kernel 3.18)
- Learn about [OLED displaying](https://learn.adafruit.com/adafruit-1-5-color-oled-breakout-board) from Adafruit.

##Purpose

This installation is meant to be run on a [Raspberry Pi 2 Model B](https://www.adafruit.com/products/2358), with N [flow meters](https://www.adafruit.com/product/828) and N [OLED displays](https://www.adafruit.com/product/1431) attached. The intent is to provide both a web-based API and analytics package and a clear display for the OLED panels (to be installed on or near the tap handles).

OLEDs may contain the following information (in order of importance)

- the beverage on tap
- Quantity remaining in the keg
- ABV (in case of alcohol)
- IBU (in case of beer)
- color

And the web-based API may collect and store some information too:

- beverages on tap
- how quickly each beverage is depleted
- beverage depletion as a function of type (beer, seltzer, cold brew) or style within that type (hefeweizen, IPA, porter)
- Estimated gas depletion (based on N volume carbonated/nitrogenated and M volume dispensed)

##Development

###TODO

This product needs a few pieces of software to do all the cool things we imagine:

- A simple sensor-display controller to take input from the flow meters, and output to log files (/mongo) and to OLED displays mounted on the tap handles.
- A web server to provide API endpoints for data input and output
- A web page to provide analytics and history data
- A Slack bot that connects to various API endpoints

In order to achieve these, I've broken them down into smaller steps:

- get the RPi a static IP address and port. Might need to be externally-visible, for Slack to ping it
- connect RPi to flow meter, read inputs
    - write inputs to mongo
    - update taphandle displays
- set up a node server
    - API endpoints for
        - "I just put this on tap"
        - "what's on tap?"
        - full data dump of history (paginated?)
    - webpages for
        - data entry
        - analytics  & history
- connect RPi to OLED displays
- Slack integration that pulls from the API ("@kegbot what's on tap?")
- pull more beverage data from RateBeer and/or BeerAdvocate (stuff like IBU, SRM. Still allow for manual input though, in case of homebrew)

###Data Model

I'm not very familiar with Mongo yet, but I've sketched out a data model in an accompanying .graffle file. Basically there are 4 important entities:

- KEG. This is a representation of some volume of liquid that is tapped, poured-from, and either kicked or untapped. It has intrinsic starting volume, and associates with its LIQUID and its POURs.
- POUR. This represents the event of someone pouring liquid from a keg. It has a start time, end time, and volume.
- LIQUID. This is a representation of a liquid that could be kegged. Perhaps mineral water, or beer, or cold-brew coffee. It contains various information about these kinds of liquid, including ABV, IBU, SRM and additives. It also has a name, link and description. It's worth noting that in some data formats it might be beneficial to subclass LIQUID into different kinds (ALCOHOL (BEER, WINE), COLDBREW, KOMBUCHA, SELTZER), but for the prototype that's probably not necessary. It associates to its KEGs and to a VENDOR
- VENDOR. Represents who made the liquid.

Other concepts of our kegerator setup will be baked into the software as settings, at least until the kegerator gets more advanced. For example, "CO2 Pressure" will remain as a setting until the CO2 distribution system is upgraded to support outlets at varying pressures. Likewise, the number of taps available will not change until the kegerator itself is upgraded or replaced.

It might be worth doing a thought excercise of how and why to model TAPs as a new entity in the model. Do we often untap a keg that isn't empty? Do we benefit from knowing when it was untapped?

It might also be good to implement some sort of error-correction on the flow meters. I'm not sure how accurate or precise they are, but if (for instance) we tap exactly 18 liters, then dispense it all and the flow meters read that we dispensed 20 L, we might need to calibrate them. Then again, when a keg kicks the flow meters will spin like crazy. We may want to plan for that, and implement some kind of high-pass filter on the sensor input (if the input is above a certain volumetric flow then assume it's gas and disregard the input. also notify Slack)

##Initial Deployment

To set up a brand new Raspberry Pi for using this package, first install Raspbian (Debian Wheezy), then add a whole slew of packages:

Node:

    $ wget http://node-arm.herokuapp.com/node_latest_armhf.deb
    $ sudo dpkg -i node_latest_armhf.deb
    $ which node # verify /usr/local/bin/node appears

Mongo:

    $ sudo apt-get install build-essential libboost-filesystem-dev libboost-program-options-dev libboost-system-dev libboost-thread-dev scons libboost-all-dev python-pymongo
    $ git clone https://github.com/skrabban/mongo-nonx86
    $ cd mongo-nonx86
    $ sudo scons
    $ sudo scons --prefix=/opt/mongo install
    $ sudo mkdir -p /data/db
    $ sudo adduser mongodb
    $ sudo chown -R mongodb /data
    $ sudo nano /home/mongodb/.bashrc  # export PATH="$PATH:/opt/mongo/bin"
    $ su mongodb
    $ which mongod # verify /opt/mongo/bin/mongod appears
    $ mongod # this runs mongo. verify that it opens properly. ctrl-c to close it.

NPM

    $ sudo apt-get install npm
    $ which npm

Download a sample project (optional)

    $ git clone https://github.com/jlouthan/actw-carpool.git
    $ cd actw-carpool/
    $ npm install
    $ # open a second terminal, log in as mongdb, run mongod, switch back to first terminal
    $ node server.js
    $ # open a browser and navigate to localhost:3088

TODO:

- start `mongod` on startup
- start web server on startup
- start sensor monitor and display on startup
