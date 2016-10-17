# WNM499: Midterm

## Title: Xmas LED Show
### [Optional] Logo

## Problem & Solution
In the late 1800's Thomas Edison invented the incandescent light bulb. From this invention the idea for Christmas lights soon followed.
The earliest lights were not programmable and simply always stayed on. Light shows with programmable logic arrived during the 1960's 
via the advent of the microcontroller. Today, you can purchase christmas lights almost everywhere however they come pre-programmed 
with only a handful of patterns. The manufactures of christmas lights do not allow the consumer to reprogram their products.

As programmable software is uniquitous is all aspects of our lives, one can't help but wonder at the possibility of extending software
towards making a personalized connection. When you buy a manufactured product that runs some amount of software, the end-user gets stuck
with non-upgradable software. Sometimes we find software that gives pre-programmable options to fit our personality, however the options
are never enough.

With the growing community of programmable internet-enabled devices and inexpensive costs to access lights, a custom setup
can be achieved to allow the end-user to program their own christmas lights. The Raspberry Pi is a high-powered mini-computer the size
of a credit card with built-in wireless internet connectivity and digitally programmable inputs and outputs. The Pi's wireless
connectivity will allow for controlling via the internet and will be accomplished using a mobile app built with Ionic. The system will be
built for private networks and will rely on a wireless router. In this system the mobile app will allow programmable light
patterns with almost an infinite amount of possibilities.

