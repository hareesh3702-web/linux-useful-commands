# Mail Troubleshooting Commands (Exim / Postfix / cPanel / DirectAdmin / Plesk)

---

# 1. Mail Queue Size

### Exim

```bash
exim -bpc
```

Shows total messages in queue.

```bash
exim -bp
```

Lists all queued messages.

```bash
exim -bp | exiqsumm
```

Queue summary by sender.

```bash
mailq
```

Shows mail queue.

---

### Postfix

```bash
postqueue -p
```

```bash
mailq
```

Queue count:

```bash
postqueue -p | tail -1
```

---

# 2. Find Queue Messages for a Specific Domain

### Exim

```bash
exiqgrep -r domain.com
```

```bash
exiqgrep -f user@domain.com
```

```bash
exiqgrep -r gmail.com
```

---

### Postfix

```bash
mailq | grep domain.com
```

---

# 3. View Message Content

### Exim

```bash
exim -Mvh MESSAGE-ID
```

Headers only.

```bash
exim -Mvb MESSAGE-ID
```

Body only.

```bash
exim -Mvl MESSAGE-ID
```

Full log for the message.

---

# 4. Remove Emails from Queue

### Single Message

```bash
exim -Mrm MESSAGE-ID
```

---

### Remove All Messages From Sender

```bash
exiqgrep -i -f sender@domain.com | xargs exim -Mrm
```

---

### Remove All Messages To Recipient

```bash
exiqgrep -i -r user@domain.com | xargs exim -Mrm
```

---

### Remove Entire Domain

```bash
exiqgrep -i -f domain.com | xargs exim -Mrm
```

---

### Remove Frozen Messages

```bash
exiqgrep -zi | xargs exim -Mrm
```

---

# 5. Force Queue Delivery

### Exim

```bash
exim -qff
```

Run entire queue immediately.

```bash
exim -M MESSAGE-ID
```

Retry specific message.

---

### Postfix

```bash
postqueue -f
```

Force queue run.

---

# 6. Search Exim Logs

### cPanel

Main log:

```bash
/var/log/exim_mainlog
```

Reject log:

```bash
/var/log/exim_rejectlog
```

Panic log:

```bash
/var/log/exim_paniclog
```

---

### Useful Searches

Sender:

```bash
grep "sender@domain.com" /var/log/exim_mainlog
```

Recipient:

```bash
grep "user@domain.com" /var/log/exim_mainlog
```

Message ID:

```bash
grep "1xYZaA-000123-AB" /var/log/exim_mainlog
```

Recent entries:

```bash
tail -100 /var/log/exim_mainlog
```

Live monitoring:

```bash
tail -f /var/log/exim_mainlog
```

Failures only:

```bash
grep "failed" /var/log/exim_mainlog
```

Deferred mails:

```bash
grep "defer" /var/log/exim_mainlog
```

Rejected mails:

```bash
grep "rejected" /var/log/exim_rejectlog
```

---

# 7. Trace Email Delivery

Find Message ID:

```bash
grep "user@domain.com" /var/log/exim_mainlog
```

Then:

```bash
exim -Mvl MESSAGE-ID
```

Shows complete delivery history.

---

# 8. Find Top Mail Senders

Today:

```bash
eximstats /var/log/exim_mainlog | less
```

Top senders:

```bash
grep "<=" /var/log/exim_mainlog | awk '{print $5}' | sort | uniq -c | sort -nr | head
```

Top domains:

```bash
grep "<=" /var/log/exim_mainlog | awk '{print $5}' | cut -d@ -f2 | sort | uniq -c | sort -nr | head
```

---

# 9. Check Which IP Sent an Email

Search sender:

```bash
grep "sender@domain.com" /var/log/exim_mainlog
```

Example output:

```text
H=(server.example.com) [1.2.3.4]
```

The IP inside [] is the source IP.

---

# 10. Find Emails Sent from Specific IP

