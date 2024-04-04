# WORK IN PROGRESS
(and future plans)

## IN MAIN 
These changes are already merged into the main branch and available in https://github.com/fraxflax/nw-watchdog <br>
Unless plans change, they will be in the next release.

* _currently nothing_

## WORK IN PROGRESS 
These changes are currently being worked on in separate branches.

* __LINKDOWN - LINKUP__ alert statees<br>
  In version 1.0.0, if there is no link on the interface a WARNING alert is sent with the idea of this could possibly be a configuration issue with to short --ifup-grace or tweaking --max-nolink BUT if the link is lost e.g. due to switch being down, cable out, etc and we use --force-interface and --max-nolink > 0 there will be repeated alerts on the same issue sent every ~ <ins>max-nolink * ifup-grace</ins> seconds.<br>
  With the new alert states thare will be only one alert if the link is down and we can't get it up and one when it comes up.

## TODO:
Planned future features / changes.

* MAYBE: Support multiple targets in one instance ????
* The BIG one: IPv6 support
