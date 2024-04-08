## NAME
__nw-watchdog__ - Network Watchdog

## SYNOPSIS
__nw-watchdog__ <ins>TARGET</ins> [ OPTIONS ]

__nw-watchdog__ __--list-systemd__

__nw-watchdog__ __--remove-systemd__ <ins>SERVICENAME</ins> | <ins>UNITNAME<ins>

## DESCRIPTION
__nw-watchdog__ is a higly configurable network watchdog written in POSIX shell script for use in Linux, depending only on Linux most standard tools that are normally installed by default in all distributions (also see the __DEPENDENCIES__ section).

It monitors the network connectivity to a specified <ins>TARGET</ins> and/or the next hop towards that <ins>TARGET</ins>, alerting upon lost connectivity explaining what is wrong. It can reset the source interface and will detect topology changes and, if allowed, reconfigure itself accordingly. It's intended to run as a daemon and has an option to install itself as a systemd service.  If you want to monitor the connectivity to several <ins>TARGET</ins>s, you can run several instances of __nw-watchdog__ using different `--pidfile` option arguments.

__nw-watchdog__ is free software written by Fredrik Ax \<frax@axnet.nu\>.<br>
Feel free to modify and/or (re)distribute it in any way you like.<br>
... it's always nice to be mentioned though ;-)<br>

__nw-watchdog__ comes with ABSOLUTELY NO WARRANTY.

Get the latest version from https://github.com/fraxflax/nw-watchdog

## <INS>TARGET</INS>
The mandatory (unless `--list-systemd` or `--remove-systemd` is specified) argument <ins>TARGET</ins> is the target (destination) for which he connection is monitored. <ins>TARGET</ins> can be an IP address or a resolvable hostname / FQDN. If it's a hostname / FQDN, it will be resolved to an IPv4 address (first one found by getent). The resolved IP address will be used for the monitoring. The name is continuously resolved and if the resolved ip address changes the new IP address will be used for the monitoring from there on.<br>
Use `--no-continuous-topology-detect` to resolve the <ins>TARGET</ins> only at startup and failed connectivity checks.

<ins>TARGET</ins> can be placed before, after or between valid OPTIONS.


## OPTIONS (no arguments)
These options take no arguments, and may be specified in any order. They can be grouped (e.g. `-vAP`) in their short form, also having one of the OPTIONS that requires an argument last in the group.

* __--help | -h__
	Shows this help, using `$PAGER` if set to an executable, otherwise `less` or `more` if available in `/sbin:/bin:/usr/sbin:/usr/bin:$PATH`<br>
(Use `--no-pager` to avoid using a pager).

* __--no-pager | --no-less | --no-more | -M__
Do NOT use a pager for help and error messages.
__--install-systemd__ implies __--no-pager__

* __--no-ping-target | -P__
	If the <ins>TARGET</ins> is the next hop (on the same subnet), reachability of the <ins>TARGET</ins> is checked by arp cache status and ping.<br>
If the <ins>TARGET</ins> is not on the same subnet as the source, the reachability of the <ins>TARGET</ins> is checked by pinging it in a certain pattern (see `--slow-up-timeout` for details).

	`--no-ping-target` disables the ping-checks for the <ins>TARGET</ins>. Only connectivity to the NEXTHOP for the <ins>TARGET</ins> is checked.<br>
	This can be useful if <ins>TARGET</ins> does not reply to ping, or if it desirable to only alert if there is no route to the <ins>TARGET</ins> or NEXTHOP is unreachable.

	`--no-ping-target` cannot be used in combination with `--no-ping-nexthop`.

* __--no-ping-nexthop | -N | --no-ping-gateway | -G__<br>
	By default, if the connectivity to the <ins>TARGET</ins> cannot be verified, and the next hop (NEXTHOP) is not the <ins>TARGET</ins> itself, the reachability of the NEXTHOP (usually a gateway) is checked, firstly by checking it's status in the arp cache and then by pinging it, rechecking the arp cache status upon failed ping. 

	`--no-ping-nexthop` disbles the reachaility check for the NEXTHOP so only connectivity to <ins>TARGET</ins> itself is checked. Useful if the NEXTHOP is a peer-to-peer address and not setup to reply to ping.

	`--no-ping-nexthop` cannot be used in combination with `--no-ping-target`.

