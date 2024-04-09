# WORK IN PROGRESS
(and future plans)

## IN MAIN 
These changes are already merged into the main branch and available in https://github.com/fraxflax/nw-watchdog <br>
Unless plans change, they will be in the next release.

* _currently nothing_

## WORK IN PROGRESS 
These changes are currently being worked on in separate branches.

* New alert state: __LINKDOWN__<br>
  In version 1.0.0, if there is no link on the interface a WARNING alert is sent, being a WARNING based on the idea of it possibly being a configuration issue with to short `--ifup-grace` or to few tries in `--max-nolink` and it might very well ...<br>
  BUT if the link is lost e.g. due to switch being down, cable damaged or fallen out, etc and we use `--force-interface` with a `--max-nolink` greater than 0, there will be repeated alerts on the same issue sent almost every _max-nolink * ifup-grace_ seconds.<br>
  With this new alert state thare will be only one alert if the link is down and we can't get it up.

* No warning alert for conflicting topology with `--force-interface`
  If conflicting topology is detected whilst using `--force-interface` this will be logged with at `--verbosity-level=4` (=info, which is the default) or higher, but no alert will be sent (to avoid repeated alerts on the same problem).

* __LIST ALL ERRORS__<br>
  Instead of dying with an error message immediately when an argument/option error is found, parse all options and arguments firstly, then list all errors and show USAGE

* New option: __--no-pager | --no-less | --no-more | -M__<br>
  No longer recommending using PAGER=cat not to use pager.<br>
  __--install-systemd__ will enforce __--no-pager__ when installing systemd-services
  
* New option: __--remove-systemd=<ins>SERVICENAME</ins>__.

* New option: __--list-systemd__

* Changed logfile and pidfile for systemd services:<br>
  `--pidfile=/run/nw-watchdog/SERVICENAME.pid` _(instead of `/run/nw-watchdog-SERVICENAME.pid`)_<br>
  and `--logfile=/var/log/nw-watchdog/SERVICENAME.log` _(instead of `/var/log/nw-watchdog-SERVICENAME.log`)_

## TODO
Planned future features / changes.

* MAYBE: Support multiple targets in one instance ????
* The BIG one: IPv6 support
