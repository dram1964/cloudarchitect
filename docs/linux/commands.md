# WiFi Commands
The `lshw` command can be used to quickly identify network hardware on your device:
```
sudo lshw -C network
```
The output will include both MAC address in the 'serial' field and IP address in the 'configuration' 
field, along with other information such as the 'logical name' and the 'bus id'. `ifconfig` 
will also provide the MAC and IP addresses for connect wireless adapters. Use `iwconfig` 
to see the speed and link quality for your wireless adapter
The `nmcli` command provides useful information from Network Manager. To see the saved 
connections for your device use:
```
nmcli connection show
```
To see a list of Access Points and stats on their signal strength and capabilities, use:
```
nmcli dev wifi
```
`wavemon` is an ncurses-based monitoring tool for WiFi
