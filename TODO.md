# TODO:

* In release v1.0.1 a new pair of alert states will be introduced:<br>
  LINKDOWN - LINKUP<br>
  In version 1.0.0, if there is no link on the interface a WARNING alert is sent with the idea of this could possibly be a configuration issue with to short --ifup-grace or tweaking --max-nolink BUT if the link is lost e.g. due to switch being down, cable out, etc and we use --force-interface and --max-nolink > 0 there will be repeated alerts on the same issue sent every ~ max-nolink * ifup-grace seconds With the new alert states thare will be only one alert if the link is down and we can't get it up and one when it comes up again.

