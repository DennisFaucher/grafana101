# My Initial Journey with Grafana, Telegraf, InfluxDB, <br> Prometheus to Monitor VMware, Linux, Chickens <br> Part I - The Basics
![Grafana Dashboard](https://github.com/DennisFaucher/grafana101/blob/main/images/Grafana%20-%203%20Streams%20-%20Dodge.jpg)

# Why

Well, two reasons actually:
1. As a vExpert, I have been digging into VMware's monitoring tools such as vRealize Operations and vRealize Log Insight. Wonderful tools and relatively easy to use. Some of my peer vExperts, especially [Jorge de la Cruz](https://jorgedelacruz.uk/), use Grafana to monitor VMware as well as other infrastructure and I wanted to compare the tools.
2. One of my primary customers at work is looking to get more out of their use of Grafana as a single dashboard tool

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
The "Starter Kit" for Grafana dashboards requires the installation of [Grafana](https://grafana.com/oss/grafana/), [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/), and [InfluxDB](https://www.influxdata.com/) (aka TIG). Telegraf sends metrics to the InfluxDB time series database and Grafana displays the InfluxDB (and other) data as beautiful graphs.

## Installation Options
### Raspberry Pi OS VM Option
![Raspberry Pi Logo](https://github.com/DennisFaucher/grafana101/blob/main/images/rPi-160W.jpeg)

When I started my journey, I was looking for software that would run in a Raspberry Pi OS VM on top of ESXi-ARM on a Raspberry Pi 4. All the feeds to InfluxDB became a bit much for the rPi4 I/O bus and the VM would often lock up, so I moved on to a Ubuntu VM on an Intel NUC. If you would like to try this rPi OS VM installation option, there is a great tutorial [here](https://nwmichl.net/2020/07/14/telegraf-influxdb-grafana-on-raspberrypi-from-scratch/). 

### Ubuntu VM Option
![Ubuntu Logo](https://github.com/DennisFaucher/grafana101/blob/main/images/Ubuntu160.png)

Since my rPi OS VM kept getting overwhelmed with data feeds, I decided to reinstall everything in a Ubuntu VM on ESXi on an Intel NUC. The NUC VM has been much more reliable. In Jorge de la Cruz's [blog post](https://jorgedelacruz.uk/2018/10/01/looking-for-the-perfect-dashboard-influxdb-telegraf-and-grafana-part-xii-native-telegraf-plugin-for-vsphere/) on Grafana & VMware, there is a [link](https://www.digitalocean.com/community/tutorials/how-to-monitor-system-metrics-with-the-tick-stack-on-ubuntu-16-04) for the installation of the TICK Stack (Telegraf, InfluxDB, Chronograf, Kapacitor) and also a [link](http://docs.grafana.org/installation/) on how to add Grafana to TICK. As you can see in the Parts List, not all pieces of the TICK stack are needed, but the tutorial is very helpful in installing the parts you want. Here is a brief summary of the installation step from those posts for the parts I am using:

#### Install & Configure InfluxDB
![InfluxDB](https://github.com/DennisFaucher/grafana101/blob/main/images/InfluxDB.png)

````[bash]
$ curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add -
$ source /etc/lsb-release
$ echo "deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" \
| sudo tee /etc/apt/sources.list.d/influxdb.list

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
Find the [http] section and set auth-enabled to true. 
Save the file and exit the editor.
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
![Telegraf](https://github.com/DennisFaucher/grafana101/blob/main/images/telegraf.png)

````[bash]
$ sudo apt-get install telegraf
$ sudo vi /etc/telegraf/telegraf.conf (Or nano. Or vim. Your preference)
Find the [outputs.influxdb] section and provide the username and password. Save the file and exit the editor.

[[outputs.influxdb]]
  ## The full HTTP or UDP endpoint URL for your InfluxDB instance.
  ## Multiple urls can be specified as part of the same cluster,
  ## this means that only ONE of the urls will be written to each interval.
  # urls = ["udp://localhost:8089"] # UDP endpoint example
  urls = ["http://localhost:8086"] # required
  ## The target database for metrics (telegraf will create it if not exists).
  database = "telegraf" # required

  ...

  ## Write timeout (for the InfluxDB client), formatted as a string.
  ## If not provided, will default to 5s. 0s means no timeout (not recommended).
  timeout = "5s"
  username = "sammy"
  password = "sammy_admin"
  
$ sudo systemctl restart telegraf
$ sudo systemctl status telegraf (Check and fix any issues)
$ sudo systemctl enable telegraf
````

Congratulations. Telegraf is now running and sending metrics from your Linux host to the InfluxDB for use by Grafana.
Your /etc/telegraf/telegraf.conf should, by default, have these sections uncommented and these metrics should be in InfluxDB.

````[bash]
[[inputs.cpu]]
[[inputs.disk]]
[[inputs.diskio]]
[[inputs.kernel]]
[[inputs.mem]]
[[inputs.processes]]
[[inputs.swap]]
[[inputs.system]]
````

You can test this by displaying the data in the Influx data base with these commands:

````[bash]
$ influx -username sammy -password sammy_admin

> use telegraf

> show measurements
name: measurements
name
----
cpu
disk
diskio
kernel
mem
swap
system

> show field keys from cpu
name: cpu
fieldKey         fieldType
--------         ---------
usage_guest      float
usage_guest_nice float
usage_idle       float
usage_iowait     float
usage_irq        float
usage_nice       float
usage_softirq    float
usage_steal      float
usage_system     float
usage_user       float

> select "usage_idle", "host" from "cpu"  limit 10
name: cpu
time                usage_idle        host
----                ----------        ----
1609336970000000000 98.43434343434373 ubuntu-nuc
1609336970000000000 97.57330637007071 ubuntu-nuc
1609336970000000000 99.29364278506394 ubuntu-nuc
1609336980000000000 98.20089955022426 ubuntu-nuc
1609336980000000000 97.80000000000194 ubuntu-nuc
1609336980000000000 98.50149850149987 ubuntu-nuc
1609336990000000000 99.45054945055108 ubuntu-nuc
1609336990000000000 99.5004995004977  ubuntu-nuc
1609336990000000000 99.50049950049996 ubuntu-nuc
1609337000000000000 99.5995995995977  ubuntu-nuc
````

#### Connect Grafana to InfluxDB

Point your browser to your new Grafana instance: 
````[bash]
http://[hostname]:3000
````

Default login is admim/admin and you will be prompted to change your password

![Grafana Login](https://github.com/DennisFaucher/grafana101/blob/main/images/GrafanaMain.png)

Define the InfluxDB as a data source

![Data Source](https://github.com/DennisFaucher/grafana101/blob/main/images/Data%20Source.png)

Name the source InfluxDB and change the IP address to the IP or name of your InfluxDB server

![Influx Settings 01](https://github.com/DennisFaucher/grafana101/blob/main/images/Influx%20Settings%2001.png)

Provide the database name (telegraf), username (sammy) and password (sammy_admin) you created, Save & Test

![Influx Settings 02](https://github.com/DennisFaucher/grafana101/blob/main/images/Influx%20Settings%2002.png)

#### Try Out a Dashboard!

There are lots of very nice pre-built dashboards at [this site](https://grafana.com/grafana/dashboards). Once you find one, you can import the dashboard right into your Grafana instance using the dashboard ID. I suggest trying [this Linux dashboard](https://grafana.com/grafana/dashboards/2381) with ID 2381 that works with metrics you have already collected.

First, choose Dashboards > Manage

![Manage](https://github.com/DennisFaucher/grafana101/blob/main/images/Dashboards%20-%20Manage.png)

Click "Import" and type the dashboard ID (2381) that you would like to import. Then click Load.

![Import](https://github.com/DennisFaucher/grafana101/blob/main/images/Dashboards%20-%20Import.png)

Choose InfluxDB as the data source for the dashboard and click "Import"

![Final Import](https://github.com/DennisFaucher/grafana101/blob/main/images/Dashboards%20-%20Final%20Import.png)

You will now have this dashboard and can choose which Linux host whose metrics to use

![Linux Dashboard](https://github.com/DennisFaucher/grafana101/blob/main/images/Linux%20Dashboard.png)

# Thank You

Thank you for taking the time to read this post. Part II will cover how to add VMware metrics and VMware dashboards. Feel free to contact me with any questions or improvements.
