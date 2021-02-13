# My Initial Journey with Grafana, <br> InfluxDB, Prometheus, VMware, Linux, Chickens <br> Part I - The Basics
![Grafana Dashboard](https://github.com/DennisFaucher/grafana101/blob/main/images/Grafana%20-%203%20Streams%20-%20Dodge.jpg)

# Why

Well, two reasons actually:
1. As a vExpert, I have been digging into VMware's monitoring tools such as vRealize Operations and vRealize Log Insight. Wonderful tools and relatively easy to use. Some of my peer vExperts, especially [Jorge de la Cruz](https://jorgedelacruz.uk/), use Grafana to monitor VMware as well as other infrastructure and I wanted to compare the tools.
2. One of my primary customers at work, is looking to get more out of their use of Grafana as a single dashboard tool

# How

## Parts List

* Grafana
* Telegraf
* InfluxDB
* Chronograf
* Kapacitor

## Installing the core software
The "Starter Kit" for Grafana dashboards requires the installation of [Grafana](https://grafana.com/oss/grafana/), [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/), and [InfluxDB](https://www.influxdata.com/). Telegraf sends metrics to the InfluxDB time series database and Grafana displays the InfluxDB (and other) data as beautiful graphs.

## Installation Options
### Raspberry Pi OS VM
![Raspberry Pi Logo](https://github.com/DennisFaucher/grafana101/blob/main/images/rPi-160W.jpeg)

When I started my journey, I was looking for software that would run in a Raspberry Pi OS VM on top. of ESXi-ARM on a Raspberry Pi 4. All the feeds to InfluxDB became a bit much for the rPi4 IO bus and the VM would often lock up, so I moved on to a Ubuntu VM on Intel NUC. If you would like to try this rpi OS VM installation option, there is a great tutorial [here](https://pimylifeup.com/raspberry-pi-prometheus/). 

### Ubuntu VM
![Ubuntu Logo](https://github.com/DennisFaucher/grafana101/blob/main/images/Ubuntu160.png)

Since my rPi OS VM kept getting overwhelmed with data feeds, I decided to reinstall everything in a Ubuntu VM on ESXi on an Intel NUC. This has been much more reliable. In Jorge de la Cruz's [blog post](https://jorgedelacruz.uk/2018/10/01/looking-for-the-perfect-dashboard-influxdb-telegraf-and-grafana-part-xii-native-telegraf-plugin-for-vsphere/) on Grafana & VMware, there is a [link](https://www.digitalocean.com/community/tutorials/how-to-monitor-system-metrics-with-the-tick-stack-on-ubuntu-16-04) for the installation of the TICK Stack (Telegraf, InfluxDB, Chronograf, Kapacitor) and also a [link](http://docs.grafana.org/installation/) on how to add Grafana to TICK.


# Thank You
