Asuswrt-Merlin includes a cutdown version of sendmail which can be used to send mail from the router.  Basic usage will look like this:

```
echo "This is a test email." | /usr/sbin/sendmail -S"smtp.yourisp.com" -f"router@domain.com" you@domain.com
```

You can send a more proper Email by writing the mail to a file, and then piping it to sendmail.  The following example, if saved as a wan-start script, will make your router send you an Email whenever the WAN interface comes up, containing the WAN IP:

(WIP)