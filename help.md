---
__NAME__<br>
        nw-watchdog - Network Watchdog

---
__SYNOPSIS__<br>
__nw-watchdog__ [ OPTIONS ] <u>TARGET</u>

---
__DESCRIPTION__<br>
__nw-watchdog__ is a higly configurable network watchdog written in posix shell script for use in Linux, depending only on Linux most standard tools that are normally installed by default in all distributions (also see the __DEPENDENCIES__ section).

It monitors the network connectivity to a specified target and/or the next hop towards that target, alerting upon lost connectivity explaining what is wrong. It can handle resetting the source interface and will detect topology changes and, if allowed, reconfigure itself accordingly. It's intended to run as a daemon and has an option to install itself as a systemd service.  If you want to monitor the connectivity to several targets, you can run several instances of nw-watchdog using different __--pidfile__ option arguments.

__nw-watchdog__ is free software written by Fredrik Ax \<frax@axnet.nu\>.<br>
Feel free to modify and/or (re)distribute it in any way you like.<br>
... it's always nice to be mentioned though ;-)<br>
It comes with ABSOLUTELY NO WARRANTY.

Get the latest version from https://github.com/fraxflax/nw-watchdog

---
__TARGET__<br>
The mandatory argument <u>TARGET</u> is the target (destination) to monitor the connection to. <u>TARGET</u> can be an IP address or a resolvable hostname / FQDN. If it's a hostname / FQDN, it will be resolved to an IP address (first one found) at startup and the resolved IP address will be used for the monitoring. Upon failed ping-checks the name will be resolved again and if it resolves to a new IP address, that will be used for the monitoring from there on.

---
__OPTIONS__ (no arguments)<br>
These options take no arguments, and may be specified in any order. They can be grouped (e.g. -vAP) in their short form, also having one of the OPTIONS that takes arguments last.

- __--help | -h__<br>
Shows this help, using \$PAGER if set to an executable, otherwise 'less' or 'more' if available in "/sbin:/bin:/usr/sbin:/usr/bin:\$PATH"<br>
(Use PAGER=cat to avoid using a pager).

- __--no-ping-target | -P__<br>
If the target is the nexthop (on the same subnet or a peer-to-peer address), reachability of the target is checked by arp cache status and ping.<br>
If the target is not on the same subnet as the source, the reachability of the target is checked by pinging it in a certain pattern (see __--slow-up-timeout__ for details).

`--no-ping-target` disables the ping-checks for the target. Only connectivity to the nexthop for the target is checked.<br>
It can be useful if target does not reply to ping, or if it desirable to only alert if there is no route to the target or nexthop is unreachable.

`--no-ping-target` cannot be used in combination with `--no-ping-nexthop`.

- __--no-ping-nexthop | -N | --no-ping-gateway | -G__<br>
  By default, if the connectivity to the target cannot be verified, the reachability of the nexthop (usually a gateway) is checked, firstly by checking it's status in the arp cache and then by pinging it, rechecking the arp cache status upon failed ping. 

`--no-ping-nexthop` disbles the reachaility check for the nexthop so only connectivity to target itself is checked. It can be useful if the nexthop is a peer-to-peer address and not setup to reply to ping.

`--no-ping-nexthop` cannot be used in combination with `--no-ping-target`.

- __--no-ipaddr-alert | -A__<br>
  Do not alert for not finding any global scope ip addresses on the source interface.

