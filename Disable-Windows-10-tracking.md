_Please note, this how-to will work only on asuswrt-merlin releases newer then 378.55._

This solution consists of two parts:

* collecting resolved IPs from unwanted domains list to IP set with dnsmasq,
* dropping traffic from/to collected IPs with firewall rule.

First, enable JFFS custom scripts and configs from WebUI. Put following list of unwanted domains to `/jffs/Win10tracking.txt`:
```
a.ads1.msn.com
a.ads2.msads.net
a.ads2.msn.com
a.rad.msn.com
a-0001.a-msedge.net
a-0002.a-msedge.net
a-0003.a-msedge.net
a-0004.a-msedge.net
a-0005.a-msedge.net
a-0006.a-msedge.net
a-0007.a-msedge.net
a-0008.a-msedge.net
a-0009.a-msedge.net
ac3.msn.com
ad.doubleclick.net
adnexus.net
adnxs.com
ads.msn.com
ads1.msads.net
ads1.msn.com
aidps.atdmt.com
aka-cdn-ns.adtech.de
a-msedge.net
apps.skype.com
az361816.vo.msecnd.net
az512334.vo.msecnd.net
b.ads1.msn.com
b.ads2.msads.net
b.rad.msn.com
bs.serving-sys.com
c.atdmt.com
c.msn.com
cdn.atdmt.com
cds26.ams9.msecn.net
choice.microsoft.com
choice.microsoft.com.nsatc.net
compatexchange.cloudapp.net
corp.sts.microsoft.com
corpext.msitadfs.glbdns2.microsoft.com
cs1.wpc.v0cdn.net
db3aqu.atdmt.com
df.telemetry.microsoft.com
diagnostics.support.microsoft.com
ec.atdmt.com
fe2.update.microsoft.com.akadns.net
feedback.microsoft-hohm.com
feedback.search.microsoft.com
feedback.windows.com
flex.msn.com
g.msn.com
h1.msn.com
i1.services.social.microsoft.com
i1.services.social.microsoft.com.nsatc.net
lb1.www.ms.akadns.net
live.rads.msn.com
m.adnxs.com
m.hotmail.com
msedge.net
msftncsi.com
msnbot-65-55-108-23.search.msn.com
msntest.serving-sys.com
oca.telemetry.microsoft.com
oca.telemetry.microsoft.com.nsatc.net
pre.footprintpredict.com
preview.msn.com
pricelist.skype.com
rad.live.com
rad.msn.com
redir.metaservices.microsoft.com
reports.wes.df.telemetry.microsoft.com
s.gateway.messenger.live.com
s0.2mdn.net
schemas.microsoft.akadns.net
secure.adnxs.com
secure.flashtalking.com
services.wes.df.telemetry.microsoft.com
settings-sandbox.data.microsoft.com
settings-win.data.microsoft.com
sls.update.microsoft.com.akadns.net
sqm.df.telemetry.microsoft.com
sqm.telemetry.microsoft.com
sqm.telemetry.microsoft.com.nsatc.net
static.2mdn.net
statsfe1.ws.microsoft.com
statsfe2.update.microsoft.com.akadns.net
statsfe2.ws.microsoft.com
survey.watson.microsoft.com
telecommand.telemetry.microsoft.com
telecommand.telemetry.microsoft.com.nsatc.net
telemetry.appex.bing.net
telemetry.microsoft.com
telemetry.urs.microsoft.com
view.atdmt.com
vortex.data.microsoft.com
vortex-bn2.metron.live.com.nsatc.net
vortex-cy2.metron.live.com.nsatc.net
vortex-sandbox.data.microsoft.com
vortex-win.data.microsoft.com
watson.live.com
watson.microsoft.com
watson.ppe.telemetry.microsoft.com
watson.telemetry.microsoft.com
watson.telemetry.microsoft.com.nsatc.net
wes.df.telemetry.microsoft.com
www.msftncsi.com
```
I took mine list from [here](http://forums.zyxmon.org/go.php?https://github.com/10se1ucgo/DisableWinTracking/blob/master/run.py#L208). Now put following content to `/jffs/scripts/firewall-start`:
```
#!/bin/sh
DNSMASQ_CFG=/jffs/configs/dnsmasq.conf.add
if [ ! -f $DNSMASQ_CFG ] || [ "$(grep Win10tracking $DNSMASQ_CFG)" = "" ];
then
  rm -f $DNSMASQ_CFG
  for i in `cat /jffs/Win10tracking.txt`;
  do
    echo "ipset=/$i/Win10tracking" >> $DNSMASQ_CFG
  done
  service restart_dnsmasq
fi

# Load ipset modules
lsmod | grep "ipt_set" > /dev/null 2>&1 || \
for module in ip_set ip_set_nethash ip_set_iphash ipt_set
do
    insmod $module
done

# Create ip set
if [ "x$(ipset --swap Win10tracking Win10tracking 2>&1 | grep 'Unknown set')" != "x" ]
then
  ipset -N Win10tracking iphash
fi

# Apply iptables rule
iptables-save | grep Win10tracking > /dev/null 2>&1 || \
  iptables -I FORWARD -m set --set Win10tracking src,dst -j DROP
```
Don't forget to make script executable and reboot router to take effect:
```
chmod +x /jffs/scripts/firewall-start
reboot
```
You may check it's working by trying to open some site from list (view.atdmt.com for example). Then check "black list" is populated with some IP addresses:
```
ipset --list Win10tracking
```