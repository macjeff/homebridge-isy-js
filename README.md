# homebridge-isy-js 

[![Build Status](https://api.travis-ci.org/rodtoll/homebridge-isy-js.svg?branch=master)](https://api.travis-ci.org/rodtoll/isy-js.svg?branch=master)

(C) Rod Toll 2015-2017, Licensed under the MIT-LICENSE. Additional development work by pradeepmouli, 2018-

# Major Updates

Requires new version of [isy-js](https://github.com/pradeepmouli/isy-js.git). Please see revised installation notes below.

### New Functionality:

- Support for ISY 5.0.12+
- Support for additional devices:
     Insteon Thermostat/Thermostat Adapters
     Leak Sensors
     Remotes

### Bug Fixes:

- More robust handling for device status changes from ISY and sending updates to ISY

### Known Issues:

- ELK devices currently not working (Z-Wave devices might). Please disable ELK devices to use this version (set "elkEnabled" = false). If you would like to help fix, (I need someone to test), please reach out to me.

### TODO:

- Re-add support for non-insteon native devices (Elk & ZWave) 
- Add Program/Variable support
- Incorporate individual device status in scene definitions
- Create configurable mappings for IOLinc to other types of accessories (e.g. curtains & sprinkler control).

### Future Work:

- Add NodeServer support. May implement or may implement in ISY-JS only, as using native homebridge plugins may be better solution. E.g. for nest devices, you would use a Nest NodeServer to integrate with the ISY, and the Nest homebridge plugin to integrate with HomeKit.

-----------

## Installation

1. Install homebridge using: npm install -g homebridge
2. Install this plugin using: npm install -g github:pradeepmouli/homebridge-isy-js 
3. Install ISY-JS v5 (should happen automatically during step #2, but just in case): npm install -g github:pradeepmouli/isy-js 
4. Update your configuration file. See sampleconfig.json in this repository for a sample. 

Please note, that since npm.org is not updated, running npm -g install or npm -g update (for all packages, homebridge-isy-js or isy-js) will corrupt your installation.

--------

# Old Readme

ISY-994 REST / WebSockets based HomeBridge platform. 

NOTE: Homebridge-isy-js now includes support for garage door openers. Make sure you ensure a garage door is clear before closing it.

Supports the following Insteon devices: Lights (dimmable and non-dimmable), Fans, Outlets, Door/Window Sensors, MorningLinc locks, Inline Lincs, Motion Sensors and I/O Lincs.
Also supports ZWave based locks. If elkEnabled is set to true then this will also expose your Elk Alarm Panel and all of your Elk Sensors. 

Turns out that HomeBridge platforms can only return a maximum of 100 devices. So if you end up exposing more then 100 devices through HomeBridge the HomeKit
software will fail adding the HomeBridge to your HomeKit network. To address this issue this platform provides an option to screen out devices based on 
criteria specified in the config. 

## Requirements

Only the ISY 994 and newer devices running 5.0.12+ are supported. The ISY 99i device is no longer supported as this library depends on a later version of the REST/Websocket interface. 

## Configuration

Configuration sample:

 ```
     "platforms": [
        {
            "platform": "isy-js",
            "name": "isy-js",         
            "host": "10.0.1.12",      
            "username": "admin",      
            "password": "password",   
            "elkEnabled": true,       
            "useHttps": false,
            "debugLoggingEnabled": false,
            "includeAllScenes": false,
            "includedScenes": [
                "27346"
            ],
            "garageDoors": [
                { "address": "17 79 81 1", "name": "Garage Door", "timeToOpen": 12000 }
            ],
            "ignoreDevices": [        
                { "nameContains": "ApplianceLinc", "lastAddressDigit": "", "address": ""},
                { "nameContains": "Bedroom.Side Gate", "lastAddressDigit": "", "address": ""},
                { "nameContains": "Remote", "lastAddressDigit": "", "address": "" },    
                { "nameContains": "Keypad", "lastAddressDigit": "2", "address": "" },
            ]
            "renameDevices": [
                { "nameContains": "BadName", "newName": "Good name" }
            ]
        }
     ]
```

Fields: 
* "platform" - Must be set to isy-js
* "name" - Can be set to whatever you want
* "host" - IP address of the ISY
* "username" - Your ISY username
* "password" - Your ISY password
* "elkEnabled" - true if there is an elk alarm panel connected to your ISY
* "useHttps" - true if you want to use a https connection to the ISY. Only use if you have HTTPS setup with a proper cert.
* "debugLoggingEnabled" - true if you want debug logs to be dumped to the console.
* "garageDoors" - Specifies a list of devices which are I/O Lincs which should be treated as a garage door opener by HomeKit. The entry takes the form of one entry per device. (NOTE: In ISY an IO Linc is treated as two devices, you only need one entry for each IO Linc).
Each entry should have the following elements: "address" - the address of the IO Linc primary device (the .1 device), "name" - Name to give the garage door device, "timeToOpen" - The number of ms before a door which is opening should be considered open.
* "includeAllScenes" - true if you want the platform to expose all the scenes. Setups usually have a LOT of scenes any only 100 devices are supported from a bridge. It is recommended you proactively include the scenes you want in includedScenes.
* "includedScenes" - An array of the addresses of the scenes you want to include as lighting devices.
* "ignoreDevices" - Array of objects specifying criteria for screening out devices from the network. nameContains is the only required criteria. If the other criteri are blank all devices will match those criteria (providing they match the name criteria).
* (Under ignoreDevices) "nameContains" - Specifies a substring to check against the names of the ISY devices. Required field for the criteria.
* (Under ignoreDevices) "lastAddressDigit" - Specifies a single digit in the ISY address of a device which should be used to match the device. Example use of this is for composite devices like keypads so you can screen out the non-main buttons. 
* (Under ignoreDevices) "address" - ISY address to match.
* "renameDevices" - Array of objects specifying devices you want to rename based on their address or name.
* (Under renameDevices) "nameContains" - Specifies a substring to check against the names of the ISY devices. Required field for the criteria.
* (Under renameDevices) "address" - ISY address to match.
* (Under renameDevices) "newName" - New name to give to the device.
     
```    
Examples:

{ "nameContains": "Keypad", "lastAddressDigit": "2", "address": "" } - Ignore all devices which have the word Keypad in their name and whose last address digit is 2.
{ "nameContains": "Remote", "lastAddressDigit": "", "address": "" } - Ignore all devices which have the word Remote in their name
{ "nameContains": "", "lastAddressDigit": "", "address": "15 5 3 2"} - Ignore the device with an ISY address of 15 5 3 2.
```

## Implementation Notes

* Scenes will not show as on until all light devices are on. This allows the UI to send an 'on' request to light up the rßest of them.
* Garage door openers (in this case an I/O Linc used in a garage kit) are complex to get correct. We only have the current state of the contact
 sensor to determine current status. In particular, if you startup the system while the garage door is changing state (opening or closing)
 the code will likely get the state incorrect. If you get into this state, close the garage door and restart homebridge. The garage door is assumed open at startup and closed at startup if the contact sensor
 says the door is open or closed respectively.

-----------

# History

 * 0.2.0 - Development resumed.
    - Support for v5.0.12+
    - Support for Insteon Thermostat/Insteon Thermostat Adapter added
    - Fixed characteristic update logic to capture externally initiated changes w/o triggering update back to isy.
    - Requires new version of [isy-js](https://github.com/pradeepmouli/isy-js.git) 
 * 0.1.9 - (Rodtoll) Active development ended. 
 * 0.1.8 - Fixed crash in tests (race condition) and fixed crash in garage door device.
 * 0.1.7 - Fixed crash when there is no ignoreDevices entry. Also added new renameDevices section to enable device renaming. Added note to highlight ISY 99 is no longer supported, you needed an ISY 994 or newer. Added checks to ensure device list doesn't exceed 100 devices. Simplified ignore syntax so blank elements no longer needed.
 * 0.1.6 - Addressed crash when identify called on lights.
 * 0.1.4 - Release for testing alternative garage logic. No change for anyone wanting to use it with the standard logic.
 * 0.1.3 - Added improved debug output. Fixed bug where plugin would crash when there are no garage door opener present.
 * 0.1.2 - Added garage door opener support
 * Previous - Changes not tracked in this document.

 Fork of original homebridge-isy-js by [rodtoll](https://github.com/rodtoll/homebridge-isy-js.git)
