Asuswrt-Merlin includes a cutdown version of sendmail which can be used to send mail from the router.  Basic usage will look like this:

```
echo "This is a test email." | /usr/sbin/sendmail -S"smtp.yourisp.com" -f"router@domain.com" you@domain.com
```

You can send a more proper Email by writing the mail to a file, and then piping it to sendmail.  The following example, if saved as a wan-start script, will make your router send you an Email whenever the WAN interface comes up, containing the WAN IP:

```
#!/bin/sh
SMTP="smtp.yourisp.com"
FROM="router@domain.com"
FROMNAME="Your Router"
TO="you@domain.com"

echo "Subject: WAN state notification" >/tmp/mail.txt
echo "From: \\"$FROMNAME\\"<$FROM>" >>/tmp/mail.txt
echo "I just got connected to the Interwebz." >>/tmp/mail.txt
echo "My new IP is: `nvram get wan0_ipaddr`" >>/tmp/mail.txt
echo "" >>/tmp/mail.txt
echo "--- " >>/tmp/mail.txt
echo "Your friendly router." >>/tmp/mail.txt

cat /tmp/mail.txt | /usr/sbin/sendmail -S"$SMTP" -f"$FROM" $TO

rm /tmp/mail.txt
```

If your SMTP server requires authentication, you can pass them as additional arguments.  To see the available options, just run "sendmail -h".
