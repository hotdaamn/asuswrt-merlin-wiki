The GUI page has no such ability, so I am using the command below. It needs to be run on the router and the output will be a sorted (by IP) list of all DHCP reservations.
```
nvram get dhcp_staticlist | sed 's/</\n/g' | grep ":" | awk -F">" '{ print $2">"$1">"$3; }' | \
sort -t . -k 1,1n -k 2,2n -k 3,3n -k 4,4n | awk -F">" '{ print $2">"$1">"$3; }' | \
sed 's/^/</g' | tr -d '\n' | sed 's/^/nvram set dhcp_staticlist="/' | sed 's/$/"\n/' | \
tee /tmp/dhcp_list_sorted_tmp.sh
```
The output is as below, i.e. a ready-to-run command to re-set the list. 
```
nvram set dhcp_staticlist="<XX:XX:XX:XX:XX:XX>192.168.1.100>name1<YY:YY:YY:YY:YY:YY>192.168.1.101>name2<ZZ:ZZ:ZZ:ZZ:ZZ:ZZ>192.168.1.102>name3"
```
To apply the change and if you are satisfied with the result, run the commands below 
```
sh /tmp/dhcp_list_sorted_tmp.sh
nvram commit
```
