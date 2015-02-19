I use the guest wifi for our eldest daughter - if she doesn't get her chores done, she doesn't get the password for the day!

Rather than manually coming up with a suitable random password on a daily basis, I wanted a way that it would be set automatically every day, with the randomly generated phrase being emailed to myself and my wife so that we could provide it when we felt it had been earned.

While the below may not be perfect (it's a long time since I did scripting) I wanted to ensure it was something that could be set up without needing optware or entware installed.

The password is made up of a phrase selected from a text file (I used various band names) followed by a random three digit number. I'm only using one guest network, but this could be easily modified to pick three different passwords for three different networks, or even six. You would just need to assign new variables to phrasepwd after running the getrandomphrase procedure.

First of all, create the following as /jffs/scripts/rpg-passgen.sh and ensure you make it executable :

    #!/bin/sh

    # define variables for sending email - you will need to change these!
    FROM="login@gmail.com"
    AUTH="login@gmail.com"
    PASS="password"
    FROMNAME="Asus Router"
    TO="your@email.com; other@email.com"

    # default password based on date if we cannot create one based on phrase file 
    datepasswd=`date +"%A%B%d"`
    
    # define our function 
    getrandomphrase () {
        if [ -f /jffs/scripts/rpg-phrases.txt ]; then
            phrasecount=`wc -l /jffs/scripts/rpg-phrases.txt | cut -d " " -f 1`
            if [ $phrasecount == 0 ]; then
                # file is empty
                phrasepasswd=$datepasswd
            else
                randomnumber=`tr -cd 0-9 </dev/urandom | head -c 7`
                phrasetext=`sed -n $(( $randomnumber % $phrasecount + 1 ))p /jffs/scripts/rpg-phrases.txt`
                if [ $phrasetext == "" ]; then
                    # we hit a blank line in file, bailing  
                    phrasepasswd=$datepasswd 
                else
                    if [ ${#phrasetext} -lt 7 ]; then
                        # phrase is too short to make a valid password 
                        phrasepasswd=$datepasswd
                    else
                        # we have a phrase now get the three digit number  
                        randomnumber=`tr -cd 0-9 </dev/urandom | head -c 7`
                        phrasenum=`printf "%03d" $(( $randomnumber % 1000 ))`
                        phrasepasswd=$phrasetext$phrasenum
                    fi
                fi
            fi
        else
            # file does not exist 
            phrasepasswd=$datepasswd
        fi
    }
    
    # now call the function 
    getrandomphrase
    
    # log what we have done 
    logger -t $(basename $0) "Today's Guest1 password is :" $phrasepasswd
    
    # nvram settings for the three guest 2.4 networks
    nvram set wl0.1_wpa_psk=$phrasepasswd
    nvram set wl0.2_wpa_psk=$datepasswd
    nvram set wl0.3_wpa_psk=$datepasswd
    
    # nvram settings for the three guest 5.0 networks
    nvram set wl1.1_wpa_psk=$phrasepasswd
    nvram set wl1.2_wpa_psk=$datepasswd
    nvram set wl1.3_wpa_psk=$datepasswd
    
    # passwords have been changed but we need to restart the wifi for it to pick them up
    service restart_wireless
    
    # now send out the email 
    echo "Subject: Guest network password notification" >/tmp/mail.txt
    echo "From: \\"$FROMNAME\\"<$FROM>" >>/tmp/mail.txt
    echo "Date: `date -R`" >>/tmp/mail.txt
    echo "" >>/tmp/mail.txt
    echo "Today's Guest1 password is : $phrasepasswd" >>/tmp/mail.txt
    echo "" >>/tmp/mail.txt
    
    cat /tmp/mail.txt | sendmail -H"exec openssl s_client -quiet \
    -CAfile /jffs/configs/Equifax_Secure_Certificate_Authority.pem \
    -connect smtp.gmail.com:587 -tls1 -starttls smtp" \
    -f"$FROM" \
    -au"$AUTH" -ap"$PASS" $TO 
    
    rm /tmp/mail.txt


In the event that we cannot get a good phrase to create a password, we create one based on the date, so at least the password will change on a daily basis, even if it ends up being predictable! :)

I've configured the script to use gmail, but you could also amend this to use your ISP smtp server instead. Gmail requires the secure certificate to be installed in /jffs/configs/ - if you haven't got this, check the following post as the wget command referred to on the wiki doesn't work :

[http://forums.smallnetbuilder.com/showpost.php?p=149473&postcount=95](http://forums.smallnetbuilder.com/showpost.php?p=149473&postcount=95)

Next, create the following as /jffs/scripts/rpg-phrases.txt :

    greenday
    alicecooper
    ledzeppelin
    aerosmith
    ironmaiden
    metallica
    foofighters
    blacksabbath
    defleppard

These phrases are the basis for the password - I've chosen band names, but you could use your kids names or places or anything you want. Ensure that each phrase is at least 7 characters long (+3 for random number = min length of 10 characters) and that there are no blank lines. Try and have a reasonable number of entries in here, or you will end up with the same phrase being picked on a regular basis. My full phrases file contains about 30 or 40 different bands!

To get this process to run at 4am each day, add the following into /jffs/scripts/init-start and make it executable :

    #!/bin/sh
    cru a ResetGuestPassword "0 4 * * * /jffs/scripts/rpg-passgen.sh"

This will set it up so that the script will be run at 4am every day.

Finally, to get this to run on each boot (rather than having to wait until 4am the next day) add the following to /jffs/scripts/services-start and make it executable it :

    #!/bin/sh
    sh /jffs/scripts/rpg-passgen.sh

Reboot your router, and you're done!

If you wish to comment on this, add improvements, discuss errant teenagers, then I have created a thread here : 

[http://forums.smallnetbuilder.com/showthread.php?t=22700](http://forums.smallnetbuilder.com/showthread.php?t=22700)