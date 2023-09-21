# Linux Commands

- Check your linux version

        lsb_release -a

- Check system startup time

        systemd-analyze

- Check  startup time by individual service

        systemd-analyze blame

- Disable an individual service

        sudo systemctl disable NetworkManager-wait-online.service

- Find process ids based on name patterns:

        pgrep firefox

- Find process ids based on username: 

        pgrep -u <username>

## WiFi Commands
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

## Network Commands

To see your current IP address instead of using *ifconfig* try *ip*:
```bash
ip address show
```
This will show:

1. Interface name
2. MAC address: *link/ether*
3. IP address: *inet*
4. State of the Interface: UP or DOWN

You can also use `ip link show` to check the state of your interfaces.

Use the *route* command to show the routing table on a computer:
```bash
route
```

The default route shows you which interface is used if no specific 
route matches the address specified. 

You can also use *ip route show* to show the route to a specific network:
```bash
ip route show 10.0.0.0/8
```

You can get help on the *ip* command using *man ip-address*, *man ip-link* 
or *man ip-route*. 

Use the *arp* command to check the ARP table (data link layer) addresses
stored locally. This will show the mapping of physical addresses to ip 
addresses for devices on the same LAN (e.g. your gateway or router):
```bash
arp -a 
```

To check for ports on localhost use *ss*:
```bash
ss -tunlp4
```

Whilst *telnet* can be used to test connectivity to TCP ports, netcat
also provides the ability to check connections to UDP ports:
```bash
nc 192.168.0.3 -u 80 -v
nc 127.0.0.1 8000 -v
```

Netcat can also be used to quickly setup a server to listen on a 
specified port. 
```bash 
nc -l 1234 > filename.out
```

All the input sent to port 1234 will be copied to *filename.out*:
```bash
nc -N myhost.example.com 1234 < filename.in
```

These two commands will allow you to transfer the contents if *filename.in* to *filename.out* on the listening server. 

The *-N* option closes the connection on termination.

## sed

*sed* stands for Stream Editor, and can be used to replace text from 
input to output:

    echo 'Hello, World!' | sed 's/World/Universe/'

sed can also be used directly on a file:

    sed 's/left/right/' <filename>

By default output is sent to STDOUT, but you can change the file 
contents using the *-i* switch:

    sed -i 's/left/right/' <filename>

Sed allows capturing groups and backreferences. To capture IP Address from
the weblog.txt use:

    sed -nE 's/.* h=(\d+\.\d+\.\d+\.\d+).*$/\1/p' weblog.txt

Sed works well in pipelines: to find unique values of ip_address in 
above example use:

    sed -nE 's/.* h=(\d+\.\d+\.\d+\.\d+).*$/\1/p' weblog.txt | sort -u

To count the occurrences for each unique IP Address use:

    sed -nE 's/.* h=(\d+\.\d+\.\d+\.\d+).*$/\1/p' weblog.txt | sort | uniq -c

You can also pipe the list of IPs to lookup the hostnames:

    sed -nE 's/.* h=(\d+\.\d+\.\d+\.\d+).*$/\1/p' weblog.txt | \
    sort -u | awk '{ cmd = "dig +short -x " $1; system(cmd) }'