![High-Level System Design](https://raw.githubusercontent.com/ComputerEnchiladas/arduino/master/RGB-LED-via-Serial/High-Level%20Design.jpg)

The design above shows the basic setup. The user begins by using the mobile app and explores programming the led's. When the user has 
defined a light pattern then data is transmitted via web sockets through the router and to a socket server in the Raspberry Pi. The Pi
power's an individual LED to provide visual feedback, and then propagates the signal via Serial communication to a microcontroller;
Arduino Uno. The Arduino will drive an RGB-LED strip which can be extended.

## Prototype
![Prototype](https://raw.githubusercontent.com/ComputerEnchiladas/arduino/master/RGB-LED-via-Serial/Setup.png)
The prototype above will consist of getting the system setup with just one LED. The LED is RGB which means it can provide multiple colors.
With the mobile app the single LED will be programmed with multiple patterns and manage the changing of patterns.

## Development
### Provisioning
The RaspberryPi is a mini-computer which needs to be provisioned. Here are the steps:
* Using a personal laptop download [NOOBS](https://www.raspberrypi.org/downloads/noobs/)
* Extract the contents into a micro-sd card at least 8Gb
* Open recovery.cmdline and add `silentinstall` at the end
* Put micro-sd card into Pi, turn on, and wait 30 minutes
* With laptop connected to wireless internet, chose to share internet via ethernet port
* Connect your laptop computer's ethernet into the Pi's ethernet
* SSH into the Pi with your laptop's terminal. `ssh pi@192.168.2.2` (if that doesn't work try `arp -a` to find the Pi's IP address)
* Using the terminal to control the Pi, download the remote desktop tool. `sudo apt-get install tightvncserver`
* Start VNCServer by typing `vncserver`. The first time you launch it will ask you to create a password.
* On your laptop, download [VNC Viewer](https://www.realvnc.com/download/viewer/) and use the following address to login. 192.168.2.2:1
* Using the Raspbian Desktop, open up the terminal, then install and configure the following:
   
   * git
   * node
   * vim

The Arduino is a microcontroller which needs to be programmed. Here are the steps:
* Using a personal laptop download [Arduino](https://www.arduino.cc/en/Main/Software)
* Connect your Arduino to the laptop
* Open the software and select File > Examples > Basics > Blink
* Click on the Right Arrow (->) to flash (program) the board with a simple blink program

### Integrating
The Arduino will be programmed to receive commands via Serial. Commands will include which LED, followed by the color of choice, and finally the state; on or off. Using the Arduino software you can test sending commands via Serial to ensure commands match the correct output.

The Raspberry Pi will be programmed as a web socket-server. The server will run on it's assigned IP address when connected to the wireless router. The Pi will receive commands from a mobile client via web sockets. Those commands will propagate into the Arduino which will be connected via USB-A to USB-B cable.

The Ionic mobile app will be connected to the same wireless router network as the Raspberry Pi. The mobile will use the Raspberry Pi's IP address to connect via web socket. The mobile app will control all the leds.

### Code Snippets
Before complex software can programmatically control RGB LEDs it is crucial to make the basic tests of the various pieces.
#### Arduino
The code below sets up the Arduino with a single color LED and Serial communication. During the loop the Arduino is always watching for Serial.available to see if a command has come in. When a command is received we process it and change the LED as needed.
```
int redpin = 11;
String incomingCommand = "";

void setup() {
  pinMode(redpin, OUTPUT);
  Serial.begin(115200);
}

void loop() 
{
  if( Serial.available() > 0 ) {
    incomingCommand = Serial.readStringUntil('\n');
    if( incomingCommand == "RED" ) {
      analogWrite(redpin, 128);  
    } else if( incomingCommand == "OFF" ) {
      analogWrite(redpin, 0);
    } 
  }
}
```
#### Serial Communication
When the Arduino is programmed to receive commands we then connect it to the Pi. The following Pi program opens up a connection to the Arduino and sends a command 2 seconds after opening connection. The test message "RED\n" causes the Arduino to turn on the RED LED. A new line (\n) is needed when sending commands to Arduino so it can know when you command ends.
```
var serial = require('serialport'),
  arduino = new serial('/dev/ttyACM0', {
    baudRate: 115200
  });
    
arduino.on('open', function(){
  console.log('PORT OPEN');
});

setTimeout( function(){
  arduino.write("RED\n");
}, 2000);
```
#### Web Socket-Server
The code below shows the setup of an http server with socket listening capability. Upon a connection to the server a callback function with client socket if initialized and within this callback we place the messages we want to listen to. Below we show that a client is allowed to send a hello message in the form `event:hello`. The message form is customizable to be anything but normal convention is to separate via colon (:) and due to the event-driven nature of NodeJs we'll always prefix messages with `event`.
```
var http = require('http')
  , server = http.createServer( handler )
  , io = require('socket.io')( server );
  
function handler( request, reply ) {
  reply.writeHead(200);
  reply.end('Running!');
}
io.on('connection', function( socket ) { 
  socket.on('event:hello', function(){
    console.log('Client says hello');
  });
});
server.listen( 8000, '192.168.0.6' );
```
#### Client Socket
The code below connects to a socket server at the defined url & port number. Then immediately after sends a message. We check the socket-serve's console log to confirm the server received the message.
```
var socket = io('http://192.168.0.6:8000');

socket.emit( 'event:hello' );
```
### Full System
Using the code snippets above we followed an iterative and agile approach to develop small pieces of working code followed by integration. First the Arduino was programmed and tested to ensure it properly executes the commands it received. Those commands were tested using the built in monitor. Then we connected the Arduino to the Raspberry Pi and write a node application to test sending the Arduino commands. The next step was to allow the commands to come over the internet so we built a web socket server in nodejs to listen for commands. To test that the server was receiving commands we built the client socket and had it send a command. On the client, we then proceeded to map individual buttons to send a different command.

Starting from the mobile client there were numerous buttons laid vertically (see below). Each button was mapped to a ```socket.emit( ... )``` which passed a command to the socket server on the Raspberry Pi. The Raspberry Pi then used the Serial setup to forward that command to the Arduino where it properly turned on the correct LED.

[Mobile Client](https://raw.githubusercontent.com/ComputerEnchiladas/arduino/master/testing/IMG_1020.PNG)

View a full system testing [video](https://github.com/ComputerEnchiladas/arduino/blob/master/testing/prototype_testing.m4v?raw=true)
