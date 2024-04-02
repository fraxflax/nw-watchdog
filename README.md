# nw-watchdog

__nw-watchdog__ is a higly configurable network watchdog written in posix shell script for use in Linux, depending only on Linux most standard tools that are normally installed by default in all distributions (also see the __DEPENDENCIES__ section).

It monitors the network connectivity to a specified target and/or the next hop towards that target, alerting upon lost connectivity explaining what is wrong. It can handle resetting the source interface and will detect topology changes and, if allowed, reconfigure itself accordingly. It's intended to run as a daemon and has an option to install itself as a systemd service.  If you want to monitor the connectivity to several targets, you can run several instances of nw-watchdog using different `--pidfile` option arguments.

__nw-watchdog__ is free software written by Fredrik Ax \<frax@axnet.nu\>.<br>
Feel free to modify and/or (re)distribute it in any way you like.<br>
... it's always nice to be mentioned though ;-)<br>
It comes with ABSOLUTELY NO WARRANTY.

## INSTALL

Just download the script [nw-watchdog](https://raw.githubusercontent.com/fraxflax/nw-watchdog/main/nw-watchdog) into your PATH and make it executable, e.g. like this:
```
curl -o /usr/local/bin/nw-watchdog https://raw.githubusercontent.com/fraxflax/nw-watchdog/main/nw-watchdog ; chmod a+rx /usr/local/bin/nw-watchdog
```
## DOCUMENTATION
Extensive documentation including examples, also shown below, is available by running:<br>
`nw-watchdog --help`<br>

---

## SYNOPSIS
__nw-watchdog__ <ins>TARGET</ins> [ OPTIONS ] 

## TARGET
The mandatory argument <ins>TARGET</ins> is the target (destination) to monitor the connection to. <ins>TARGET</ins> can be an IP address or a resolvable hostname / FQDN.

If it's a hostname / FQDN, it will be resolved to an IP address (first one found) The resolved IP address will be used for the monitoring. The name is continously resolved and if the resolved ip address changes the new IP address will be used for the monitoring from there on. Use `--no-continuous-topology-detect` to resolve the target only at startup and failed connectivity checks.

## OPTIONS (no arguments)
These options take no arguments, and may be specified in any order. They can be grouped (e.g. -vAP) in their short form, also having one of the OPTIONS that takes arguments last.

* __--help | -h__

	Shows this help, using $PAGER if set to an executable, otherwise 'less' or 'more' if available in "/sbin:/bin:/usr/sbin:/usr/bin:$PATH"<br>
(Use PAGER=cat to avoid using a pager).

* __--no-ping-target | -P__

	If the target is the nexthop (on the same subnet or a peer-to-peer address), reachability of the target is checked by arp cache status and ping.<br>
If the target is not on the same subnet as the source, the reachability of the target is checked by pinging it in a certain pattern (see `--slow-up-timeout` for details).

	`--no-ping-target` disables the ping-checks for the target. Only connectivity to the nexthop for the target is checked.<br>
	It can be useful if target does not reply to ping, or if it desirable to only alert if there is no route to the target or nexthop is unreachable.

	`--no-ping-target` cannot be used in combination with `--no-ping-nexthop`.

* __--no-ping-nexthop | -N | --no-ping-gateway | -G__<br>
	By default, if the connectivity to the target cannot be verified, the reachability of the nexthop (usually a gateway) is checked, firstly by checking it's status in the arp cache and then by pinging it, rechecking the arp cache status upon failed ping. 

	`--no-ping-nexthop` disbles the reachaility check for the nexthop so only connectivity to target itself is checked. It can be useful if the nexthop is a peer-to-peer address and not setup to reply to ping.

	`--no-ping-nexthop` cannot be used in combination with `--no-ping-target`.

* __--no-ipaddr-alert | -A__<br>
	Do not alert for not finding any global scope ip addresses on the source interface.

* __--no-interface-reset | -R__<br>
  Do not try to bring down and up interface after failed connectivity checks.<br>
  (Do not try to "repair" the connection", just monitor it.)

* __--no-continuous-topology-detect | -T__<br>
  Normaly the topology (resolving the ip address of the target, detecting which source interface to use and the ip address of the nexthop towards the target) is detected at startup and continuously monitored for changes.

	`--no-continuous-topology-detect` disables the topology detection for as long as the target replies (or in combination with `--no-ping-target` for as long as the nexthop is reachable). The topology will only be detected at startup and if the TARGET does not reply or if the NEXTHOP cannot be reached, meaning that routing changes making the TARGET or NEXTHOP unreachable will not be detected as long as the TARGET can be reached using the old topology.

	`--force-interface` implies `--no-continuous-topology-detect`.
    				       
* __--foreground | -f | --no-daemonize | -D__<br>
	Run in foreground, do not fork / daemonize.

* __--verbose | -v__<br>
	Shortcut for `--verbosity-level=5`<br>
	If used in combination with `--verbosity-level`, the specified verbosity level will take precedence. 

* __--debug | -d__<br>
	Shortcut for: `--verbosity-level=6  --logfile=- --logsize=0  --pidfile=/dev/null  --slow-up-timeout=1  --sleep=3 --ifup-grace=5 --alert='cat' --foreground` 

	If it's combined with any of the options it provides shortcuts for, the specified option will take precedence over the `--debug` shortcut.

	This option cannot be combined with `--install-systemd` (but it would be wise to test the configuration with `--debug` before installing as a systemd service).

## OPTIONS (with ARGUMENT)

* __--verbosity-level | -V__ <ins>level</ins><br>
	Default: `4`<br>
	<ins>level</ins> must be an integer greater than or equal to zero.
	Determines how much info is logged to the logfile and which alerts are triggered.

	__0__ - none<br>
	No output to logfile and no alerts trigged at all ... pretty useless unless you just want to keep traffic going keeping a connecting alive without any alerts.

	__1__ - error<br>
	Only logs and alerts on errors causing the nw-watchdog not to function as intended.

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

	If neither of `--interface` or `--force-interface` is specified the source interface it will be determined from the FIB by looking at the route to the target. The reason to specify it even so, would be to have nw-watchdog bring it up if it's down when starting.

	`--interface` cannot be combined with `--force-interface`.

* __--force-interface | -I__ <ins>interface</ins><br>
  Default: none<br>
  <ins>interface</ins> is the name of the source interface to always use.

	Packets will always be sent from this interface. The forwadring table will be ignored as well as conflicting topology changes.<br>
	This is useful for monitoring the preferred path and making sure it's up. It does not check if you have connectivity to the target via any other path.

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

  If `flock` is available, the logfile will be locked before written to or shrinked, otherwise there is a slight risk of loglines being lost if two or more instances of __nw-watchdog__ are concurently running using the same logfile and at least one of them have `--logsize` set to a value larger than 0.

* __--pidfile | -p__ <ins>pidfile</ins><br>
  Default: `/run/nw-watchdog.pid`<br>
  Pidfile to use.

* __--slow-up-timeout | -t__ <ins>seconds</ins><br>
  Default: `3`<br>
  <ins>seconds</ins> must be an integer greater than zero.

  The check whether the target is up or not, is performed in several steps.<br>
  First a "quick-up" test sends one single ICMP echo packet waiting for the reply for no more than 1 second. If that fails, a more thourough "slow-up" test sends 5 ICMP echos.

  `--slow-up-timeout` controls he TIMEOUT for waiting on each packet in the slow-up test.<br>
  5 packets are always sent in the slow-up test.<br>
  The packets are sent adaptively, meaning that as soon as a reply is received the next packet is sent without delay, giving slow-up a total time of 5 * RTT to the target if the connection is up.<br>
  The DEADLINE = TIMEOUT * 5 is the maximum time the slow-up test will take if the target is down.
        
  `--slow-up-timeout=7` is useful for monitoring VPN connections via interfaces that need a long wake-up time if idle (due to regotiation of encryption, exchanging keys, reauthentication, etc).

  `--slow-up-timeout=3` is useful for monitoring connections to targets with low - medium latency via interfaces that does not need a long wake-up time (e.g. ethernet interfaces).

  `--slow-up-timeout=1` is suitable to use for monitoring local targets (e.g. nexthop) on ethernet carried subnets.

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

  - ifupdown, non privilege user running nw-watchdog:<br>
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

  - ifupdown, non privilege user running nw-watchdog:<br>
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
	- %{TARGET} - the target for which the connection is monitored
	- %{TADDR} - the IP address of the TARGET
    - %{NEXTHOP}  - the IP address of the NEXTHOP towards target
	- %{STATE} - the state of the alert:
	  - UP for restored connectivity to TARGET (implies REACHABLE)
	  - DOWN for lost conenctivity to TARGET
	  - UNREACHABLE for lost conenctivity to NEXTHOP (implies DOWN)
	  - REACHABLE for restored connectivity to NEXTHOP
	  - ERROR for permanent errors
	  - WARNING for things that might need reconfiguration

	The alert command will be launched for every ERROR and WARNING, even repeated ones.<br>
	For the other states (UP, DOWN, UNREACHABLE, REACHABLE) the alert command will be launched only upon state change.<br>
	E.g. if the state goes from UP to DOWN the DOWN alert is triggered, but if the next check is also DOWN, no further alert will be sent (until the state changes again).

	EXAMPLE of how to email the alert messages using mailx (mailutils) with a custom from address:<br>
	`--alert='mailx -a "From: nw-watchdog@this.hst" -s "nw-watchdog %{STATE} alert for %{TARGET} via %{IFC}" my@email.adr'`

* __--install-systemd__ <ins>SERVICENAME</ins><br>
  Default: none
	
  Will write a systemd service file `/etc/systemd/system/nw-watchdog-SERVICENAME.service` launching nw-watchdog as a daemon with<br>
  `--pidfile=/run/nw-watchdog-SERVICENAME.pid`<br>
  `--lofile=/var/log/nw-watchdog-SERVICENAME.log`<br>
	and otherwise with the exact same options as run (apart from the `--install-systemd` option itself of course).

	If `/etc/systemd/system/nw-watchdog-SERVICENAME.service` already exists, the service will be stopped and the file overwritten.

	It will then enable, start it and show status of the newly created nw-watchdog-<ins>SERVICENAME</ins>.service.

	The <ins>SERVICENAME</ins> must consist of at least 1 valid character ( 'a-z', 'A-Z', '0-9', '-' and '_' ) and be no longer than 236 characters.

    This option requires root privileges.

	Note:<br>
	To completely remove the service do (as root):
    ```shell
	systemctl stop nw-watchdog-SERVICENAME.service
	systemctl disable nw-watchdog-SERVICENAME.service
	rm /etc/systemd/system/nw-watchdog-SERVICENAME.service
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
Firstly, we start a nw-watchdog monitoring the connectivity to the IPSec peers public address (1.2.3.4 in this example), emailing alerts to the admin.

```shell
nw-watchdog 1.2.3.4 \
--verbosity-level=3 \
--alert='mailx -a "From: nwwatchdog@`hostname -f`" -s "IPSec Peer 1.2.3.4 %{STATE} via %{IFC}" admin@`cat /etc/mailname\`' \
--slow-up-timeout=2 \
--ifup-grace=10 \
--interval=10
```

Then we setup the monitor for the IPSec VTI tunnel which will also be created, configured and brought up.

Note: It would be smoother to use a script and `--ifcup=/path/script`, but it can also be done like this:

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
--ifcdown='ipsec down connection-name ; ip tunnel del %{IFC}' 
```

__OBSERVE__ that the entire `--ifcup=` command needs to be on a single line without line breaks (linebreaks added above for readability):
```
--ifcup='ipsec up connection-name ; ip tunnel add %{IFC} mode vti local 4.3.2.1 remote 1.2.3.4 ttl 255 key 111 ; ip addr add dev %{IFC} 169.254.0.2 remote 169.254.0.1 ; ip link set %{IFC} up multicast on mtu 1424 state UP ; ip route add 10.10.0.0/20 via 169.254.0.1 dev %{IFC}'
```


We use `--ifup-grace=25` to allow enough time for IPSec to establish the connection upon start and interface reset.
`--slow-up-timeout=5` (making the deadline 25 seconds for the slow-up ping test) should be enough for an idle IPSec conmnection to wakeup and start forwarding packets when monitored.

In the above example we monitor the connectivity to the peer address inside the VTI tunnel, which is also the nexthop.
If we are interested in the connectivity to something in the remote network routed via the tunnel we could use that as a target instead of the peer address:

`nw-watchdog 10.10.1.1 ...`<br>
and the rest of the options exactly the same as above.

This would give alerts if `10.10.1.1` is down.<br>
In all other ways the effect would be the same as using `169.254.0.1` as target.<br>
Even with target `10.10.1.1`, the nexthop (`169.254.0.1`) will also be monitored (we are __NOT__ using --no-ping-nexthop) and the interface will __NOT__ be reset as long as the nexthop is reachable.


#### <ins>Wireguard full tunnel management:</ins>
This is an example of how one can use nw-watchdog to setup and monitor a wireguard full tunnel, also monitoring the connectivity to the wireguard server, running both whatchdogs as systemd services getting alerts via e-mail:

Firstly, we setup the nw-watchdog systemd service for the wireguard server which we reach via the default route:

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
         ip route add 128.0.0.0/1 via 10.0.0.1 dev wg0 src 10.0.0.2 2>/dev/null' \
--ifcdown='ip link del wg0' \
--install-systemd=wg0
```

__OBSERVE__ that the entire `--ifcup='...'` command need to be on a single line without line breaks (linebreaks added above for readability):
```
--ifcup='ip link add wg0 type wireguard ; ip link set wg0 up ; wg setconf wg0 /etc/wireguard/wg0.conf ; ip address add 10.0.0.2 peer 10.0.0.1 dev wg0 ; getent ahostsv4 wgserver.my.dom | grep -oE "^[0-9.]+" | uniq | while read addr; do ip route add $addr/32 via `ip route show default | head -1 | cut -d" " -f3` ; done ; ip route add 0.0.0.0/1 via 10.0.0.1 dev wg0 src 10.0.0.2 ; ip route add 128.0.0.0/1 via 10.0.0.1 dev wg0 src 10.0.0.2 2>/dev/null'
```

We use `--no-ping-nexthop` as the nexthop is the same as the target peer-to-peer address we monitor the connection for (in reality we don't need to specify it as it is the default behaviour if the target address is the same as the nexthop address).
    

#### <ins>Several VPN paths with one preferred interface:

This is a (real life) cornor case but worth explaining to get an understanding of the capabilities of nw-watchdog.

We have setup ifupdown to handle three different vpn-interfaces: vpnL vpnP and vpnF<br>
They all use the same VPN server but have different routes to the server.<br>
Only one of the interfaces can be up at any given time.

If `vpnL` is up we route just to the peer local network `10.0.10.0/24` via it.<br>
If `vpnP` is up we route to all private addresses via it, including `10.0.0.0/8`<br>
If `vpnF` is up we use it ass a full tunnel, routing all traffic (apart from the VPN-connection itself) via it.

Our preferred interface is vpnL and if we can't get any traffic through the vpnserver that is the one we want to reset.
BUT as long as can get traffic through using any of the three interfaces we don't want to get alerted and have any interface reset,
so we can't use `--force-interface`. The soloution will be to allow continuous topology detection but hardcode the preferred interface in `--ifcup` and `--ifcdown`.

```shell
nw-watchdog vpn.inside.dom \
--interface=vpnL \
--no-ping-nexthop \
--slow-up-timeout=7 \
--ifup-grace=35 \
--ifcdown='ifdown vnpL ; ifdown vpnP  ; ifdown vpnF' \
--ifcup='ifup vpnL'
```
Starting the watchdog like the above with any of the three vpn-interfaces up, the vpn-interface that is up will be detected and used as source interface. The continuous topology detection will ensure switching to which ever interface currently is up. If all interfaces are down the topology detection will think the interface for the default gw is the one to monitor (in our example `eth0`) but since we have `--no-ping-nexthop` and hardcoded the preferred vpn-interfaces in `--ifcup` the watchdog will bring up vpnL. If any of the vpn interfaces are up, but have problems the watchdog will bring them all down and then bring up `vpnL`.

If we add --verbosity-level=5 to the above, allowing us to get a trace of what is happening in the logfile in the scenario of none of the vpn interfaces being up at start, after a while we see that vpnF is replacing vpnL (as somebody brought up the full tunnel for a while) then vpnF is brought down (as somebody did not need it anymore) and nw-watchdog detetcs that the connectivity via the VPN-server is lost and brings up `vpnL`:
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
^^^ Here sombody replaced `vpnL` with the full tunnel `vpnF`
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
^^^ Here the nw-watchdog gives up and judge that the target is down. 
```
00:00:37  ALERT: DOWN - Not getting replies from target 'vpn.inside.dom' (10.0.10.1) on interface 'eth0'.
                 Resetting interface:
                 ifdown vpnL ; ifdown vpnP ; ifdown vgfull
                 sleep 1
                 ifup vpnL
00:00:37   INFO: Resetting interface (eth0).
```
^^^ This can be confusing since it's actually not eth0 that is reset (it's just the current source interface) 
Below we see that the watchdog actually brings down all the vpn-interfaces and brings up `vpnL` 
```
00:00:37  TRACE: sh -c 'ifdown vpnL ; ifdown vpnP ; ifdown vgfull'ifup vpnL    TRACE: sh -c ''
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
nw-watchdog depends on the below executables being available in `/sbin:/bin:/usr/sbin:/usr/bin:$PATH` or being shell-builtin. A check is done at startup and if any of these tools are missing, nw-watchdog will exit with an error telling which are lacking.
	
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
- `printf`
- `readlink`
- `sed`
- `sleep`
- `stat`
- `tail`
- `touch`
- `wc`

__nw-watchdog__ will function without the below listed utilities, but will use them to enhance its functionality if available.

- `flock`<br>
  If available the logfile will be locked before truncated or written to.<br>
  If not available, there is a slight risk of loglines being lost if two or more instances of __nw-watchdog__ are concurently running using the same logfile and at least one of them have `--logsize` set to a value larger than 0.

- `wall`<br>
  If available it will be used as default `--alert` command. Otherwise, alerting will be done to stderr by default.

- `tput`<br>
  Is used to determine the terminal width and output bold and underlined text in this help page.

- fmt`<br>
  Is used to format the help message if available.

- `less` (or `more`)<br>
  Is used to page this help unless $PAGER is set to something else.

