# openHAB-xbox-remote-power
Power on/off a xbox one with openHAB and the [Exec Binding](https://www.openhab.org/addons/bindings/exec/) using the python script [xbox-remote-power](https://github.com/Schamper/xbox-remote-power). Additionally I use the [Network Binding](https://www.openhab.org/addons/bindings/network/).

## Things

The Thing `network:pingdevice:xbox` is using the [Network Binding](https://www.openhab.org/addons/bindings/network/) and the both other things are using the [Exec Binding](https://www.openhab.org/addons/bindings/exec/).

```
Thing network:pingdevice:xbox   "Network Device: Xbox One"   [ hostname="<ip>", macAddress="<mac>", retry=1, timeout=5000, refreshInterval=60000 ]
Thing exec:command:XboxPowerOn [command="/usr/bin/python3 </path/to>/xbox-remote-power.py -a <ip> -i <live_id>"]
Thing exec:command:XboxPingState [command="/usr/bin/python3 </path/to>/xbox-remote-power.py -a <ip> -i <live_id> -p"]
```

You have to change `<ip>` to the ip address of your xbox. Also you have to change `<mac>` to the mac address of your xbox. You can receive this informations by opening the network settings on your xbox. That the python script can run you also have to change the `<live_id>`. I found nothing which sound like `live id`. But I found instead Serial number, Console ID, Xbox Network Device ID, and Global Device ID. It worked for me to use the `Xbox Network Device ID`. So please replace `<live_id>` with your Xbox Network Device ID.

At least you have to add following two line to `/etc/openhab/misc/exec.whitelist`:

```
/usr/bin/python3 </path/to>/xbox-remote-power.py -a <ip> -i <live_id>
/usr/bin/python3 </path/to>/xbox-remote-power.py -a <ip> -i <live_id> -p
```

## Items

You have to create following items:

```
Group Xbox "Xbox One" <player>

Switch Xbox_Schalter "Power on" (Xbox) { channel="exec:command:XboxPowerOn:run" }
String Xbox_Online_State "Network Device: Xbox One Online Status" (Xbox)  { channel="network:pingdevice:xbox:online" }
Number:Time Xbox_ResponseTime "Network Device: Xbox One Response Time" (Xbox)  { channel="network:pingdevice:xbox:latency" }
DateTime Xbox_LastSeen "Network Device: Xbox One Point Last Seen"  (Xbox)     { channel="network:pingdevice:xbox:lastseen" }
String Xbox_Ping_Status "Ping-Status" (Xbox) { channel="exec:command:XboxPingState:run" }
```

## Rules

It make sense that if you powered on the Xbox that your TV will also be powered on and changed it source to the source where your xbox is connected to.

```
rule "Xbox One On"
when
    Item Xbox_Schalter changed from OFF to ON
then
    SmartTV_Power.sendCommand(ON)
    SmartTV_Source_Name.sendCommand("HDMI2")
end

rule "Xbox One Off"
when
    Item Xbox_Schalter changed from ON to OFF
then
    SmartTV_Power.sendCommand(OFF)
end
```

## Sitemaps

At least you have to add following to your sitemap:

```
Text label="Xbox One" icon="player" {
    Switch item=Xbox_Schalter
    Switch item=Xbox_Online_State
    Text item=Xbox_ResponseTime
    Text item=Xbox_LastSeen
    Text item=Xbox_Ping_Status
}
```
