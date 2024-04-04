# CHANGELOG

## v. 1.0.0 - First stable Release
https://github.com/fraxflax/nw-watchdog/releases/tag/v1.0.0 <br>
Stability tested thoroughly.<br>
POSIX and dependency checks are in place.<br>
All functionality is documented.<br>
Documentation is up to date with sufficient number of EXAMPLES to show the functionality.

Please see
[Release-version-1.0.0/README.md](https://github.com/fraxflax/nw-watchdog/blob/Release-version-1.0.0/README.md),
or [Release-version-1.0.0/documentation/help.md](https://github.com/fraxflax/nw-watchdog/blob/Release-version-1.0.0/documentation/help.md) 
for capabilities / fetures.

## IN MAIN 
These changes are already merged into the main branch and available in https://github.com/fraxflax/nw-watchdog <br>
Unless plans change, they will be in the next release.

* currently nothing

## WORK IN PROGRESS 
These changes are currently worked on in a separate branches.

* __LINKDOWN - LINKUP_ alert statees<br>
  In version 1.0.0, if there is no link on the interface a WARNING alert is sent with the idea of this could possibly be a configuration issue with to short --ifup-grace or tweaking --max-nolink BUT if the link is lost e.g. due to switch being down, cable out, etc and we use --force-interface and --max-nolink > 0 there will be repeated alerts on the same issue sent every ~ <ins>max-nolink * ifup-grace</ins> seconds.<br>
  With the new alert states thare will be only one alert if the link is down and we can't get it up and one when it comes up again.

## TODO:
Planned future features / changes.

* currently nothing
