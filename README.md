# Google Local Fulfilment : Cannot switch on all color lights with Local Path

My issue here is when I try to switch on all lights in once, my execute request is handled by my local Fulfiment implementation but it is ALSO handled by my cloud implementation. Everything goes ok but the speaker respond: "Local Fulfilment is not reachable".

Here I try to reproduce this behavior with the sample code provided here: 

> https://github.com/actions-on-google/smart-home-local.git

But first a "request and response" recap when i try to switch on/off all lights:

Request send by my google nest mini, when i say: "Ok Google, switch off all lights":

```
{
    inputs: [
        {
            context: {
                locale_country: 'FR',
                locale_language: 'fr',
            },
            intent: 'action.devices.EXECUTE',
            payload: {
                commands: [
                    {
                        devices: [
                            {
                                id: 'LIGHT:1',
                            },
                            {
                                id: 'LIGHT:2',
                            },
                            {
                                id: 'LIGHT:3',
                            },
                        ],
                        execution: [
                            {
                                command: 'action.devices.commands.OnOff',
                                params: {
                                    on: false,
                                },
                            },
                        ],
                    },
                ],
                structureData: {},
            },
        },
    ],
    requestId: '17639346050416761654',
};
```

Response i return:

```
{
    requestId: '17639346050416761654',
    payload: {
        commands: [
            {
                ids: ['LIGHT:1'],
                status: 'SUCCESS',
                states: 'SUCCESS',
            },
            {
                ids: ['LIGHT:2'],
                status: 'SUCCESS',
                states: 'SUCCESS',
            },
            {
                ids: ['LIGHT:3'],
                status: 'SUCCESS',
                states: 'SUCCESS',
            },
        ],
    },
    metrics: {
        latencyInMilliseconds: 730,
        receivedAt: 1587594181986,
        source: 'local_home_sdk',
    },
};
```
---
 Informations about device i use:

|   Google home nest mini           |   |
| --------------------- | :------------: |
| System Firmware | 200660 |
| Cast Firmware               |       1.46.200660     |
| Language                 |     Français      |
| Country Code             |    FR     |

---

# How to Reproduce it with code sample:

I suppose here that you already create a smart home project.

## Deploy firbase cloud functions

Go in *functions* folder, and run these commands on a terminal:

1. `npm install`

2. `npm run firebase login`

3. `npm run firebase use ${Your Project Id}`

4. `npm run firebase functions:config:set hub.leds=16 hub1.channel=1,2,3 hub1.control_protocol=TCP`

5. `npm run firebase functions:config:get` to check if everything is ok. I have this: 

```
{
  "hub1": {
    "channel": "1,2,3",
    "control_protocol": "TCP"
  },
  "hub": {
    "leds": "16"
  }
}
```
6. `npm run deploy`

## Set Up Your virtual devices

Go in *device* folder, and run these commands on a terminal:

1. `npm install`

2. `npm start`

## Start your Local Fulfiment Implementation

Go in *app* folder, and run these commands on a terminal:

1. `npm install`

2. `npm start`

## Ask your request

I ask: "ok Google, set color lights to blue".

Speaker response: "Local Fulfilment is not reachable".

Execute Request :

```
{
  "inputs": [
    {
      "context": {
        "locale_country": "FR",
        "locale_language": "fr"
      },
      "intent": "action.devices.EXECUTE",
      "payload": {
        "commands": [
          {
            "devices": [
              {
                "customData": {
                  "channel": 1,
                  "control_protocol": "TCP",
                  "leds": 16,
                  "port": 7890,
                  "proxy": "hub1"
                },
                "id": "hub1-1"
              },
              {
                "customData": {
                  "channel": 3,
                  "control_protocol": "TCP",
                  "leds": 16,
                  "port": 7890,
                  "proxy": "hub1"
                },
                "id": "hub1-3"
              },
              {
                "customData": {
                  "channel": 2,
                  "control_protocol": "TCP",
                  "leds": 16,
                  "port": 7890,
                  "proxy": "hub1"
                },
                "id": "hub1-2"
              }
            ],
            "execution": [
              {
                "command": "action.devices.commands.ColorAbsolute",
                "params": {
                  "color": {
                    "name": "bleu",
                    "spectrumRGB": 255
                  }
                }
              }
            ]
          }
        ],
        "structureData": {}
      }
    }
  ],
  "requestId": "4159764943369176424"
}

```


Response:

```
{
  "requestId": "4159764943369176424",
  "payload": {
    "commands": [
      {
        "ids": [
          "hub1-1"
        ],
        "status": "SUCCESS",
        "states": {
          "color": {
            "name": "bleu",
            "spectrumRGB": 255
          },
          "online": true
        }
      },
      {
        "ids": [
          "hub1-3"
        ],
        "status": "SUCCESS",
        "states": {
          "color": {
            "name": "bleu",
            "spectrumRGB": 255
          },
          "online": true
        }
      },
      {
        "ids": [
          "hub1-2"
        ],
        "status": "SUCCESS",
        "states": {
          "color": {
            "name": "bleu",
            "spectrumRGB": 255
          },
          "online": true
        }
      }
    ]
  }
}
```