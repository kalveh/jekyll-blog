---
layout: post
title: 使用电脑上的无线网卡分析WiFi协议
categories: 流量分析
description: 
tags: [WiFi, GNURadio]
topmost: false
---

在电脑上使用无线网卡分析WiFi协议，需要将网卡设置为monitor模式。如果在虚拟机里用网卡抓取空中的数据包，由于它无法识别内置的无线网卡，所以需要接外置的USB网卡，以EDIMAX AC1750为例： 

``` 

sudo apt-get install dkms 

git clone -b v5.6.4.1 [https://github.com/aircrack-ng/rtl8812au.git](https://github.com/aircrack-ng/rtl8812au.git)

cd rtl8812au 

sudo ./dkms-install.sh 

```

该网卡驱动在Ubuntu 16.04系统下似乎有些问题，连接WiFi网络总是不成功，但设置为monitor模式勉强可以工作。 

换一个直接针对该网卡芯片rtl8814au的驱动，可正常工作： 

``` 

git clone [https://github.com/tpircher/rtl8814AU.git](https://github.com/tpircher/rtl8814AU.git)

cd rtl8814AU 

make 

sudo make install 

```

在Makefile文件中有关于monitor模式的配置项： 

``` 

CONFIG_WIIF_MONITOR = n 

```

设置为y(yes)或n(no)似乎没有影响。随后使用如下方法设置wlan0网口（根据iwconfig的提示更改网卡名称）为monitor模式： 

``` 

sudo ifconfig wlan0 down 

sudo iwconfig wlan0 mode monitor 

## sudo iwconfig wlan0 channel 157 

sudo ifconfig wlan0 up 

```

注意：该驱动似乎还是有些Bug，在设置为Monitor模式后，用iwconfig查看一直显示为Auto，不过不影响上网和抓包。 

此外，wireshark建议用如下方法设置监听端口： 

``` 

sudo apt-get install aircrack-ng 

sudo airmon start wlan0 

```

该方法会生成一个monitor模式的虚拟网卡mon0，供抓包使用。但经实测，在EDIMAX AC1750的上述驱动，不支持该操作。 


手头还有一块Netgear A6210无线网卡，采用如下方法安装驱动： 

``` 

git clone -b port-to-4.15 [https://github.com/kaduke/Netgear-A6210.git](https://github.com/kaduke/Netgear-A6210.git)

cd Netgear-A6210 

make 

sudo make install 

```

该方法能够正确加载Netgear A6210网卡，但不能以monitor模式工作。经查找资料，可使用如下git源： 

``` 

git clone [https://github.com/cyangy/A6210-mt76x2u_Ubuntu.git](https://github.com/cyangy/A6210-mt76x2u_Ubuntu.git)

cd A6210-mt76x2u_Ubuntu 

make -j $(expr $(nproc) + 1) 

sudo make install 

```

拔出并重新插上网卡，即可正确识别： 

``` 

sudo service network-manager restart 

iwconfig 

```

然后可以将网卡设置为monitor模式，并使用Wireshark进行空中抓包，包括Beacon，Probe Request/Response等，注意在monitor模式下网卡不需要连接AP。 


设置Wireshark的Filter条件： 

``` 

wlanradio.channel == 11 (2462MHz) 

wlanradio.channel == 157 (5785MHz) 

wlan.fc.type_subtype == 0x8 (Beacon) 

wlan.fc.type_subtype == 0x4 (Probe Request) 

wlan.fc.type_subtype == 0x5 (Probe Response) 

wlan.fc.type_subtype == 0x28 (QoS Data) 

wlan.sa == 12:69:6c:6e:e1:28 (Source Address) 

wlan.da == ff:ff:ff:ff:ff:ff (Destination Address) 

wlan.ta == 12:69:6c:6e:e1:28 (Transmitter Address) 

wlan.da == ff:ff:ff:ff:ff:ff (Receiver Address) 

wlan.addr == 74:da:38:d3:a4:90 (Edimax网卡) 

wlan.ssid == "seu-wlan-5G" 

wlan.fixed.beacon == 100 (0.1024 Seconds) 

```

设置IEEE 802.11解密密钥： 

Edit->Preferences->IEEE 802.11->Decription Keys->Edit 

``` 

wep:a1:b2:c3:d4:e5 

wpa-pwd:MyPassword:MySSID 

wpa-psk:0102030405060708091011...6061626364 

```

**See also:** 

Wireshark: How to Decrypt 802.11 

[https://wiki.wireshark.org/HowToDecrypt802.11](https://wiki.wireshark.org/HowToDecrypt802.11)


Note: WPA and WPA2 use keys derived from an EAPOL handshake, which occurs when a machine joins a Wi-Fi network, to encrypt traffic. Unless all four handshake packets are present for the session you're trying to decrypt, Wireshark won't be able to decrypt the traffic. You can use the display filter eapol to locate EAPOL packets in your capture. 


{:toc}
