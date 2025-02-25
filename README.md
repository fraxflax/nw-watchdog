# nw-watchdog
__nw-watchdog__ is a higly configurable network watchdog written in [POSIX](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/contents.html) shell script for use in Linux, depending only on Linux most standard tools that are normally installed by default in all distributions (also see the __[DEPENDENCIES](https://github.com/fraxflax/nw-watchdog/blob/main/documentation/help.md#deps)__ section).

It monitors the network connectivity to a specified <ins>TARGET</ins> and/or the next hop towards that <ins>TARGET</ins>, alerting upon lost connectivity explaining what is wrong. It can reset the source interface and will detect topology changes and, if allowed, reconfigure itself accordingly. It's intended to run as a daemon and has an option to install itself as a systemd service.  If you want to monitor the connectivity to several <ins>TARGET</ins>s, you can run several instances of __nw-watchdog__ using different `--pidfile` option arguments.

__nw-watchdog__ is free software written by Fredrik Ax \<nw-watchdog@axnet.nu\>.<br>
Feel free to modify and/or (re)distribute it in any way you like.<br>
... it's always nice to be mentioned though ;-)<br>

__nw-watchdog__ comes with ABSOLUTELY NO WARRANTY.

If you expirence any problems with __nw-watchdog__, are lacking any functionality or just want to voice your opions about it, feel free to contact me via e-mail.

## INSTALL
Just copy / download the `nw-watchdog` script into your PATH and make it executable.<br>

* Latest release of the script is version 1.1.6.
  
  Download v1.1.6: [nw-watchdog](https://raw.githubusercontent.com/fraxflax/nw-watchdog/v1.1.6/nw-watchdog)
  ```
  curl -o /usr/local/bin/nw-watchdog https://raw.githubusercontent.com/fraxflax/nw-watchdog/v1.1.6/nw-watchdog ; chmod a+rx /usr/local/bin/nw-watchdog
  ```
* All releases are available for download here: https://github.com/fraxflax/nw-watchdog/releases

* The latest TESTING version (unreleased main branch) of the script may contain new features / changes that will be in the next release. If so, they are documented in [documentation/changelog.md](https://github.com/fraxflax/nw-watchdog/blob/main/documentation/changelog.md).
  
  Download TESTING: [nw-watchdog](https://raw.githubusercontent.com/fraxflax/nw-watchdog/main/nw-watchdog)
  ```
  curl -o /usr/local/bin/nw-watchdog https://raw.githubusercontent.com/fraxflax/nw-watchdog/main/nw-watchdog ; chmod a+rx /usr/local/bin/nw-watchdog
  ```

## DOCUMENTATION
See [documentation/help.md](https://github.com/fraxflax/nw-watchdog/blob/main/documentation/help.md) for extensive documentation including examples.<br>
The same documentation is also available by running:<br>
`nw-watchdog --help`<br>

## Other network utils
Also check out https://github.com/fraxflax/frax-net-utils/ for other network utilities written by me.
