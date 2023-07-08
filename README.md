# Volvo-RTI-Crankshaft-Canbus-Melbus
I'm not a programmer/coder and this is my first project, so if anyone can help with editing the codes so I could use less hardware, that would be awesome. In a 2005 Volvo V70 the RTI screen displays Crankshaft, can be navigated using the steering wheel buttons and plays sound to the original stereo. Raspberry Pi + 3b, Arduino Nano, ESP32, ESP8266

https://youtu.be/Y5DPQHbPuwg

Right when I decided to purchase a Volvo V70 with an RTI I started Googling what to use the screen for.

I found this project: https://github.com/LuukEsselbrugge/Volve

That gave me an idea to try to do the same thing.
The problem is I hadn't had much experience with microcontrollers and stuff like that, so I didn't actually understand what I had to do, but decided to give it a shot anyway and started by getting a raspberry and installing Crankshaft (Open Auto / Android Auto) on it: 
https://getcrankshaft.com

After getting Crankshaft installed on the Raspberry, I went and got myself an Arduino Nano to emulate the CD-CHANGER. I think, that the code, that finally worked for me, was this one: https://gist.github.com/klalle/1ae1bfec5e2506918a3f89492180565e

Now I was able to use Crasnkshaft to have the cars stereo play music from my phone.

Next I got an esp8266 which controls the RTI screen, telling it to drive out of the dash, what source to use and what brightness to use.

I think the code I ended up using, I got from here: https://www.swedespeed.com/threads/volvo-rti-navigation-project-with-android-odroid-platform-controlled-with-arduino.434729/

Now I was finally able to see what's going on in Crankshaft but had no way of controlling it - you can't open Waze on your phone while it's connected to Arduino Auto.

So, I would always set the destination on my phone and only then plug the phone in, that way I could still kind of use it. Meanwhile liking or skipping songs on Spotify was done on my smartwatch - I used it like that for about a year.

Now to the CAN bus:

Ordered myself an ESP32 and got a can transceiver from the local store.

Every once in a while i would have these fits of googling and I would go out to the car and try something out but nothing ever worked.

I had trouble connecting to the cars can bus.

So what I did - I got a second set of esp32 and can transceiver and tried to get them to talk to each other, so I would know the hardware was working.

By finding this I finally could be certain, that the hardware is connected correctly, because I could get the two ESP32's to communicate to each other: https://www.circuitstate.com/tutorials/what-is-can-bus-how-to-use-can-interface-with-esp32-and-arduino/

BUT I still couldn't get any contact with the car.... Or actually I could - every once in a while i would get the whole dash to shut down, or the airbags turned off or some other weird faultcodes to appear, but that was always by mistake.

I came upon this source and realised I can not use the can lines on the OBDII port, but instead would have to use the can lines on the RTI, which is what I had been doing half the times: https://www.swedespeed.com/threads/swedespeed-canfoolery-project-understanding-and-using-the-volvo-canbus.136362/´

From that same link I actually found something that finally changed everything: All the sources I could find, said that the bitrate for this cars LS CAN was supposed to be 125kbps but on this link they said, it's supposed to be 250kbps. I finally realised, that every time I tried using 125 and had the hardware wired up correctly, that's when all the weird fault codes would appear.

Now I was finally seeing the messages on canbus but Arduino IDE didn't allow me to copy them all, so I installed Coolterm to get a better understanding of these messages: https://coolterm.en.lo4d.com/windows

I still wasn't entirely sure, that the codes I was getting were correct, until I realised that some of the ID's I was getting matched the ones I found here: https://github.com/andrewgabler/VolvoDIM/tree/master/Research

Now that I knew I was getting the correct data I couldn't stop anymore, so I tried to filter the data somehow to understand it better and so on, but had no luck.... The weird thing was, all of the sources I found, said that the ID for the steering wheel buttons should be 0x0400066 but I never got a code with this ID.

So I kept Googling until I came across this: https://github.com/vtl/volvo-kenwood/blob/master/kenwood/kenwood.ino

And in that code the id for the steering wheel buttons is 0x0131726c and I had that ID in my data as well. 

After getting the correct ID I was able to filter the CAN messages so I would receive only the messages with the id 0x0131726c

What I saw was:

ID: 0x131726C
Example message: 00 0C 2E 52 E0 00 00 3F - No buttons are pressed

00 0C 2E 52 E0 00 10 3F - Back is pressed.
00 0C 2E 52 E0 00 20 3F - Enter is pressed.
00 0C 2E 52 E0 00 08 3F - Up is pressed.
00 0C 2E 52 E0 00 04 3F - Down is pressed.
00 0C 2E 52 E0 00 02 3F - Left is pressed.
00 0C 2E 52 E0 00 01 3F - Right is pressed.
00 0C 2E 52 E0 00 00 1F - No is pressed.
00 0C 2E 52 E0 00 00 2F - Yes is pressed.
00 0C 2E 52 E0 00 00 3D - Next track is pressed.
00 0C 2E 52 E0 00 00 3E - Previous track is pressed.


Now it was time to forward this information to the raspberry as events of keyboard presses.

It took me a while but with the help of ChatGPT i finally managed to get the python code in raspberry to work.

I then turned it into a service and made it run every time the Raspberry was booted.

Unfortunately what I discovered, was that the can is so fast, that by pressing the button

 on the steering wheel once, I got about 5-10  can messages which meant, I got about 5-10 "keyboard presses"... So I added a "filter" to the buttons.py code - it reacts to the same button only once in 0.3 seconds.


I probably will go more into detail at some point, but for now that's it.

![Image](https://user-images.githubusercontent.com/50247619/251963724-3f1f68fc-9c28-4a6d-a523-0d869fcc9419.png)
