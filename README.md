# My Initial Journey with Grafana, Telegraf,<br> InfluxDB, Prometheus, VMware, Linux, Chickens <br> Part I - The Basics
![Grafana Dashboard](https://github.com/DennisFaucher/grafana101/blob/main/images/Grafana%20-%203%20Streams%20-%20Dodge.jpg)

# Why

Well, two reasons actually:
1. As a vExpert, I have been digging into VMware's monitoring tools such as vRealize Operations and vRealize Log Insight. Wonderful tools and relatively easy to use. Some of my peer vExperts, especially [Jorge de la Cruz](https://jorgedelacruz.uk/), use Grafana to monitor VMware as well as other infrastructure and I wanted to compare the tools.
2. One of my primary customers at work, is looking to get more out of their use of Grafana as a single dashboard tool

# How

## Parts List
![Rube Goldberg](https://github.com/DennisFaucher/grafana101/blob/main/images/Rube_Goldberg's__Self-Operating_Napkin__(cropped).gif)

* An OS (You'll probably find the most help for Linux)
* Grafana (Dashboards and graphs)
* Telegraf (Metrics collector)
* InfluxDB (Time series database)
* Chronograf (InfluxDB simple dashboards. I use Grafana dashboards instead.)
* Kapacitor (InfluxDB alerts. I use Grafana alerts instead)
* Prometheus (Another Time Series Database. Not covered in Part I)

## Installing the core software
The "Starter Kit" for Grafana dashboards requires the installation of [Grafana](https://grafana.com/oss/grafana/), [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/), and [InfluxDB](https://www.influxdata.com/). Telegraf sends metrics to the InfluxDB time series database and Grafana displays the InfluxDB (and other) data as beautiful graphs.

## Installation Options
### Raspberry Pi OS VM Option
![Raspberry Pi Logo](https://github.com/DennisFaucher/grafana101/blob/main/images/rPi-160W.jpeg)

When I started my journey, I was looking for software that would run in a Raspberry Pi OS VM on top of ESXi-ARM on a Raspberry Pi 4. All the feeds to InfluxDB became a bit much for the rPi4 IO bus and the VM would often lock up, so I moved on to a Ubuntu VM on Intel NUC. If you would like to try this rPi OS VM installation option, there is a great tutorial [here](https://pimylifeup.com/raspberry-pi-prometheus/). 

### Ubuntu VM Option
![Ubuntu Logo](https://github.com/DennisFaucher/grafana101/blob/main/images/Ubuntu160.png)

Since my rPi OS VM kept getting overwhelmed with data feeds, I decided to reinstall everything in a Ubuntu VM on ESXi on an Intel NUC. The NUC VM has been much more reliable. In Jorge de la Cruz's [blog post](https://jorgedelacruz.uk/2018/10/01/looking-for-the-perfect-dashboard-influxdb-telegraf-and-grafana-part-xii-native-telegraf-plugin-for-vsphere/) on Grafana & VMware, there is a [link](https://www.digitalocean.com/community/tutorials/how-to-monitor-system-metrics-with-the-tick-stack-on-ubuntu-16-04) for the installation of the TICK Stack (Telegraf, InfluxDB, Chronograf, Kapacitor) and also a [link](http://docs.grafana.org/installation/) on how to add Grafana to TICK. As you can see in the Parts List, not all pieces of the TICK stack are needed, but the tutorial is very helpful in installing the parts you want. Here is a brief summary of the installation step from those posts for the parts I am using:

#### Install & Configure InfluxDB

````[bash]
$ curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add -
$ source /etc/lsb-release
$ echo "deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/influxdb.list

$ sudo apt-get update
$ sudo apt-get install influxdb
$ sudo systemctl start influxdb
$ sudo systemctl status influxdb (Check and fix any issues)
$ sudo systemctl enable influxdb

$ influx (If not found "sudo apt install influxdb-client")
> CREATE USER "sammy" WITH PASSWORD 'sammy_admin' WITH ALL PRIVILEGES
> show users (Check that the sammy user is there)
> exit

$ sudo vi /etc/influxdb/influxdb.conf (Or nano. Or vim. Your preference)
Find the [http] section and set auth-enabled to true. Save the file and exit the editor.
...
    [http]
      # Determines whether HTTP endpoint is enabled.
      # enabled = true

      # The bind address used by the HTTP service.
      # bind-address = ":8086"

      # Determines whether HTTP authentication is enabled.
      auth-enabled = true
...

$ sudo systemctl restart influxdb

````

#### Install & Configure Telegraf


# Thank You
