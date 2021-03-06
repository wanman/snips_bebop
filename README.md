# Jarvis, fly my drone !

I've made a proof of concept of a voice controlled UAV using Bebop 2. With small modification you can make it work with Mambo because it use pyParrot that support Mambo.
Quite useless but maybe it could help disable people to fly UAV and experiment FPV. 
A largest use case for vocal assistant in UAV could be to talk to the flight controller to query information about battery or altitude.

Demo Video :

[![Youtube video](https://i.ytimg.com/vi_webp/GNpz7S0B6Gs/sddefault.webp)](https://www.youtube.com/watch?v=GNpz7S0B6Gs)


I've made it just for fun to play with vocal assistant. I use Snips.ai fot that because :
- No need for an internet connection
- Privacy : nothing is send to the cloud unlike Google Home or Amazon Alexa
But it has a price :
- You have to download the assistant to the Raspberry every time you change it.


It was a fun project. Surprisingly, the must difficult part was to find a working microphone. I begin with a USB sound card and it is not working well because of low volume input.
Using a mic-array will be a huge improvement.

How it works :
- You talk to the microphone
- The keyword "Jarvis" start the Snips voice recognition
- Snips send a message to the MQTT bus with action
- My code listen to the bus and send action to the uav through Wifi
- The uav fly according to the voice message



Hardware part that you need :
- A Raspberry Pi 3 with Raspian Stretch (not Jessie and not a Raspberry Pi 2 : not enough processing power for on-board voice recognition with Snips)
- A USB microphone, or better a mic-array like ReSpeaker from Seeed Studio
- A speaker with amplifier (or a headphone)
- A Bebop 2, or a Mambo
- A power source (LiPo with 5V BEC)
- An HDMI FPV Screen and a keyboard if you want to debug in the field


[![Hardware part](https://github.com/trancept/snips_bebop/blob/master/hardwarePart-800.jpg?raw=true)]

Software part :
- Install Raspian, Python 3 and pip3
- Install ZeroConf (https://en.wikipedia.org/wiki/Zero-configuration_networking) for networking, untangle (https://github.com/stchris/untangle) for XML and paho-mqtt  for MQTT message :
```
sudo -H pip3 install paho-mqtt  zeroconf untangle
```
- Snips, see https://github.com/snipsco/snips-platform-documentation/wiki/1.-Setup-the-Snips-Voice-Platform  for detailed installation. It cover the sound configuration.
- Test the sound by recording and replay the record to check that it works :
```
arecord  --duration=10 -f cd -vv ~/rectest.wav
aplay ~/rectest.wav
```
- Snips FlyUAV bundle, you can create it from https://console.snips.ai as the bundle is in the Marketplace :
  - Create an account in https://console.snips.ai
  - Create a new assistant and add the "Bebop Fly (English)" (or create it yourself from the config file in https://github.com/trancept/snips_bebop/tree/master/snips_config )
  - Download it to the Raspberry and deploy it according to https://github.com/snipsco/snips-platform-documentation/wiki/1.-Setup-the-Snips-Voice-Platform 
  - My project include the dependency of pyParrot as a sub-module :
```
git clone git@github.com:trancept/snips_bebop.git
cd snips_bebop
git submodule init
git submodule update
```

## Connect to the Bebop
Power on the Bebop only, not the controller.
Check the wifi AP name with :
```
sudo iwlist wlan0 scan
```
Convert the password :
```
wpa_passphrase Bebop2-XXXX your_password
```
Write it in the wifi config file :
```
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
network={
        ssid="Bebop2-XXXX"
        psk=the_long_alphanumeric_string_printed_by_wpa_supplicant
}
```
Reload the config :
```
wpa_cli -i wlan0 reconfigure
```
Wait a while then check the connection
```
ifconfig wlan0
```
## Test

To view messages run
```
snips_watch -v &
```
Then run programm :
```
python3 ./snips_bebop.py
```

Say "Jarvis, takeoff"
And you will see this message :
```
[20:34:24] [Hotword] detected on site default
[20:34:24] [Dialogue] session with id 'ed6a20ab-f4ca-4e9e-9682-9f41a127d1e2' was started on site default
[20:34:28] [Asr] captured text "takeoff" in 3.9s
[20:34:28] [Nlu] detected intent trancept:BebopFly with probability 1.000 for input "takeoff"
              Slots ->
                 Action -> takeoff
b'{"sessionId":"ed6a20ab-f4ca-4e9e-9682-9f41a127d1e2","customData":null,"siteId":"default","input":"takeoff","intent":{"intentName":"trancept:BebopFly","probability":1.0},"slots":[{"rawValue":"takeoff","value":{"kind":"Custom","value":"takeoff"},"range":{"start":0,"end":7},"entity":"FlyAction","slotName":"Action"}]}'
OK, I will takeoff my lord.
Taking off
Going to takeoff
```

But the UAV will net takeoff because you are in safe mode by default.
Say "land"

And you will see this message :
```
[20:34:37] [Asr] captured text "land" in 6.6s
[20:34:37] [Nlu] detected intent trancept:BebopFly with probability 1.000 for input "land"
              Slots ->
                 Action -> land
[20:34:37] [Dialogue] New intent detected trancept:BebopFly with probability 1.000
              Slots ->
                 Action -> land
b'{"sessionId":"ed6a20ab-f4ca-4e9e-9682-9f41a127d1e2","customData":null,"siteId":"default","input":"land","intent":{"intentName":"trancept:BebopFly","probability":1.0},"slots":[{"rawValue":"land","value":{"kind":"Custom","value":"land"},"range":{"start":0,"end":4},"entity":"FlyAction","slotName":"Action"}]}'
OK, I will land my lord.
Landing
Going to land
```

## Fly for real
Edit snips_bebop.py and change `"with_drone = False"` to `"with_drone = True"`
 
That's it. Let me know if you find it useful.