* __--no-ipaddr-alert | -A__<br>
	Do not alert for not finding any global scope ip addresses on the source interface.

* __--no-interface-reset | -R__<br>
  Do not try to bring down and up interface after failed connectivity checks.<br>
  (Do not try to "repair" the connection", just monitor it.)

* __--no-continuous-topology-detect | -T__<br>
  Normaly the topology (resolving the ip address of the <ins>TARGET</ins>, detecting which source interface to use and the ip address of the NEXTHOP towards the <ins>TARGET</ins>) is detected at startup and continuously monitored for changes.

	`--no-continuous-topology-detect` disables the topology detection for as long as the <ins>TARGET</ins> replies (or in combination with `--no-ping-target`, for as long as the NEXTHOP is reachable). The topology will only be detected at startup and if the <INS>TARGET</INS> does not reply or if the NEXTHOP cannot be reached, meaning that routing changes making the <INS>TARGET</INS> or NEXTHOP unreachable will not be detected as long as the <INS>TARGET</INS> / NEXTHOP can be reached using the old topology.

	`--force-interface` implies `--no-continuous-topology-detect`.
    				       
* __--foreground | -f | --no-daemonize | -D__<br>
	Run in foreground, do not fork / daemonize.

* __--list-systemd__<br>
    If systemd is installed, a brief status of all installed nw-watchdog systemd-services are listed to stdout.

	Cannot be combined with any other options.
	
* __--verbose | -v__<br>
	Shortcut for `--verbosity-level=5`<br>
	If used in combination with `--verbosity-level`, the specified __--verbosity-level__ will take precedence. 

* __--debug | -d__<br>
	Shortcut for: `--verbosity-level=6  --logfile=- --logsize=0  --pidfile=/dev/null  --slow-up-timeout=1  --sleep=3 --ifup-grace=5 --alert='cat' --foreground` 

	If it's combined with any of the options it provides shortcuts for, the specified option will take precedence over the `--debug` shortcut.

	This option cannot be combined with `--install-systemd` (but it would be wise to test the configuration with `--debug` before installing as a systemd service).

## OPTIONS (with ARGUMENT)
These opions takes a single argument each and may be specified in any order. Specify with equalsign or space or no space between option and argument. They can only be grouped together with the shortform of the NO-ARGUMENT-OPTIONS above, and must be last in such groupings (e.g. `-PAV5`).

* __--verbosity-level | -V__ <ins>level</ins><br>
	Default: `4`<br>
	<ins>level</ins> must be an integer greater than or equal to zero.
	Determines how much info is logged to the logfile and which alerts are triggered.

	__0__ - none<br>
	No output to logfile and no alerts triggered at all ... pretty useless unless you just want to keep traffic going keeping a connecting alive without any alerts.

	__1__ - error<br>
	Only logs and alerts on errors causing the __nw-watchdog__ not to function as intended.

	__2__ - warning<br>
	Also logs and alerts on warnings about configuration, etc.

	__3__ - alert<br>
	Log and alert on irreparable connectivity failures, interface down, and other things disrupting the monitored connection, as well as on warnings and errors.<br>
	Also see the `--alert` option.

	__4__ - info (default)<br>
	Same as level 3, but also logs some useful information on what's going on, such as topology changes, some test failures forcing more testing,  interface resets, etc

	__5__ - trace<br>
	Logs even more info about which action is currently performed, including, all secondary tests that are run, when sleeping longer than usual, etc.

	__6__ - debug<br>
	Logs even more internal details, e.g. SETTINGS used, all tests performed, all sleeps, and other debug info.

* __--interface | -i__ <ins>interface</ins><br>
  Default: none<br>
  <ins>interface</ins> is the name of the source interface to initially use.

	The interface may dynamically change due to topology detection. If you want to force the use of a specific interface, use `--force-interface` instead.

	If neither of `--interface` or `--force-interface` is specified the source interface it will be determined from the FIB by looking at the route to the <ins>TARGET</ins>. The reason to specify it even so, would be to have __nw-watchdog__ bring it up if it's down when starting.

	`--interface` cannot be combined with `--force-interface`.

* __--force-interface | -I__ <ins>interface</ins><br>
  Default: none<br>
  <ins>interface</ins> is the name of the source interface to always use.

	Packets will always be sent from this interface. The forwadring table will be ignored as well as conflicting topology changes.<br>
	This is useful for monitoring the preferred path and making sure it's up. It does not check if you have connectivity to the <ins>TARGET</ins> via any other path.

	Implies `--no-continuous-topology-detect`.

	`--force-interface` cannot be combined with `--interface`.

* __--logfile | -l__ <ins>logfile</ins><br>
	Default: `/var/log/nw-watchdog.log`<br>
	Logfile to use. If specified as '-' logs are written to stdout.

* __--logsize | -z__ <ins>size</ins><br>
  Default: `0`

  If the logfile grows beyond this size, the oldest entries will be removed.

  You can use suffixes K, M, G for kilo / mega / giga bytes. (No suffix is same as K).<br>
  Set to 0 for unlimited logfile size (which you would want if you do log rotation). 

  If the logfile is set to '-' (stdout) this option is ignored.

  If `flock` is available, the logfile will be locked before written to or shrinked, otherwise there is a slight risk of log entries being lost if two or more instances of __nw-watchdog__ are concurently running using the same logfile and at least one of them have `--logsize` set to a value larger than 0.

* __--pidfile | -p__ <ins>pidfile</ins><br>
  Default: `/run/nw-watchdog.pid`<br>
  Pidfile to use.

* __--slow-up-timeout | -t__ <ins>seconds</ins><br>
  Default: `3`<br>
  <ins>seconds</ins> must be an integer greater than zero.

  The check whether the <ins>TARGET</ins> is up or not, is performed in several steps.<br>
  First a "quick-up" test sends one single ICMP echo packet waiting for the reply for no more than 1 second. If that fails, a more thourough "slow-up" test sends 5 ICMP echos.

  `--slow-up-timeout` controls he TIMEOUT for waiting on each packet in the slow-up test.<br>
  5 packets are always sent in the slow-up test.<br>
  The packets are sent adaptively, meaning that as soon as a reply is received the next packet is sent without delay, giving slow-up a total time of 5 * RTT to the <ins>TARGET</ins> if the connection is up.<br>
  The DEADLINE = TIMEOUT * 5 is the maximum time the slow-up test will take if the <ins>TARGET</ins> is down.
        
  `--slow-up-timeout=7` is useful for monitoring VPN connections via interfaces that need a long wake-up time if idle (due to regotiation of encryption, exchanging keys, reauthentication, etc).

  `--slow-up-timeout=3` is useful for monitoring connections to <ins>TARGET</ins>s with low to medium latency via interfaces that does not need a long wake-up time (e.g. ethernet interfaces).

  `--slow-up-timeout=1` is suitable to use for monitoring local <ins>TARGET</ins>s (e.g. NEXTHOP) on ethernet carried subnets.

* __--sleep | -s | --interval__ <ins>seconds</ins><br>
  Default: `10`<br>
  <ins>seconds</ins> must be an integer greater than zero.

  How many seconds to sleep after sucessful ping check. 

* __--ifup-grace | -g__ <ins>seconds</ins><br>
  Default: `20`<br>
  <ins>seconds</ins> must be an integer greater than zero.

  How many seconds to sleep before next check after interface has been reset.

* __--max-nolink | -n__ <ins>number</ins><br>
  Default: `1`<br>
  <ins>number</ins> must be an integer greater than or equal to zero.

  Maximum number of consecutive failed link checks in which the interface have been reset (brought down and up again) before doing new topology check.

  A word of warning: If set to 0 and interface is not up / goes down, infinite retries to bring the interface up will be made before checking topology. Only set it to 0 if you are sure that the specified interface should always be used and you want to make sure it's up before starting to monitor the connection.<br>
  Typically, you would want to also use `--force-interface` when using `--max-nolink=0`.

* __--ifcup | -u__ <ins>STRING</ins> <br>
  Default: `ip link set up %{IFC}`<br>
  <ins>STRING</ins> will be passed to 'sh -c' to bring the interface up.<br>
  %{IFC} will be dynmaically replaced with the interface name currently in use as source interface.

  Examples:
  - ifupdown:<br>
    `--ifcup='ifup %{IFC}'`

  - ifupdown, non privilege user running __nw-watchdog__:<br>
	`--ifcup='sudo ifup %{IFC}'`

  - NetworkManager device:<br>
	`--ifcup='nmcli device up %{IFC}'`

  - NetworkManager connection:<br>
	`--ifcup='nmcli connection up connection-name'`

  - iproute2 + isc-dhcp-client:<br>
	`--ifcup='ip link set %{IFC} up && dhclient -pf /run/dhclient-%{IFC}.pid %{IFC}'`

  - strongSwan IPSec (setup for IPSec policy routing):<br>
	`--ifcup='ipsec up connection-name'`<br>
	(see __EXAMPLES__  below for a more extensive IPSec example using vti tunnel interface)

* __--ifcdown | -U__ <ins>STRING</ins><br>
  Default: `ip link set down %{IFC}`<br>
  <ins>STRING</ins> will be passed to 'sh -c' to bring the interface down.<br>
  %{IFC} will be dynmaically replaced with the interface name currently in use as source interface.

  Examples:
  - ifupdown:<br>
	`--ifcdown='ifdown %{IFC}'`

  - ifupdown, non privilege user running __nw-watchdog__:<br>
	`--ifcdown='sudo ifdown %{IFC}'`

  - NetworkManager device:<br>
	`--ifcdown='nmcli device down %{IFC}'`

  - NetworkManager connection:<br>
	`--ifcdown='nmcli connection down %{IFC}-connection-name'`

  - iproute2 + isc-dhcp-client:<br>
	{--ifcdown='kill `cat /run/dhclient-%{IFC}.pid\` ; ip link set down %{IFC}'}

  - strongSwan IPSec (setup for IPSec policy routing):<br>
	`--ifcup='ipsec down connection-name'`<br>
	(see __EXAMPLES__  below for a more extensive IPSec example using vti tunnel interface)

* __--alert | -a__ <ins>STRING</ins><br>
	Default: `if which wall >/dev/null; then exec wall; else cat 1>&2; fi`

	Errors, warnings and alerts regarding change of state will be piped to:<br>
	sh -c '<ins>STRING</ins>'

	Within <ins>STRING</ins> the following placeholders can be used:		
	- %{IFC} - the interface name
	- %{TARGET} - the <ins>TARGET</ins> for which the connection is monitored
	- %{TADDR} - the IP address of the <INS>TARGET</INS>
    - %{NEXTHOP}  - the IP address of the NEXTHOP towards <ins>TARGET</ins>
	- %{STATE} - the state of the alert:
	  - UP for restored connectivity to <INS>TARGET</INS> (implies REACHABLE)
	  - DOWN for lost conenctivity to <INS>TARGET</INS>
	  - UNREACHABLE for lost conenctivity to NEXTHOP (implies DOWN)
	  - REACHABLE for restored connectivity to NEXTHOP
	  - LINKDOWN for lost link on source interface
	  - LINKUP for restored link on source interface
	  - ERROR for permanent errors
	  - WARNING for things that might need reconfiguration

	The alert command will be launched for every ERROR and WARNING, even repeated ones.<br>
	For the other states (UP, DOWN, UNREACHABLE, REACHABLE) the alert command will be launched only upon state change.<br>
	E.g. if the state goes from UP to DOWN the DOWN alert is triggered, but if the next check is also DOWN, no further alert will be sent (until the state changes again).

	EXAMPLE of how to email the alert messages using mailx (mailutils) with a custom from address:<br>
	`--alert='mailx -a "From: nw-watchdog@this.hst" -s "nw-watchdog %{STATE} alert for %{TARGET} via %{IFC}" my@email.adr'`

* __--install-systemd__ <ins>SERVICENAME</ins><br>
  Default: none
	
	If systemd is installed, a systemd service file is written to `/etc/systemd/system/nw-watchdog-SERVICENAME.service` launching __nw-watchdog__ as a daemon with<br>
  `--pidfile=/run/nw-watchdog/SERVICENAME.pid`<br>
  `--logfile=/var/log/nw-watchdog/SERVICENAME.log`<br>
	and otherwise with the exact same options as run (apart from the `--install-systemd` option itself of course).

	If `/etc/systemd/system/nw-watchdog-SERVICENAME.service` already exists, the service will be stopped and the file overwritten.

	It will then enable, start it and show status of the newly created nw-watchdog-<ins>SERVICENAME</ins>.service.

	The <ins>SERVICENAME</ins> must consist of at least 1 valid character ( 'a-z', 'A-Z', '0-9', '-' and '_' ) and be no longer than 236 characters.

    This option requires root privileges.

* __--remove-systemd__ <ins>SERVICENAME</ins> | <ins>UNITNAME</ins><br>
    Default: none<br>
	Specify either the short <ins>SERVICENAME</ins> or the full <ins>UNITNAME<ins> (`nw-watchdog-SERVICENAME.service`) to remove.
	Stops and completely removes the `nw-watchdog-SERVICENAME.service` from the system if systemd is installed.

	This option requires root privileges and cannot be combined with any other options.

	Note: The logfile, `/var/log/nw-watchdog/SERVICENAME.log`, will NOT be removed by `--remove-systemd`
	To manually remove the logfile do:
	```shell
	sudo rm `/var/log/nw-watchdog/SERVICENAME.log`
	```

	To manually (instead of using `--remove-systemd`) remove the `nw-watchdog-SERVICENAME.service` systemd service completely do:
	```shell
	sudo systemctl stop nw-watchdog-SERVICENAME.service
	sudo systemctl disable nw-watchdog-SERVICENAME.service
	sudo rm /etc/systemd/system/nw-watchdog-SERVICENAME.service
	sudo systemctl daemon-reload
	```

## EXAMPLES

#### <ins>ISP gateway monitoring:</ins>
```shell
nw-watchdog 1.2.3.4 \
--interval=60 \
--no-ping-target \
--interface=__eth0 \
--slow-up-timeout=1 \
--ifup-grace=300
```
Checking once a minute (`--interval=60`) that we have connectivity to the Internet Service Provider's gateway without actually pinging anything on the Internet (`--no-ping-target` `1.2.3.4` ... any Intenet address will do) allowing interface and topology detection (not using --force-interface or --max-no-link=0) but still, if down, bring the supposed initial interface towards the ISP up on startup (`--interface=eth0`) expecting the ISP gateway to have an RTT below 1 second (`--slow-up-timeout=1`) and allowing the interface to be down for up to 5 minutes before considering it a permanent error rechecking topology (`--ifup-grace=300`).


#### <ins>ISP gateway monitoring with forced interface:</ins>
```shell
nw-watchdog 1.2.3.4 \
--no-ping-target \
--force-interface=eth0 \
--slow-up-timeout=1 \
--ifup-grace=30 \
--max-no-link=0 \
--interval=10
```
Same as above but enforcing the use of the eth0 interface as we know that the ISP gateway should always be reachable via that interface and that is the only Internet facing interface we have. It must be brought up on startup if not already up and there is no use rechecking the topology if it's not up (`--force-interface=eth0`), so if down, we retry to reset it every 30 seconds (`--ifup-grace=30`) forever (`--max-no-link=0`), checking the connectivity every 10 seconds (`--interval=10`).


#### <ins>Management of Strongswan IPSec with VTI tunnel interface:</ins>
Firstly, we start a __nw-watchdog__ for monitoring the connectivity to the IPSec peers public address (1.2.3.4 in this example), emailing alerts to the admin.

```shell
nw-watchdog 1.2.3.4 \
--verbosity-level=3 \
--alert='mailx -a "From: nwwatchdog@`hostname -f`" -s "IPSec Peer 1.2.3.4 %{STATE} via %{IFC}" admin@`cat /etc/mailname\`' \
--slow-up-timeout=2 \
--ifup-grace=10 \
--pidfile=/run/nw-watchdog-ipsecserver.pid
```

Then we setup the monitor for the IPSec VTI tunnel which will also be created, configured and brought up.<br>
Note that we are using different `--pidfile` for the two watchdogs allowing them to run simultaneously.<br>
It would be smoother to use a script and `--ifcup=/path/script`, but it can also be done like this:

```shell
nw-watchdog 169.254.0.1 \
--alert='mailx -a "From: nwwatchdog@`hostname -f`" -s "IPSec connection-name %{STATE} via %{IFC}" admin@`cat /etc/mailname\`' \
--force-interface=ipsec0 \
--slow-up-timeout=5 \
--ifup-grace=25 \
--ifcup='ipsec up connection-name ;
         ip tunnel add %{IFC} mode vti local 4.3.2.1 remote 1.2.3.4 ttl 255 key 111 ;
         ip addr add dev %{IFC} 169.254.0.2 remote 169.254.0.1 ;
         ip link set %{IFC} up multicast on mtu 1424 state UP ;
         ip route add 10.10.0.0/20 via 169.254.0.1 dev %{IFC}' \
--ifcdown='ipsec down connection-name ; ip tunnel del %{IFC}' \
--pidfile=/run/nw-watchdog-ipsectunnel.pid
```

__OBSERVE__ that the entire `--ifcup=` command needs to be on a single line without line breaks (linebreaks added above for readability):
```
--ifcup='ipsec up connection-name ; ip tunnel add %{IFC} mode vti local 4.3.2.1 remote 1.2.3.4 ttl 255 key 111 ; ip addr add dev %{IFC} 169.254.0.2 remote 169.254.0.1 ; ip link set %{IFC} up multicast on mtu 1424 state UP ; ip route add 10.10.0.0/20 via 169.254.0.1 dev %{IFC}'
```


We use `--ifup-grace=25` to allow enough time for IPSec to establish the connection upon start and interface reset.
`--slow-up-timeout=5` (making the deadline 25 seconds for the slow-up ping test) should be enough for an idle IPSec conmnection to wakeup and start forwarding packets when monitored.

In the above example we monitor the connectivity to the peer address inside the VTI tunnel, which is also the NEXTHOP.
If we are interested in the connectivity to something in the remote network routed via the tunnel we could use that as a <ins>TARGET</ins> instead of the peer address:

`nw-watchdog 10.10.1.1 ...`<br>
and the rest of the options exactly the same as above.

This would give alerts if `10.10.1.1` is down.<br>
In all other ways the effect would be the same as using `169.254.0.1` as <ins>TARGET</ins>.<br>
Even with <ins>TARGET</ins> `10.10.1.1`, the NEXTHOP (`169.254.0.1`) will also be monitored (we are __NOT__ using --no-ping-nexthop) and the interface will __NOT__ be reset as long as the NEXTHOP is reachable.


#### <ins>Wireguard full tunnel management:</ins>
This is an example of how one can use __nw-watchdog__ to setup and monitor a wireguard full tunnel, also monitoring the connectivity to the wireguard server, running both whatchdogs as systemd services getting alerts via e-mail:

Firstly, we setup the __nw-watchdog__ systemd service for the wireguard server which we reach via the default route:

```shell
nw-watchdog wgserver.domain.dom \
--verbosity-level=3 \
--alert='mailx -a "From: nwwatchdog@`hostname -f`" -s "wgserver %{STATE} via %{IFC}" admin@`cat /etc/mailname`' \
--slow-up-timeout=3 \
--ifup-grace=20 \
--install-systemd=wgserver"
```
Then we setup the monitor for the wireguard full tunnel which will also create and configure and bring the tunnel up (if not already) upon start of the sytemd service, making sure we have a /32 routes to all of the wireguard server's ip addresses that the hostname resolves to.<br>
Note: It would be smoother to use a script and `--ifcup=/path/script`, but it can also be done like this:

```shell
nw-watchdog 10.0.0.1 \
--no-ping-nexthop \
--verbosity-level=3 \
--alert='mailx -a "From: nwwatchdog@`hostname -f`" -s "wireguard wg0" admin@`cat /etc/mailname`' \
--force-interface=wg0 \
--ifcup='ip link add wg0 type wireguard ;
         ip link set wg0 up ;
         wg setconf wg0 /etc/wireguard/wg0.conf ;
         ip address add 10.0.0.2 peer 10.0.0.1 dev wg0 ;
         getent ahostsv4 wgserver.my.dom | grep -oE "^[0-9.]+" | uniq
         | while read addr; do
             ip route add $addr/32 via `ip route show default | head -1 | cut -d" " -f3` ;
           done ;
         ip route add 0.0.0.0/1 via 10.0.0.1 dev wg0 src 10.0.0.2 ;
         ip route add 128.0.0.0/1 via 10.0.0.1 dev wg0 src 10.0.0.2' \
--ifcdown='ip link del wg0' \
--install-systemd=wg0
```

__OBSERVE__ that the entire `--ifcup='...'` command need to be on a single line without line breaks (linebreaks added above for readability):
```
--ifcup='ip link add wg0 type wireguard ; ip link set wg0 up ; wg setconf wg0 /etc/wireguard/wg0.conf ; ip address add 10.0.0.2 peer 10.0.0.1 dev wg0 ; getent ahostsv4 wgserver.my.dom | grep -oE "^[0-9.]+" | uniq | while read addr; do ip route add $addr/32 via `ip route show default | head -1 | cut -d" " -f3` ; done ; ip route add 0.0.0.0/1 via 10.0.0.1 dev wg0 src 10.0.0.2 ; ip route add 128.0.0.0/1 via 10.0.0.1 dev wg0 src 10.0.0.2'
```

We use `--no-ping-nexthop` as the NEXTHOP is the same as the <ins>TARGET</ins> peer-to-peer address we monitor the connection for (in reality we don't need to specify it as it is the default behaviour if the <ins>TARGET</ins> address is the same as the NEXTHOP address).
    
	
#### <ins>Several VPN paths with one preferred interface:

This is a (real life) cornor case but worth explaining to get an understanding of the capabilities of __nw-watchdog__.

We have setup ifupdown to handle three different vpn-interfaces: `vpnL` `vpnP` and `vpnF`<br>
They all use the same VPN server but have different routes via the server.<br>
Only one of the interfaces can be up at any given time.

If `vpnL` is up we route just to the peer local network `10.0.10.0/24` via it.<br>
If `vpnP` is up we route to all private addresses via it, including `10.0.0.0/8`<br>
If `vpnF` is up we use it as a full tunnel, routing all traffic (apart from the VPN-connection itself) via it.

Our preferred interface is `vpnL` and if we can't get any traffic through the vpn-server, `vpnL` is the interface we want to reset.<br>
BUT as long as we can get traffic through using any of the three interfaces, we don't want to get alerted and have any interface reset, so we can't use `--force-interface`. The soloution will be to allow continuous topology detection but hardcode the preferred interface in `--ifcup` and `--ifcdown`.

```shell
nw-watchdog vpn.inside.dom \
--interface=vpnL \
--no-ping-nexthop \
--slow-up-timeout=7 \
--ifup-grace=35 \
--ifcdown='ifdown vnpL ; ifdown vpnP  ; ifdown vpnF' \
--ifcup='ifup vpnL'
```
Starting the watchdog like the above with any of the three vpn-interfaces up, the vpn-interface that is up will be detected and used as source interface. The continuous topology detection will ensure switching to which ever interface currently is up. If all interfaces are down the topology detection will think the interface for the default gw is the one to monitor (in our example `eth0`) but since we have `--no-ping-nexthop` and hardcoded the preferred vpn-interface in `--ifcup` the watchdog will bring up `vpnL`. If any of the vpn interfaces are up, but have problems the watchdog will bring them all down and then bring up `vpnL`.

We add `--verbosity-level=5` to the above, allowing us to get a trace of what is happening in the logfile in the following scenario:<br>
`vpnL` is up when __nw-watchdog__ starts,<br>
after a while we see that `vpnF` is replacing `vpnL` (as somebody brought up the full tunnel for a while)<br>
then `vpnF` is brought down (as somebody did not need it anymore) and<br>
__nw-watchdog__ detetcs that the connectivity via the VPN-server is lost and brings up `vpnL`:
```
00:00:01   INFO: Target (vpn.inside.dom) resolved to 10.0.10.1 (instead of '').
00:00:01   INFO: Detected topology: IFC='vpnL' -> 'vpnL' ; NEXTHOP='' -> '10.0.10.1'
```
^^^ This shows that `vpnL` is the interface that is up at start
```
00:00:01  TRACE: Continuously checking if target is up...
00:00:01  TRACE: ... and for changes in topology
00:00:26   INFO: Detected topology: IFC='vpnL' -> 'vpnF' ; NEXTHOP='10.0.10.1' -> '10.0.10.1'
```
^^^ Here somebody replaced `vpnL` with the full tunnel `vpnF`
```
00:00:31   INFO: Detected topology: IFC='vpnF' -> 'eth0' ; NEXTHOP='10.0.10.1' -> '192.168.0.1'
```
^^^ and here the full tunnel was brought down again (without replacing it with any of the other vpn-interfaces) 
```
00:00:32  TRACE: quick-up failed ... trying slow-up ...
00:00:32  TRACE: slow-up failed or ambigious result ... verifying ...
00:00:34  TRACE: No reply from target 'vpn.inside.dom' (10.0.10.1), checking link and topology.
00:00:35  TRACE: quick-up failed ... trying slow-up ...
00:00:35  TRACE: slow-up failed or ambigious result ... verifying ...
```
^^^ Here the __nw-watchdog__ gives up and concludes that the <ins>TARGET</ins> is down. 
```
00:00:37  ALERT: DOWN - Not getting replies from target 'vpn.inside.dom' (10.0.10.1) on interface 'eth0'.
                 Resetting interface:
                 ifdown vpnL ; ifdown vpnP ; ifdown vpnF
                 sleep 1
                 ifup vpnL
00:00:37   INFO: Resetting interface (eth0).
```
^^^ This can be confusing since it's actually not eth0 that is reset (but it's the current source interface)<br>
Below we see that the watchdog actually brings down all the vpn-interfaces and brings up `vpnL` 
```
00:00:37  TRACE: sh -c 'ifdown vpnL ; ifdown vpnP ; ifdown vpnF'
00:00:38  TRACE: sh -c 'ifup vpnL'
00:00:38  TRACE: Sleeping for 35 seconds.
```
^^^ This is the grace period we have set in `--ifup-grace`
```
00:01:13   INFO: Detected topology: IFC='eth0' -> 'vpnL' ; NEXTHOP='192.168.0.1' -> '10.0.10.1'
00:01:13  TRACE: Continuously checking if target is up...
00:01:13  TRACE: ... and for changes in topology
00:01:13  ALERT: UP - Target 'vpn.inside.dom' (10.0.10.1) is up.
```

## DEPENDENCIES
__nw-watchdog__ depends on the below executables being available in `/sbin:/bin:/usr/sbin:/usr/bin:$PATH` or being shell-builtin. A check is done at startup and if any of these tools are missing, __nw-watchdog__ will exit with an error telling which are lacking.
	
- `sh`
- `printf`
- `which`
- `basename`
- `cat`
- `cut`
- `date`
- `getent`
- `grep`
- `head`
- `id`
- `ip`
- `ping`
- `readlink`
- `sed`
- `sleep`
- `stat`
- `tail`
- `touch`
- `wc`

__nw-watchdog__ will function without the below listed utilities, but will use them to enhance its functionality if available.

- `systemd` (`mkdir`, `chmod`, `rm`)<br>
  `systemd` is (obviously) required for the `--install-systemd`, `--list-systemd` and `--remove-systemd` options to be functional. The folder `/etc/systemd/system/` must exist, `systemctl` must be in the PATH and `systemd` must be running.<br>
  Unless all required directories are already existing, `mkdir` and `chmod` are also used by `--install-systemd`.<br>
  `rm` is used by `--remove-systemd` to remove the systemd unit file for the service.
  
- `flock`<br>
  If available the logfile will be locked before truncated or written to.<br>
  If not available, there is a slight risk of log entries being lost if two or more instances of __nw-watchdog__ are concurently running using the same logfile and at least one of them have `--logsize` set to a value larger than 0.

- `fmt`<br>
  Is used to format the help message if available.

- `less` (or `more`)<br>
  Is used to page this help unless $PAGER is set to something else.

- `tput`<br>
  Is used to determine the terminal width and output bold and underlined text in this help page.

- `wall`<br>
  If available it will be used as default `--alert` command. Otherwise, alerting will be done to stderr by default.

