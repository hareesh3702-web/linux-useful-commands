# Disk Space Investigation Commands (Linux)

## 1. Filesystem Usage

```bash
df -h
```

Shows overall filesystem usage.

```bash
df -ih
```

Shows inode usage.

---

## 2. Current Directory Usage

```bash
du -sch .[!.]* *
```

Shows size of all files and directories including hidden files.

```bash
du -sch * | awk '$1+0 > 15 && /G/'
```

Shows items larger than 15 GB in the current directory.

```bash
ls -lh --time-style=long-iso
```

Lists files with sizes and timestamps.

---

## 3. Find Large Home Directories

```bash
cd /home && du -h --max-depth=1 | sort -hr
```

Shows disk usage per cPanel account.

```bash
cd /home && du -h --max-depth=1 | awk '$1 ~ /[0-9]+G/ && $1+0 > 40'
```

Shows accounts using more than 40 GB.

---

## 4. Find Large Files

```bash
find /home -type f -size +200M -exec ls -lh {} + 2>/dev/null
```

Lists files larger than 200 MB.

```bash
find /home -type f -name "*.sql" -size +200M -exec ls -lh {} + 2>/dev/null
```

Lists SQL dumps larger than 200 MB.

```bash
find /home -type f -name "*.sql" -size +1G
```

Lists SQL dumps larger than 1 GB.

```bash
find / -type f -size +300M -iname "*.tar.gz" 2>/dev/null
```

Lists TAR.GZ files larger than 300 MB.

```bash
find / -type f -size +200M -iname "*.zip" 2>/dev/null
```

Lists ZIP files larger than 200 MB.

---

## 5. Updraft Backup Investigation

```bash
find /home -type d -name updraft -exec du -sh {} \;
```

Shows size of all Updraft backup directories.

```bash
find /home -type d -name updraft -exec du -sBG {} + | awk '$1+0 > 4'
```

Shows Updraft directories larger than 4 GB.

```bash
find /home -type f -path "*/updraft/*" -name "*.zip" -mtime +2
```

Lists Updraft backup ZIP files older than 2 days.

```bash
find /home -type f -path "*/updraft/*" -name "*.zip" -mtime +2 -delete
```

Deletes Updraft backup ZIP files older than 2 days.

---

## 6. LiteSpeed Cache (LSCache)

```bash
find /home* -type d -iname lscache -exec du -sh {} \;
```

Shows size of all LSCache directories.

```bash
find /home* -type d -iname lscache -exec du -sh --threshold=100M {} +
```

Shows LSCache directories larger than 100 MB.

```bash
find /home*/*/lscache/ -type f -mtime +1 -delete
```

Deletes LSCache files older than 1 day.

```bash
find /home*/*/lscache/ -type f -mtime +2 -delete
```

Deletes LSCache files older than 2 days.

---

## 7. Trash Directory Investigation

```bash
find /home -type d -name ".trash" -exec du -sh {} +
```

Shows all .trash directories.

```bash
find /home -type d -name ".trash" -exec du -sm {} + | awk '$1 > 100'
```

Shows .trash directories larger than 100 MB.

```bash
find /home -type d -name ".Trash" -exec du -sm {} + | awk '$1 > 100'
```

Shows .Trash directories larger than 100 MB.

```bash
find /home -type f -name '.trash*' -exec du -sh {} +
```

Shows large trash files.

---

## 8. Error Logs

```bash
find /home -type f -iname "*log" -size +100M -exec du -h {} \; 2>/dev/null
```

Lists error logs larger than 50 MB.

---

## 9. Backup Directory Usage

```bash
du -h --max-depth=1 /backup | sort -hr
```

Shows backup directory usage.

```bash
du -h --max-depth=1 /backup/disk*/ 2>/dev/null | sort -hr
```

Shows usage of all backup disks.

---

## 10. Find Suspiciously Large Directories

```bash
find /home -type d -exec du -sm {} + 2>/dev/null | sort -nr | head -50
```

Top 50 largest directories under /home.

```bash
find / -xdev -type d -exec du -sm {} + 2>/dev/null | sort -nr | head -50
```

Top 50 largest directories on the server.

---

## 11. Find Admin-Related Directories

```bash
find / -type d -iname "*admin*" 2>/dev/null
```

Finds directories containing "admin" in their name.

---

## 12. Quick Disk Usage Summary

```bash
df -h
```

```bash
df -ih
```

```bash
du -sh /home/*
```

```bash
du -sh /var/*
```

```bash
du -sh /backup/*
```

These commands cover nearly everything needed for disk-space troubleshooting on a Linux/cPanel server while removing duplicates and redundant commands from your original list.

