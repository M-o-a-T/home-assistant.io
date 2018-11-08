---
layout: page
title: "OWFS"
description: "Instructions on how to integrate 1wire components with Home Assistant."
date: 2018-10-26 12:00
sidebar: true
comments: false
sharing: true
footer: true
logo: onewire.png
ha_category: Hub
ha_release: 0.90
ha_iot_class: "Local Polling"
---


The [OWFS](http://owfs.org) integration for Home Assistant allows you to talk to 1wire devices.

This component differs from the "onewire" component in the following ways:
- support for more device types (not only sensors)
- support for polling, alarms, background measurements, and more
- support for multiple 1wire servers and buses
- network access via owserver
- 100% async code

This component requires at least one running [owserver](http://owfs.org/index.php?page=owserver).
Accessint 1wire components requires a completed scan of the 1wire bus on
which that component is located. By default this happens during HASS startup.

There is currently support for the following device types within Home Assistant:

- [Sensor](/components/sensor.owfs)

All sensors supported by OWFS may be used.

## {% linkable_title Configuration %}

To use a locally-running owserver in your installation, add the following lines to your `configuration.yaml` file:

```yaml
owfs:
  server:
  - host: localhost
```

{% configuration %}
host:
  description: The host name or IP address of the computer running an "owserver" instance.
  type: string
port:
  description: The network port to connect to. Default: 4304 (owserver port).
  type: integer
scan:
  description: Interval for periodically scanning the bus. Defaults to zero
      (no repeated scanning)
  type: float
scan_delay:
  description: Initial delay before scanning the bus. Default: The bus is
    scanned immediately and HASS set-up will delay until the initial scan is
    completed. You can disable the initial scan by using "-1".
  type: integer
poll:
  description: Flag whether to periodically scan the bus, if requested by a device. Defaults to True.
  type: boolean

{% endconfiguration %}

<p class='note warning'>
  "owserver" is able to export a uniform bus system by talking to other
  owserver instances in the background, and forwarding requests.

  You should not use this feature when using Home Assistant.

  Talking to a single 1wire bus via two different ways, either by adding a
  server twice or by adding a server that forwards to another server which
  you also talk to directly, is not supported and will cause problems.
</p>

Auto-discovering owserver instances via Zeroconf/Avahi/mDNS is not yet supported.


### {% linkable_title Services %}

In order to directly set a 1wire value, you can use this service:

```
Domain: owfs
Service: write
Service Data: {"device": "10.123456789ABC.90", "attr": "temphigh", "value": 100}
```

{% configuration %}
device:
  description: 64-bit ROM address of the device, including the checksum.
  type: string
attr:
  description: Attribute to set.
  type: string
value:
  description: Payload, either an integer or a string
  type: [integer, string]
{% endconfiguration %}


You can also use a service to trigger reading a 1wire value:

```
Domain: owfs
Service: read
Service Data: {"device": "10.123456789ABC.90", "attr": "latesttemp"}
```

{% configuration %}
device:
  description: 64-bit ROM address of the device, including the checksum.
  type: string
attr:
  description: Attribute to read.
  type: string
{% endconfiguration %}

Due to the unreliability and the asynchronous nature of the 1wire bus, the
actual data will (hopefully) be sent as an event:

```
Event: owfs.value
Event Data:
  device: 10.123456789ABC.90
  attr: temperature
  value: 12.75
```

{% configuration %}
device:
  description: 64-bit ROM address of the device, including the checksum.
  type: string
attr:
  description: Attribute that's been read.
  type: string
value:
  description: Value that's been read.
  type: string
{% endconfiguration %}

## {% linkable_title Polling %}

1wire is a passive bus, i.e. all communication needs to originate with the
bus master. Reading and writing data usually is instantaneous, but some
requests require a significant delay for data conversion to happen.

Thus, some devices support "simultaneous" access, which triggers a
conversion step on all similar devices on a bus in the background. Some
other devices support "alarm" conditions: OWFS can quickly scan which
device requires attention.

The "poll" configuration item controls how often these steps should happen.
It accepts the following entries (all values must be in seconds):

* attr

  This entry controls how often Home Assistant will read your attribute directly. 
  Do *not* use this for attributes that are accessed by simultaneous conversion.

* alarm

  This entry controls how often OWFS polls for alarm conditions. Note that
  for devices that are not directly supported, it's your responsibility to
  turn the alarm condition off. Alarms are reported using an "owfs.alarm"
  event.

* temperature

  This entry controls how often a simultaneous temperature measurement is
  triggered. *Do not* use parasitically-powered thermometers.

* voltage

  This entry controls how often a simultaneous voltage measurement is
  triggered. *Do not* use parasitically-powered sensors.

Restrictions:

* For "attr": Polling only happens every *scan_interval* seconds; see
  <a href="/docs/configuration/platform_options/#scan-interval">
  platform-specific options</a> for details.

* For all other parameters: If you use different values for devices on the
  same bus, the smallest value will be used.

### {% linkable_title Alarms %}

Alarms are reported in an "owfs.alarm" event.

{% configuration %}
device:
  description: The device that reports an alarm condition.
  type: string
results:
  description: device- and alarm-specific
  type: map
{% endconfiguration %}

### {% linkable_title Known issues %}

1wire slaves that are not known to the "trio_owfs" module may delay alarm
processing and/or polling. If you are affected, please submit an issue to
<a href="https://github.com/python-trio/trio-owfs">trio_owfs</a>.

