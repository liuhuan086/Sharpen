这是一个非常简单的脚本，它每天用默认的端口运行nmap，然后使用ndiff比较结果。

```bash
#!/bin/bash
mkdir /opt/nmap_diff
d=$(date +%Y-%m-%d)
y=$(date -d yesterday +%Y-%m-%d)
/usr/bin/nmap -T4 -oX /opt/nmap_diff/scan_$d.xml 10.100.100.0/24 >
/dev/null 2>&1
if [ -e /opt/nmap_diff/scan_$y.xml]; then
/usr/bin/ndiff /opt/nmap_diff/scan$y.xml /opt/nmap_diff/scan$d.xml >
/opt/nmap_diff/diff.txt
fi
```



```bash
#!/bin/sh
TARGETS="192.168.1.0/24"
OPTIONS="-v -T4 -F -sV"
date=$(date +%Y-%m-%d-%H-%M-%S)
cd /nmap/diffs
nmap $OPTIONS $TARGETS -oA scan-$date > /dev/null
slack(){
curl -F file=@diff-$date -F initial_comment="Internal Port Change Detected" -F channels=#alerts -F token=xxxx-xxxx-xxxx https://slack.com/api/files.upload
}

if [ -e scan-prev.xml ]; then
ndiff scan-prev.xml scan-$date.xml > diff-$date
[ "$?" -eq "1" ] && sed -i -e 1,3d diff-$date && slack
fi
ln -sf scan-$date.xml scan-prev.xml
```

