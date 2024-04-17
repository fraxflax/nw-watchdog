# CHANGELOG
Also see __[WIP-TODO.md](https://github.com/fraxflax/nw-watchdog/blob/main/documentation/WIP-TODO.md)__ for __Work in Progress__ (features / changes currently being worked on) and future plans.

## IN MAIN (unreleased version v1.1.x)
These changes are already merged into the main branch and available in https://github.com/fraxflax/nw-watchdog <br>
Unless plans change, they will be in the next release.

* _currently nothing_

## v1.1.0 - Feature Release, 2024-04-17
https://github.com/fraxflax/nw-watchdog/tree/v1.1.0

* New alert states: __LINKDOWN__ and __LINKUP__<br>
  In version 1.0.0, if there is no link on the interface a WARNING alert is sent causing alerts to be sent repeatedly under certain circumstances (e.g. every _max-nolink * ifup-grace_ second  if the link is physically lost (switch port down, cable damaged / out, etc) and `--force-interface` with a `--max-nolink` greater than 0 is used.<br>
  With these new alert states there will be only one alert when the link is lost and we can't get it up again within _max-nolink__ tries.<br>
  Once the link comes up there will be one LINKUP (or REAHCABLE / UP depending on what is detected firstly) alert.
  
* Information level (`--verbosity-level=4`) logging (instead of WARNING alerts when  conflicting topology detected using `--force-interface`.

* __LIST ALL ERRORS__<br>
  Instead of dying with an error message upon first found error whilst parsing the options, now all options and arguments are parsed and checked firstly, then if there are error(s), all error messages and USAGE instruction is shown.

* New option: __--no-pager | --no-less | --no-more | -M__<br>
  No longer recommending using PAGER=cat not to use pager.<br>
  __--install-systemd__ will enforce __--no-pager__ when installing systemd-services
  
* New option: __--remove-systemd=<ins>SERVICENAME</ins>__.<br>
  (Manual removal proces is also described in `--help`.)

* New option: __--list-systemd__

* Changed logfile and pidfile filename for systemd services:<br>
  `--pidfile=/run/nw-watchdog/SERVICENAME.pid` _(instead of `/run/nw-watchdog-SERVICENAME.pid`)_<br>
  and `--logfile=/var/log/nw-watchdog/SERVICENAME.log` _(instead of `/var/log/nw-watchdog-SERVICENAME.log`)_
  
* __Initial device check__ (addresses the "__False --force-interface UP bug__") <br>
  If an interface is specified with `--interface` or `--force-interface` and the corresponding device is non-existing or has no link, attempts to bring it up or reset it is performed unless `--no-interface-reset` is specified. Using `--force-interface` will now require the interface to come up with link before proceeding. This addresses the bug of when a non-existing device was specified as `--force-interface` the <ins>TARGET</ins> could be considered UP if reachable via another interface.

* New alert state: __INITIAL__<br>
  For alerting of initial problems preventing or delaying the start of the monitoring:<br>
  `--force-interface` non-existing device, cannot be brought up or ha no link, non-existing `--interface` device in combination with `--no-interface-reset`<br>
  and also if initial topology is immediately changed due to specified non-existing / no-link `--interface` could not be brought up with link from the start.
  
* __Options and Arguments Parsing__
All options and arguments are now parsed in a more general approach making it easier and more fool proof to introduce or change options and argument verification, allowing for quicker development less likely to introduce bugs.

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


