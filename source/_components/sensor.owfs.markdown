---
layout: page
title: "One wire Sensor"
description: "Instructions on how to integrate One wire (1-wire) sensors into Home Assistant."
date: 2018-11-04 10:10
sidebar: true
comments: false
sharing: true
footer: true
logo: onewire.png
ha_category: DIY
ha_release: 0.12
ha_iot_class: "Local Polling"
---

The `onewire` platform supports sensors which are using the One wire (1-wire) bus for communication.

Directly supported devices (polling is done by OWFS):

- [DS18B20](https://datasheets.maximintegrated.com/en/ds/DS18B20.pdf)
- [DS18S20](https://www.maximintegrated.com/en/products/analog/sensors-and-sensor-interface/DS18S20.html)

Indirectly supported devices (polling is done by Home Assistant): 
- Any device attribute that's supported by OWFS.

## {% linkable_title Configuration %}

To enable an 1wire sensor in your installation, add the following to your `configuration.yaml` file:

```yaml
# Example configuration.yaml entry
sensor:
  - platform: owfs
    name: your_name
    id: 12.345678901234.56
    attr: "TAI8570/temperature"
```

{% configuration %}
name:
  description: friendly name of your sensor attribute.
  required: true
  type: string
id:
  description: Device address of your sensor, in F.I.C format.
  required: true
  type: string
attr:
  description: Attribute to be read.
  required: false
  type: string
poll:
  description: Polled attributes. See <a href="/components/knx/#Polling"> the Hub documentation</a> for details.
  required: false
  type: map
{% endconfiguration %}

If you do not use an attribute, the family-specific default attribute will
be used. For instance, temperature sensors (family code 0x10) use
"latesttemp" or "temperature".

Reading the DS18x20 sensor's temperature is handled via background
conversion; see the

