# IoT-RaspberryPi-Scale
Implementing a RaspberryPi as an IoT device to weigh objects and show the weight on a web app

## Table of Contents
* [About the project](#about-the-project)
* [Setup Raspberry Pi](#setup-raspberry-pi)

## About the project
The **aim** of this project is to present a way to use a Raspberry Pi Model 1 and a load cell as a scale. The Pi should then be configured as an IoT device via Azure IoT Hub. All the data streamed to the IoT Hub should then be presented on a ASP.NET Web App by using Azure Function.
All the source code referenced in this project can be found either in this repository or in one of the following repositories:
* [Azure Function](https://github.com/jekolbe/IoT-RaspberryPi-Scale-Azure-Function)
* [ASP.NET Web App](https://github.com/jekolbe/IoT-RaspberryPi-Scale-WebApp)

## Setup Raspberry Pi
First of all we have to setup the Pi to weigh objects. To do so I am referring to this [tutorial](https://tutorials-raspberrypi.de/raspberry-pi-waage-bauen-gewichtssensor-hx711/).
### Needed materials
We need the following:
* [Load Cell & HX711 sensor](https://www.amazon.de/gp/product/B07L82YWPV/ref=ppx_yo_dt_b_asin_title_o05_s01?ie=UTF8&psc=1)
* [Jumper cable & breadboard](https://www.amazon.de/gp/product/B078JGQKWP/ref=ppx_yo_dt_b_asin_title_o05_s00?ie=UTF8&psc=1)
* 2 boards
* M4 & M5 screws & nuts with about 5 cm length
* Raspberry Pi (Model 1 or newer)

### Mount the load cell
You should screw the load cell to the bottom part of your setup.

<img src="https://drive.google.com/uc?export=view&id=1vsJ1lDNEGJMJXQbsmKYEYWTfHEFyitwR" width="300">

After this you can also mount the top part and it should look something like this.

<img src="https://drive.google.com/uc?export=view&id=1vs3QUzlrAaI5DuxFxXhAz5Sk0sxeEGcF" width="300">

### Connecting the HX711
Now we somehow have to connect the four cables of the HX711 to the Pi via the breadboard. Just follow these connection pattern:
* Red: E+
* Black: E-
* Green: A-
* White: A+

We can leave B- and B+ empty.

<img src="https://drive.google.com/uc?export=view&id=1w2gtAXfPY_oxsHO4_tTr6H4D04rnA-_2" width="300">

At this point you have to use four jumper cables connecting the HX711 and the pi. You can also use different kind of colors.
* Black: GND to pin 6 (Ground)
* Orange: DT to pin 12 (GPIO 18)
* Yellow: SCK to pin 16 (GPIO 23)
* Red: VCC to pin 2 (+5V)
You can find a scheme of the pins for the Pi 1 [here](https://developer-blog.net/wp-content/uploads/2013/09/raspberry-pi-rev2-gpio-pinout.jpg). If you are using a newer Pi with more Pins then just look up and connect to orange (DT) and yellow (SCK) to empty pins. Later on you have to change the pin number in the code.

### Configuring the software
In order to weigh an object we are going to use a python library. First of all you have to clone this library on the Pi:
```
git clone https://github.com/tatobari/hx711py
```
You will find a `example.py` which you can also find as a final version in this repository in order to assist you.
You can execute this script on your raspberry pi by doing the following in the command line.
```
sudo python example.py
```
Before adding some weight you will see some values. You can just ignore them and add a object of which you know the weight. As soon as you add the weight to the board you can divide the appearing values by your weight in gram. In my case I used a glas of honey with about 400 gram
For example: -310000 ÷ 400 = -775
Now I know that -775 is my reference unit. We can now go on by editing the `example.py`
```
cd hx711py
sudo nano example.py
```
Search for the line which says `referenceunit = 1` and change the value to your reference unit.
```
referenceunit = -775
```
You also have to pay attention to line which says `hx = HX711(5, 6)`. The numbers 5 and 6 indicate which GPIO are used. In my case I had to change the numbers to 18 and 23 as I described earlier.
