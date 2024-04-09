# WORK IN PROGRESS
(and future plans)

## IN MAIN 
These changes are already merged into the main branch and available in https://github.com/fraxflax/nw-watchdog <br>
Unless plans change, they will be in the next release.

* _currently nothing_

## WORK IN PROGRESS 
These changes are currently being worked on in separate branches.

* __LINKDOWN - LINKUP__ alert states<br>
  In version 1.0.0, if there is no link on the interface a WARNING alert is sent, being a WARNING based on the idea of it possibly being a configuration issue with to short `--ifup-grace` or to few tries in `--max-nolink` and it might very well ...<br>
  BUT if the link is lost e.g. due to switch being down, cable damaged or fallen out, etc and we use `--force-interface` with a `--max-nolink` greater than 0, there will be repeated alerts on the same issue sent almost every _max-nolink * ifup-grace_ seconds.<br>
  With these new alert states thare will be only one alert if the link is down and we can't get it up, and one when it comes up.

* __--no-pager | --no-less | --no-more | -M__<br>
  option not to use pager for errors and help
  __--install-systemd__ will enforce __--no-pager__ when installing systemd-services
  will also update help to reflect the new option and no longer recommend using PAGER=cat not to use pageer with __--help__  .

* Introduced __--remove-systemd=<ins>SERVICENAME</ins>__.

* Introduced __--list-systemd=<ins>SERVICENAME</ins>__

* Changed logfile and pidfile when installing systemd service:<br>
  `--pidfile=/run/nw-watchdog/SERVICENAME.pid` _(instead of `/run/nw-watchdog-SERVICENAME.pid`)_<br>
  and `--logfile=/var/log/nw-watchdog/SERVICENAME.log` instead of _(`/var/log/nw-watchdog-SERVICENAME.log`)_

## TODO:
Planned future features / changes.

* MAYBE: Support multiple targets in one instance ????
* The BIG one: IPv6 support
