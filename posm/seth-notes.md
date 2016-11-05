# Seth Notes
_Fri Jul 22 2016_

## POSM Build

If you have an Etheros card, you can use 802.11n rather than 802.11g. In `settings`, you can enable 802.11n:

    posm_wifi_80211n="1" # set to 1 to enable 802.11n (e.g. ath9k)

## New Hardware

Seth has new hardware that is almost as good as the POSM. It is the Compulab Fitlet -i Barebone.  It has 2 network ports. This is great, because you can connect an access point that allows heavy, industrial grade, wifi client load. We are using the uniFi AP AC LITE.

It gets pretty hot, so it is recommended to buy the extra heatsink.

To get this working, in `settings`, change:

    posm_wan_netif="p2p1"

to
    
    posm_wan_netif="eth0"
    posm_lan_netif="eth1"

This enables the ethernet port in between the antennae to function as a WAN port.