- __--no-interface-reset | -R__<br>
  Do not try to bring down and up interface after failed connectivity checks.<br>
  (Do not try to "repair" the connection", just monitor it.)

- __--no-continuous-topology-detect | -T__<br>
  Normaly the topology (resolving the ip address of the target, detecting which source interface to use and the ip address of the nexthop towards the target) is detected at startup and continuously monitored for changes.

`--no-continuous-topology-detect` disables the topology detection for as long as the target replies (or in combination with `--no-ping-target` for as long as the nexthop is reachable). The topology will only be detected at startup and if the TARGET does not reply or if the NEXTHOP cannot be reached, meaning that routing changes making the TARGET or NEXTHOP unreachable will not be detected as long as the TARGET can be reached using the old topology.

`--force-interface` implies `--no-continuous-topology-detect`.
    				       
- __--foreground | -f | --no-daemonize | -D__<br>
  Do not fork / daemonize, run in foreground.

- __--verbose | -v__<br>
  Shortcut for `--verbosity-level=5`<br>
  If used in combination with `--verbosity-level`, the specified verbosity level will take precedence. 

- __--debug | -d__<br>
  Shortcut for: `--verbosity-level=6  --logfile=- --logsize=0  --pidfile=/dev/null  --slow-up-timeout=1  --sleep=3 --ifup-grace=5 --alert='cat' --foreground` 

  If it's combined with any of the options it provides shortcuts for, the specified option will take precedence over the `--debug` shortcut.

  This option cannot be combined with `--install-systemd` (but it would be wise to test the configuration with `--debug` before installing as a systemd service).

---
__OPTIONS__ (with ARGUMENT)<br>
        These opions takes a single argument each and may be specified in any order. Specify with equalsign or space or no space between option and argument. They can only be grouped together with the shortform of the NO-ARGUMENT-OPTIONS above, and must be last in such groupings (e.g. -PAV5).

    __--verbosity-level | -V__ <u>level</u>
        Default: 4
	<u>level</u> must be an integer greater than or equal to zero.
        Determines how much info is logged to the logfile and which alerts are triggered.

        __0__ - none
        No output to logfile and no alerts trigged at all ... pretty useless unless you just want to keep traffic going keeping a connecting alive without any alerts.

        __1__ - error
        Only logs and alerts on errors causing the nw-watchdog not to function as intended.

        __2__ - warning
        Also logs and alerts on warnings about configuration, etc.

        __3__ - alert
        Log and alert on irreparable connectivity failures, interface down, and other things disrupting the monitored connection, as well as on warnings and errors.
        Also see the __--alert__ option.

        __4__ - info (default)
        Same as level 3, but also logs some useful information on what's going on, such as topology changes, some test failures forcing more testing,  interface resets, etc

        __5__ - trace
        Logs even more info about which action is currently performed, including, all secondary tests that are run, when sleeping longer than usual, etc.

        __6__ - debug
        Logs even more internal details, e.g. SETTINGS used, all tests performed, all sleeps, and other debug info.

    __--interface | -i__ <u>interface</u>
    	Default: none
        <u>interface</u> is the name of the source interface to initially use.

        The interface may dynamically change due to topology detection. If you want to force the use of a specific interface, use __--force-interface__ instead.

        If neither of __--interface__ or __--force-interface__ is specified the source interface it will be determined from the FIB by looking at the route to the target. The reason to specify it even so, would be to have nw-watchdog bring it up if it's down when starting.

        __--interface__ cannot be combined with __--force-interface__.

    __--force-interface | -I__ <u>interface</u>
    	Default: none
        <u>interface</u> is the name of the source interface to always use.

        Packets will always be sent from this interface. The forwadring table will be ignored as well as conflicting topology changes.
        This is useful for monitoring the preferred path and making sure it's up. It does not check if you have connectivity to the target via any other path.

	Implies __--no-continuous-topology-detect__.

	__--force-interface__ cannot be combined with __--interface__.

    __--logfile | -l__ <u>logfile</u>
        Default: '/var/log/nw-watchdog.log'
        Logfile to use. If specified as '-' logs are written to stdout.

    __--logsize | -z__ <u>size</u>
        Default: 0

	If the logfile grows beyond this size, the oldest entries will be removed.

	You can use suffixes K, M, G for kilo / mega / giga bytes. (No suffix is same as K).
	Set to 0 for unlimited logfile size (which you would want if you do log rotation). 

	If the logfile is set to '-' (stdout) this option is ignored.

	If __flock__ is available, the logfile will be locked before written to or shrinked, otherwise there is a slight risk of loglines being lost if two or more instances of __nw-watchdog__ are concurently running using the same logfile and at least one of them have __--logsize__ set to a value larger than 0.

    __--pidfile | -p__ <u>pidfile</u>
        Default: '/run/nw-watchdog.pid'
        Pidfile to use.

    __--slow-up-timeout | -t__ <u>seconds</u>
        Default: 7
        <u>seconds</u> must be an integer greater than zero.

        The check whether the target is up or not, is performed in several steps.
	First a "quick-up" test sends one single ICMP echo packet waiting for the reply for no more than 1 second. If that fails, a more thourough "slow-up" test sends 5 ICMP echos.

	__--slow-up-timeout__ controls he TIMEOUT for waiting on each packet in the slow-up test.
        5 packets are always sent in the slow-up test.
        The packets are sent adaptively, meaning that as soon as a reply is received the next packet is sent without delay, giving slow-up a total time of 5 * RTT to the target if the connection is up.
        The DEADLINE = TIMEOUT * 5 is the maximum time the slow-up test will take if the target is down.
        
        __--slow-up-timeout=7__ is useful for monitoring VPN connections via interfaces that need a long wake-up time if idle (due to regotiation of encryption, exchanging keys, reauthentication, etc).

        __--slow-up-timeout=3__ is useful for monitoring connections to targets with low - medium latency via interfaces that does not need a long wake-up time (e.g. ethernet interfaces).

        __--slow-up-timeout=1__ is suitable to use for monitoring local targets (e.g. nexthop) on ethernet carried subnets.

    __--sleep | -s | --interval__ <u>seconds</u>
        Default: 10
        <u>seconds</u> must be an integer greater than zero.

        How many seconds to sleep after sucessful ping check. 

    __--ifup-grace | -g__ <u>seconds</u>
        Default: 40
        <u>seconds</u> must be an integer greater than zero.

        How many seconds to sleep before next check after interface has been reset.

    __--max-nolink | -n__ <u>number</u>
        Default: 1
        <u>number</u> must be an integer greater than or equal to zero.

        Maximum number of consecutive failed link checks in which the interface have been reset (brought down and up again) before doing new topology check.

        A word of warning: If set to 0 and interface is not up / goes down, infinite retries to bring the interface up will be made before checking topology. Only set it to 0 if you are sure that the specified interface should always be used and you want to make sure it's up before starting to monitor the connection.
        Typically, you would want to also use __--force-interface__ when using __--max-nolink=0__.

    __--ifcup | -u__ <u>STRING</u> 
        Default: 'ifup %{IFC}'
        <u>STRING</u> will be passed to 'sh -c' to bring the interface up.
	%{IFC} will be dynmaically replaced with the interface name currently in use as source interface.

	Examples:
	    ifupdown:
              --ifcup='ifup %{IFC}'

	    ifupdown, non privilege user running nw-watchdog:
              --ifcup='sudo ifup %{IFC}'

	    NetworkManager device:
              --ifcup='nmcli device up %{IFC}'

	    NetworkManager connection:
              --ifcup='nmcli connection up connection-name'

	    iproute2 + isc-dhcp-client:
              --ifcup='ip link set %{IFC} up && dhclient -pf /run/dhclient-%{IFC}.pid %{IFC}'

	    strongSwan IPSec (setup for IPSec policy routing): 
              --ifcup='ipsec up connection-name'
	      (see __EXAMPLES__  below for a more extensive IPSec example using vti tunnel interface)

    __--ifcdown | -U__ <u>STRING</u>
        Default: 'ifdown %{IFC}' 
        <u>STRING</u> will be passed to 'sh -c' to bring the interface down.
	%{IFC} will be dynmaically replaced with the interface name currently in use as source interface.

	Examples:
	    ifupdown:
              --ifcdown='ifdown %{IFC}'

	    ifupdown, non privilege user running nw-watchdog:
              --ifcdown='sudo ifdown %{IFC}'

	    NetworkManager device:
              --ifcdown='nmcli device down %{IFC}'

	    NetworkManager connection:
              --ifcdown='nmcli connection down %{IFC}-connection-name'

	    iproute2 + isc-dhcp-client:
              --ifcdown='kill \`cat /run/dhclient-%{IFC}.pid\` ; ip link set down %{IFC}'

	    strongSwan IPSec (setup for IPSec policy routing): 
              --ifcup='ipsec down connection-name'
	      (see __EXAMPLES__  below for a more extensive IPSec example using vti tunnel interface)

    __--alert | -a__ <u>STRING</u>
    	Default: 'if which wall >/dev/null; then exec wall; else cat 1>&2; fi'

	Errors, warnings and alerts regarding change of state will be piped to: sh -c '<u>STRING</u>'
	Within <u>STRING</u> the following placeholders can be used:		

	%{IFC} - the interface name

	%{TARGET} - the target for which the connection is monitored

	%{TADDR} - the IP address of the TARGET

	%{NEXTHOP}  - the IP address of the NEXTHOP towards target

	%{STATE} - the state of the alert:
	- UP for restored connectivity to TARGET (implies REACHABLE)
	- DOWN for lost conenctivity to TARGET
	- UNREACHABLE for lost conenctivity to NEXTHOP (implies DOWN)
	- REACHABLE for restored connectivity to NEXTHOP
	- ERROR for permanent errors
	- WARNING for things that might need reconfiguration

	The alert command will be launched for every ERROR and WARNING, even repeated ones.
	For the other states (UP, DOWN, UNREACHABLE, REACHABLE) the alert command will be launched only upon state change.
	E.g. if the state goes from UP to DOWN the alert is triggered but if the next check is also DOWN, no further alert will be sent (until the state changes again).

	EXAMPLE of how to email the alert messages using mailx (mailutils) with a custom from address:
	    --alert='mailx -a "From: nw-watchdog@this.hst" -s "nw-watchdog %{STATE} alert for %{TARGET} via %{IFC}" my@email.adr'

    __--install-systemd__ <u>SERVICENAME</u>
	Default: none
	
        Will write a systemd service file /etc/systemd/system/nw-watchdog-<u>SERVICENAME</u>.service file launching nw-watchdog as a daemon with
        __--pidfile__=/run/nw-watchdog-<u>SERVICENAME</u>.pid__
        __--lofile__=/var/log/nw-watchdog-<u>SERVICENAME</u>.log__
	and otherwise with the exact same options as run (apart from the __--install-systemd__ option itself of course).

	If /etc/systemd/system/nw-watchdog-<u>SERVICENAME</u>.service already exists, the service will be stopped and the file overwritten.

	It will then enable, start it and show status of the newly created nw-watchdog-<u>SERVICENAME</u>.service.

	The <u>SERVICENAME</u> must consist of at least 1 valid character ( 'a-z', 'A-Z', '0-9', '-' and '_' ) and be no longer than 236 characters.

        This option requires root privileges.

	Note:
	To completely remove the service do (as root):
	    systemctl stop nw-watchdog-<u>SERVICENAME</u>.service
	    systemctl disable nw-watchdog-<u>SERVICENAME</u>.service
	    rm /etc/systemd/system/nw-watchdog-<u>SERVICENAME</u>.service

__EXAMPLES__

	<u>ISP gateway monitoring:</u>

        __nw-watchdog__ 1.2.3.4 __\\
EOU
printf "        %s\n        %s\n        %s\n        %s\n        %s\n" \
       "  __--interval=__60 __\\" \
       "  __--no-ping-target __\\" \
       "  __--interface=__eth0 __\\" \
       "  __--slow-up-timeout=__1 __\\" \
       "  __--ifup-grace=__300"

$FMT<<EOU

	Checking once a minute (__--interval=__60) that we have connectivity to the Internet Service Provider's gateway without actually pinging anything on the Internet (__--no-ping-target__ 1.2.3.4 ... any Intenet address will do) allowing interface and topology detection (not using --force-interface or --max-no-link=0) but still, if down, bring the supposed initial interface towards the ISP up on startup (__--interface=__eth0) expecting the ISP gateway to have an RTT below 1 second (__--slow-up-timeout=__1) and allowing the interface to be down for up to 5 minutes before considering it a permanent error rechecking topology (__--ifup-grace=__300).


	<u>ISP gateway monitoring with forced interface:</u>

        __nw-watchdog__ 1.2.3.4 __\\
EOU
printf "        %s\n        %s\n        %s\n        %s\n        %s\n        %s\n" \
       "  __--no-ping-target \\" \
       "  __--force-interface=__eth0 __\\" \
       "  __--slow-up-timeout=__1 __\\" \
       "  __--ifup-grace=__30 __\\" \
       "  __--max-no-link=__0 __\\" \
       "  __--interval=__10__"
$FMT<<EOU

        Same as above but enforcing the use of the eth0 interface as we know that the ISP gateway should always be reachable via that interface and that is the only Internet facing interface we have. It must be brought up on startup if not already up and there is no use rechecking the topology if it's not up (__--force-interface=__eth0), so if down, we retry to reset it every 30 seconds (__--ifup-grace=__30) forever (__--max-no-link=__0), checking the connectivity every 10 seconds (__--interval=__10).


	<u>Management of Strongswan IPSec with VTI tunnel interface:</u>

	Firstly, we setup the nw-watchdog systemd service for monitoring the connectivity to the IPSec peers public address (1.2.3.4 in this example), emailing alerts to the admin.

        __nw-watchdog__ 1.2.3.4 __\\
EOU
printf "        %s\n        %s\n        %s\n        %s\n        %s\n" \
       "  __--verbosity-level=__3 __\\" \
       "  __--alert=__'mailx -a \"From: nwwatchdog@\`hostname -f\`\" -s \"IPSec Peer 1.2.3.4 %{STATE} via %{IFC}\" admin@\`cat /etc/mailname\`' __\\" \
       "  __--slow-up-timeout=__2 __\\" \
       "  __--ifup-grace=__10 __\\" \
       "  __--interval=__10__"
$FMT<<EOU

	Then we setup the monitor for the IPSec VTI tunnel which will also be created, configured and brought up.
	Note: It would be smoother to use a script and __--ifcup=__/path/script, but it can also be done like this:

	__nw-watchdog__ 169.254.0.1 __\\
EOU
printf "        %s\n        %s\n        %s\n        %s\n        %s\n        %s\n        %s\n        %s\n        %s\n        %s\n" \
       "  __--alert=__'mailx -a \"From: nwwatchdog@\`hostname -f\`\" -s \"IPSec connection-name %{STATE} via %{IFC}\" admin@\`cat /etc/mailname\`' __\\" \
       "  __--force-interface=__ipsec0 __\\" \
       "  __--slow-up-timeout=__5 __\\" \
       "  __--ifup-grace=__25 __\\" \
       "  __--ifcup=__'ipsec up connection-name ; " \
       "           ip tunnel add %{IFC} mode vti local 4.3.2.1 remote 1.2.3.4 ttl 255 key 111 ; " \
       "           ip addr add dev %{IFC} 169.254.0.2 remote 169.254.0.1; " \
       "           ip link set %{IFC} up multicast on mtu 1424 state UP;" \
       "           ip route add 10.10.0.0/20 via 169.254.0.1 dev %{IFC}'__ __\\" \
       "  __--ifcdown=__'ipsec down connection-name ; ip tunnel del %{IFC}' __\\" 
$FMT<<EOU

	_____OBSERVE__ that the entire __--ifcup=__'...' command need to be on a single line without line breaks (linebreaks added above for readability).

	We use __--ifup-grace=25__ to allow enough time for IPSec to establish the connection upon start and interface reset.
	__--slow-up-timeout=5__ (making the deadline 25 seconds for the slow-up ping test) should be enough for an idle IPSec conmnection to wakeup and start forwarding packets when monitored.

	In the above example we monitor the connectivity to the peer address inside the VTI tunnel, which is also the nexthop.
        If we are interested in the connectivity to something in the remote network routed via the tunnel we could use that as a target instead of the peer address:

	__nw-watchdog__ 10.10.1.1 __ ...
	and the rest of the options exactly the same.

	Now we would get alerts if 10.10.1.1 is down.
	In all other ways the effect would be the same as using 169.254.0.1 as target.
	Even with target 10.10.1.1, the nexthop (169.254.0.1) will also be monitored (we are NOT using --no-ping-nexthop) and the interface will NOT be reset as long as the nexthop is reachable.


	<u>Wireguard full tunnel management:</u>

	This is an example of how one can use nw-watchdog to setup and monitor a wireguard full tunnel, also monitoring the connectivity to the wireguard server, running both whatchdogs as systemd services getting alerts via e-mail:

	Firstly, we setup the nw-watchdog systemd service for the wireguard server which we reach via the default route:

	__nw-watchdog__ wgserver.domain.dom __\\
EOU
printf "        %s\n        %s\n        %s\n        %s\n        %s\n" \
       "  __--verbosity-level=__3 __\\" \
       "  __--alert=__'mailx -a \"From: nwwatchdog@\`hostname -f\`\" -s \"wgserver %{STATE} via %{IFC}\" admin@\`cat /etc/mailname\`' __\\" \
       "  __--slow-up-timeout=__3 __\\" \
       "  __--ifup-grace=__20 __\\" \
       "  __--install-systemd=__wgserver"
    $FMT<<EOU

	Then we setup the monitor for the wireguard full tunnel which will also create and configure and bring the tunnel up (if not already) upon start of the sytemd service, making sure we have a /32 routes to all of the wireguard server's ip addresses that the hostname resolves to.
	Note: It would be smoother to use a script and __--ifcup=__/path/script, but it can also be done like this:

	__nw-watchdog__ 10.0.0.1 __\\
EOU
printf "        %s\n        %s\n        %s\n        %s\n        %s\n        %s\n        %s\n        %s\n        %s\n        %s\n        %s\n        %s\n        %s\n        %s\n        %s\n        %s\n" \
       "  __--no-ping-nexthop \\" \
       "  __--verbosity-level=__3 __\\" \
       "  __--alert=__'mailx -a \"From: nwwatchdog@\`hostname -f\`\" -s \"wireguard wg0\" admin@\`cat /etc/mailname\`' __\\" \
       "  __--force-interface=__wg0 __\\" \
       "  __--ifcup=__'ip link add wg0 type wireguard ;" \
       "           ip link set wg0 up ;" \
       "           wg setconf wg0 /etc/wireguard/wg0.conf ;" \
       "           ip address add 10.0.0.2 peer 10.0.0.1 dev wg0 ;" \
       "           getent ahostsv4 wgserver.my.dom | grep -oE \"^[0-9.]+\" | uniq" \
       "           | while read addr; do" \
       "               ip route add \$addr/32 via \`ip route show default | head -1 | cut -d\" \" -f3\` ;" \
       "             done ;" \
       "           ip route add 0.0.0.0/1 via 10.0.0.1 dev wg0 src 10.0.0.2 ;" \
       "           ip route add 128.0.0.0/1 via 10.0.0.1 dev wg0 src 10.0.0.2 2>/dev/null'__ __\\" \
       "  __--ifcdown=__'ip link del wg0' __\\" \
       "  __--install-systemd=__wg0"

    $FMT<<EOU

   ___OBSERVE__ that the entire __--ifcup=__'...' command need to be on a single line without line breaks (linebreaks added above for readability).

    We use __--no-ping-nexthop__ as the nexthop is the same as the target peer-to-peer address we monitor the connection for (in reality we don't need to specify it as it is the default behaviour if the target address is the same as the nexthop address).
    

__DEPENDENCIES__
        nw-watchdog depends on the below executables being available in "/sbin:/bin:/usr/sbin:/usr/bin:\$PATH" or being shell-builtin. A check is done at startup and if any of these tools are missing, nw-watchdog will exit with an error telling which are lacking.
	
EOU
    for dep in $DEPENDENCIES; do
		printf "        - __%s__\n" "$dep"
    done
    $FMT<<EOU

        __nw-watchdog__ will function without the below listed utilities, but will use them to enhance its functionality if available.

	- __flock__
            If available the logfile will be locked before truncated or written to.
	    If not available, there is a slight risk of loglines being lost if two or more instances of __nw-watchdog__ are concurently running using the same logfile and at least one of them have __--logsize__ set to a value larger than 0.

	- __wall__
	If available it will be used as default __--alert__ command. Otherwise, alerting will be done to stderr by default.

	- __tput__
	Is used to determine the terminal width and output bold and underlined text in this help page.

	- __fmt__
	Is used to format the help message if available.

	- __less__ (or __more__)
	Is used to page this help unless \$PAGER is set to something else.

