# kegbot

all code relating to RPi-controlled kegerator monitoring system

##Other Sources

Check out [Kegomatic](https://learn.adafruit.com/adafruit-keg-bot/overview) from Adafruit for flow metering ([repo](https://github.com/adafruit/Kegomatic)).

Install a copy of [Raspbian](https://www.raspberrypi.org/downloads/) (Development of prototype is based on Debian Wheezy, kernel 3.18)

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

###TODO

- connect to flow meter, read inputs
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
    - update displays
- Slack integration that pulls from the API ("@kegbot what's on tap?")
- pull more beverage data from RateBeer and/or BeerAdvocate (stuff like IBU, SRM. Still allow for manual input though, in case of homebrew)

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
