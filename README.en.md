<div align="center">
  <div align="center">
    <a href="README.md">中文</a> | English
  </div>
</div>

# AdGuard Home for IPFire

AdGuard Home is a network-wide DNS filtering solution that provides ad blocking and privacy protection for all devices on your home or business network. All DNS requests from clients, including smartphones, computers, smart TVs and IoT devices, are routed through AdGuard Home, which can block advertisements, trackers and malicious domains while providing secure and controllable DNS resolution.

This project integrates AdGuard Home with the IPFire firewall and provides a native management experience on IPFire. It includes service management scripts, an IPFire Web UI menu entry, CGI management pages and a persistent configuration directory, allowing users to install, configure and maintain AdGuard Home as if it were a built-in IPFire component.

Tested on IPFire 2.29 (x86_64) - Core Update 203.

## User Interface

![](image/AdGuardHome.en.png)

## Recommended DNS Architecture

```text
Client -> AdGuard Home:53 -> Knot Resolver:5353 -> Upstream DNS
```

## Install

```sh
sh install.sh
```

After the first startup, open:

```text
http://<ipfire-host>:3000/
```

## Take Over Port 53

```sh
/etc/rc.d/init.d/adguardhome stop
cp -a /etc/knot-resolver/config.yaml /etc/knot-resolver/config.yaml.bak.$(date +%Y%m%d%H%M%S)
cp -a /var/ipfire/adguardhome/AdGuardHome.yaml /var/ipfire/adguardhome/AdGuardHome.yaml.bak.$(date +%Y%m%d%H%M%S)

sed -i 's/interface: 0.0.0.0@53/interface: 127.0.0.1@5353/' /etc/knot-resolver/config.yaml

perl -0pi -e 's/(dns:\n(?:.*\n)*?  port: )\d+/${1}53/s; s/(  upstream_dns:\n)(?:    - .*\n)+/${1}    - 127.0.0.1:5353\n/s' /var/ipfire/adguardhome/AdGuardHome.yaml

/etc/rc.d/init.d/knot-resolver restart
/etc/rc.d/init.d/adguardhome start
```

## Verification

```sh
ss -lntup | grep -E ':(53|5353|3000)\b'
dig @127.0.0.1 -p 53 example.com
dig @127.0.0.1 -p 5353 example.com
```

## Restore Default Configuration

```sh
/etc/rc.d/init.d/adguardhome stop
sed -i 's/interface: 127.0.0.1@5353/interface: 0.0.0.0@53/' /etc/knot-resolver/config.yaml
/etc/rc.d/init.d/knot-resolver restart
```

## Uninstall

```sh
sh uninstall.sh
```

## Disclaimer

This is an unofficial community project and is not affiliated with, endorsed by, or supported by the IPFire team. Please review the source code carefully before deployment and use it at your own risk.
