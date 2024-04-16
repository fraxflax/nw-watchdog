# CHANGELOG
Also see __[WIP-TODO.md](https://github.com/fraxflax/nw-watchdog/blob/main/documentation/WIP-TODO.md)__ for __Work in Progress__ (features / changes currently being worked on) and future plans.

## IN MAIN (unreleased version 1.1.0-beta)
These changes are already merged into the main branch and available in https://github.com/fraxflax/nw-watchdog <br>
Unless plans change, they will be in the next release.

* New alert states: __LINKDOWN__ and __LINKUP__<br>
  In version 1.0.0, if there is no link on the interface a WARNING alert is sent, being a WARNING based on the idea of it possibly being a configuration issue with to short `--ifup-grace` or to few tries in `--max-nolink` and it might very well ...<br>
  BUT if the link is lost e.g. due to switch being down, cable damaged or fallen out, etc and we use `--force-interface` with a `--max-nolink` greater than 0, there will be repeated alerts on the same issue sent almost every _max-nolink * ifup-grace_ seconds.<br>
  With these new alert states thare will be only one alert if the link is down and we can't get it up.<br>
  If nw-watchdog manages to get the link up there will be one LINKUP alert.<br>
  If the link comes up for other reasons there might be an UP or REACHABLE alert before the link is checked (which both implies LINKUP).

* No warning alert for conflicting topology with `--force-interface`
  If conflicting topology is detected whilst using `--force-interface` this will be logged with at `--verbosity-level=4` (=info, which is the default) or higher but, to avoid repeated alerts on the same problem, no alert will be sent.

* __LIST ALL ERRORS__<br>
  Instead of dying with an error message upon first found error, parsing the options, now all options and arguments are parsed and checked firstly, then if there are error(s) list all errors messages and show USAGE.

* New option: __--no-pager | --no-less | --no-more | -M__<br>
  No longer recommending using PAGER=cat not to use pager.<br>
  __--install-systemd__ will enforce __--no-pager__ when installing systemd-services
  
* New option: __--remove-systemd=<ins>SERVICENAME</ins>__.

* New option: __--list-systemd__

* Changed logfile and pidfile for systemd services:<br>
  `--pidfile=/run/nw-watchdog/SERVICENAME.pid` _(instead of `/run/nw-watchdog-SERVICENAME.pid`)_<br>
  and `--logfile=/var/log/nw-watchdog/SERVICENAME.log` _(instead of `/var/log/nw-watchdog-SERVICENAME.log`)_
  
* New alert state: __INITIAL__<br>
  For alerting of initial problems that needs to be resolved before proceeding.
  
* __Initial device check__<br>
  If an interface is specified with `--interface` or `--force-interface` an initial check for existance of the device is made.
  Attempts to bring it up is performed unless `--no-interface-reset` is specified. Using `--force-interface` will require the interface to come up before proceeding. This addresses the bug of when a non existing interface was specified as `--force-interface` TARGET could be considered UP if reachable via other paths.

* __Options and Arguments Parsing__
All options and arguments are now parsed in a more general approach making it easier and more fool proof to introduce or change options and possible argument values allowing for quicker development with less bugs introduced.

## v1.0.0 - First stable Release, 2024-04-03
https://github.com/fraxflax/nw-watchdog/tree/v1.0.0 <br>
Download release: https://github.com/fraxflax/nw-watchdog/releases/tag/v1.0.0 <br>
Stability tested thoroughly.<br>
POSIX and dependency checks are in place.<br>
All functionality is documented.<br>
Documentation is up to date with sufficient number of EXAMPLES to show the functionality.

Please see
[v1.0.0 README](https://github.com/fraxflax/nw-watchdog/blob/v1.0.0/README.md),
or [v1.0.0 documentation/help.md](https://github.com/fraxflax/nw-watchdog/blob/v1.0.0/documentation/help.md) 
for capabilities / fetures.