```bash
grep "\[1.2.3.4\]" /var/log/exim_mainlog
```

Count:

```bash
grep "\[1.2.3.4\]" /var/log/exim_mainlog | wc -l
```

---

# 11. Find Authenticated SMTP Users

```bash
grep "A=dovecot_login" /var/log/exim_mainlog
```

Specific user:

```bash
grep "user@domain.com" /var/log/exim_mainlog | grep "A=dovecot_login"
```

---

# 12. Check SMTP Authentication Failures

```bash
grep "authentication failed" /var/log/exim_mainlog
```

```bash
grep "failed" /var/log/exim_mainlog | grep LOGIN
```

---

# 13. Mail Log Locations

## cPanel

```bash
/var/log/exim_mainlog
/var/log/exim_rejectlog
/var/log/exim_paniclog
```

## DirectAdmin

```bash
/var/log/exim/mainlog
/var/log/exim/rejectlog
/var/log/exim/paniclog
```

## Plesk (Postfix)

```bash
/var/log/maillog
```

or

```bash
/var/log/mail.log
```

---

# 14. Dovecot / POP3 / IMAP Logs

### cPanel

```bash
grep dovecot /var/log/maillog
```

```bash
tail -f /var/log/maillog
```

Authentication failures:

```bash
grep "auth failed" /var/log/maillog
```

Successful logins:

```bash
grep "Login:" /var/log/maillog
```

Specific mailbox:

```bash
grep "user@domain.com" /var/log/maillog
```

---

# 15. Find High Mail Queue Domains

```bash
exim -bp | grep "<" | awk '{print $4}' | cut -d@ -f2 | sort | uniq -c | sort -nr | head -20
```

---

# 16. Check Mail Rate Abuse

Messages sent by user:

```bash
grep "cwd=/home/USERNAME" /var/log/exim_mainlog
```

PHP mail() abuse:

```bash
grep "cwd=/home/" /var/log/exim_mainlog | grep "P=local"
```

Top PHP senders:

```bash
grep "cwd=/home/" /var/log/exim_mainlog | awk -F'cwd=' '{print $2}' | cut -d' ' -f1 | sort | uniq -c | sort -nr | head
```

---

# 17. Search Queue by Subject

```bash
exim -bp | exiqgrep -i
```

View message body:

```bash
exim -Mvb MESSAGE-ID
```

Search for subject text:

```bash
grep -Ri "Invoice Payment" /var/spool/exim/input/
```

---

# 18. Delete Emails With Same Subject

Find IDs:

```bash
grep -Rl "Invoice Payment" /var/spool/exim/input/ | sed 's/-H$//' | awk -F/ '{print $NF}'
```

Delete:

```bash
for i in $(grep -Rl "Invoice Payment" /var/spool/exim/input/ | sed 's/-H$//' | awk -F/ '{print $NF}'); do exim -Mrm $i; done
```

---

# 19. Check Mail Services

### Exim

```bash
systemctl status exim
```

Restart:

```bash
systemctl restart exim
```

---

### Postfix

```bash
systemctl status postfix
```

Restart:

```bash
systemctl restart postfix
```

---

### Dovecot

```bash
systemctl status dovecot
```

Restart:

```bash
systemctl restart dovecot
```

---

# 20. Most Useful Commands During Mail Troubleshooting

```bash
exim -bpc
mailq
tail -f /var/log/exim_mainlog
grep "user@domain.com" /var/log/exim_mainlog
exim -Mvl MESSAGE-ID
grep dovecot /var/log/maillog
grep "A=dovecot_login" /var/log/exim_mainlog
exiqgrep -zi
exim -qff
eximstats /var/log/exim_mainlog
```

These are the commands I use most often when troubleshooting mail delivery, queue issues, SMTP authentication problems, spam outbreaks, POP3/IMAP duplicate mail issues, Outlook blacklisting, and PHP mail abuse on cPanel/WHM servers.

