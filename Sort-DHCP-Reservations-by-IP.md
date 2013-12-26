The GUI page has no such ability, so I am using the command below. It needs to be run on the router and the output will be a sorted list of all DHCP reservations.

nvram get dhcp_staticlist | sed 's/</\n/g' | grep ":" | awk -F">" '{ print $2">"$1">"$3; }' | sort -t . -k 1,1n -k 2,2n -k 3,3n -k 4,4n | awk -F">" '{ print $2">"$1">"$3; }' | sed 's/^/</g' | tr -d '\n' | sed 's/^/nvram set dhcp_staticlist="/' | sed 's/$/"\n/'

The output is as below, i.e. a ready-to-run command to re-set the list followed by "nvram commit" .

nvram set dhcp_staticlist="......"