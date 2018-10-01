---
layout:     post
title:      Testing true WiFi speeds
date:       2018-7-7
summary:    Wifi routers claim Gigabits speed, but is it really possible to get
            that kind of speeds. I tried checking in local network what is the
            best speed that can be achieved in regular home conditions.
categories: wifi routers, gigabit routers
---

Test tool: iperf (2.0.5)

Laptop1: Ubuntu 18.04 (Huawei Matebook X Pro, i7 8550U)

Laptop2: Ubuntun 16.04 (HP EliteBook 840 G3, i7)

## Scenario1:

Wifi Router: [Dlink 816](http://www.dlink.co.in/products/?pid=677)

Time: 7pm

<p align="center">
<img src="/images/dlink.png" alt="Dlink 816"/>
</p>

| Sender(laptop1) | Rcvr(laptop2) | Reading1 | Reading2 | Reading3 | Average |
|-----------------|---------------|----------|----------|----------|---------|
| 2.4G | 2.4G | 17.7 | 21.3 | 15.1 | 18.03 |
| 2.4G | 5G | 40.9 | 43.9 | 36.6 | 40.46 |
| 5G | 2.4G | 47.8 | 58.5 | 53.5 | 53.26 |
| 5G | 5G | 42.6 | 45 | 45.1 | 44.23 |
| 5G | Eth | 89.1 | 92.5 | 91.8 | 91.13 |
| 2.4G | Eth | 37.1 | 35.3 | 34.5 | 35.63 |

## Scenario2:

Wifi Router: [NetGear R6800 AC1900 Dual Band Gigabit Wireless Router]()

Time: 7:20pm

<p align="center">
<img src="/images/netgear.png" alt="NetGear R6800"/>
</p>

| Sender(laptop1) | Rcvr(laptop2) | Reading1 | Reading2 | Reading3 | Average |
|-----------------|---------------|----------|----------|----------|---------|
| 2.4G | 2.4G | 53.7 | 52.4 | 48.5 | 51.53 |
| 2.4G | 5G | 104 | 101 | 104 | 103 |
| 5G | 2.4G | 130 | 138 | 135 | 134.33 |
| 5G | 5G | 242 | 243 | 240 | 241.66 |
| 5G | Eth | 444 | 449 | 452 | 448.33 |
| 2.4G | Eth | 99.5 | 102 | 100 | 100.50 |

## Notes:
* Max CPU usage of iperf on sender & receiver (laptop2) < 20%
* The performance of 2.4GHz differed widely between Netgear and Dlink. Netgear hugely out-performed Dlink.
* The performance of 5GHz differed widely between Netgear and Dlink. Netgear hugely out-performed Dlink. Netgear supports advanced 5GHz modes and thus may have produced better results.
* The aim of this experiment was to check if the Netgear can operate at 1Gbps rate with regular (highend) laptops. The best i can achieve was 450Mbps. Netgear advertises 1900 Mbps but i guess these speeds are achievable only with special purpose client devices which supports multiple wifi data streams in parallel.

