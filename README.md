# maildir-migration
A set of scripts, tools, and notes to help convert from an mbox-formatted set of mailboxes to Maildir mailboxes.

The Obscure Organization has historically used mbox-format mailboxes, but we are finally moving to Maildir.

As part of this effort, we are suspending mail delivery to mailboxes that have not been checked for more than a year. See the `find-active-mail-users` script.

mbox to maildir conversion for obscure.org
==========================================
I followed the guidelines here roughly:

http://adam.rosi-kessel.org/weblog/2007/04/18/adams-super-simple-guide-to-mbox-maildir-conversion

I did this once straight through with mb2md 3.20, and found that the messages
were in a weirdly random order in alpine in the 'arrived' state. To fix this I
amended the mb2md program to prefix mail file names with the timestamp
corresponding to the From date of the email. Those changes are now here:

https://github.com/obscureorganization/mb2md

The mb2md.pl script lives in /usr/local/bin/mb2md.pl and
the sources are in /usr/local/src/production/mb2md on tiamat:

I cleaned up my own mail directories before I began:

* Changed the procmail mail default directory back to mail/
* Changed the names of procmail-sorted incoming directories so that they started with ">" because they sort better that way in iOS mail

Here is a snippet of my procmail config: 
```
MAILDIR=$HOME/Maildir
DEFAULT=$MAILDIR/
#Set on when debugging
VERBOSE=off
#Directory for storing procmail log and rc files
PMDIR=$HOME/.procmail
LOGFILE=$PMDIR/log

:0
* ^X-Spam-Status: Yes
.ZZZ_caughtspam/


* ^TOmajordomo@obscure.org
.majordomo.log/

# SSL Certificate warnings need to go into main inbox
:0
* ^TOpostmaster@obscure.org
* $ ${FROM}(comodo.com)
$DEFAULT

# Put messages to postmaster in their own folder
:0
* ^TO(postmaster|MAILER-DAEMON|majordomo-owner)
.AAA_postmaster/

```

Alter procmail recipies to have all folders prefixed with "." and suffixed with
"/" and convert mailbox names with "." in them to "_", since mb2md changes
mailbox names that have a "." in them to have a "_" instead.

Here is the series of scripts that I ran to convert my own mail spool:

time ~/bin/mb2md.pl -U -K -S -t -m
echo time ~/bin/mb2md.pl -U -K -S -t -s mail -R | batch

It would be better to do it all in one go, though, and batch is
a good way of moderating that:
batch <<EOF
time /usr/local/bin/mb2md.pl -U -K -S -t -m
time /usr/local/bin/mb2md.pl -U -K -S -t -s mail -R
EOF

This script is now in convert-mbox-to-maildir in this project.

Here is a good way to gauge how many people are still using the legacy imapd:

sudo grep imapd /var/log/maillog /var/log/maillog.*  | awk '/Login/{print $7}'
| cut -f2 -d= |  sort -u > imapusers.txt
wc -l imapusers.txt


