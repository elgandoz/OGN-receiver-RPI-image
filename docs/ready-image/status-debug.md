---
description: >-
  This page shows some information about how to get information out of your OGN
  receiver
---

# Status & Debug

This page make the assumption you use the Sebastien's RPi OGN image or followed his [image creation steps](../expert/image-creation-steps.md).

## Getting receiver status

The receiver provides some status information via a webserver, visible with any browser.

Access with your RPi OGN receiver IP. On MacOs or if you have Bonjour installed you can use `ogn-receiver.local`

### Status for ogn-rf

* Visit [http://ogn-receiver.local:8080](http://ogn-receiver.local:8080)

### Status for ogn-rtl

* Visit [http://ogn-receiver.local:8081](http://ogn-receiver.local:8081)

## Getting services information

Log in your RPi ogn-receiver via ssh of directly.  
Assuming you have telnet installed \(see Image Creation Steps\), you can get debug information from the services typing:

### Debug for **ogn-rf**

```c
telnet localhost 50000
```

### **Debug for ogn-rtl**

```c
telnet localhost 50001
```

{% hint style="info" %}
You can press`Ctrl+C` to stop the stream and exit to the prompt.
{% endhint %}

## Remote Status

* Visit [`https://ogn.peanutpod.de/`](https://ogn.peanutpod.de/) and lookup your station name

This page will fetch informations from the OGN networks, from the receiver itself and from the [wiki's list of receivers](http://wiki.glidernet.org/list-of-receivers).

