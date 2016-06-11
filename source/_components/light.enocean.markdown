---
layout: page
title: "EnOcean Light"
description: "Instructions on how to set up EnOcean lights within Home Assistant."
date: 2016-05-25 23:49
sidebar: true
comments: false
sharing: true
footer: true
logo: enocean.png
ha_category: Light
---

An EnOcean light can take many formes. Currently only one type has been tested: Eltako FUD61 dimmer.


To use your EnOcean device, you first have to set up your [EnOcean hub](../enocean) and then add the following to your `configuration.yaml` file:

```yaml
# Example configuration.yaml entry
light:
  - name: Living_room
    platform: enocean
    id: [0x01,0x90,0x84,0x3C]
    sender_id: [0xFF,0xC6,0xEA,0x04]
```

Configuration variables:

- **id** (*Required*): The ID of the device. This is the 4 bytes long number written on the dimmer.
- **sender_id** (*Required*): The Sender ID of the device. This is a 4 bytes long number.
- **platform** (*Required*): Set to `enocean`.
- **name** (*Required*): An identifier for the switch
