# The OSCP Enumeration Handbook
This is a little handbook that I made for myself so I can't miss anything that I believe I need to do for the OSCP exam.
This is not meant to help exploit things, it's meant to help you find things and then from those you do your research or exploit how you know to do it.
It also contains a bit of information that I like to have on hand.

> PRO TIP:
> Reset the machines before starting (even though you just started the machines)

**OSID:** `OSID`
**MD5:** `MD5`
If you have any issues connecting, then email proctoring@offsec.com.
# General Info
## Exam Structure
### Targets
3 Independent Targets = 60 points
- 20 points per machine (local.txt and proof.txt)
Active Directory Network = 40 points
- 2 Clients
- 1 Domain Controller
- Need Domain Admin to get any points
### Connecting to the exam VPN

```sh
tar xvfj exam-connection.tar.bz2
sudo openvpn OS-XXXXXX-OSCP.ovpn 
```

### Exam Submission
- OSCP-OS-XXXXX-Exam-Report.7z
  - PDF Report
  - Custom Exploit code Used
- upload.offsec.com
## Enumeration

### Scans to Run
- Autorecon on the standalone machines
``` sh
autorecon $IP -t standalone.txt --only-scans-dir -o autorecon --single-target --exclude-tags dirbuster

sudo $(which autorecon) $IP -t standalone.txt --only-scans-dir -o autorecon --single-target --exclude-tags dirbuster
```
- Nmap scan on the pivot machine (MS01)
```sh
nmap -sVC -T4 -v <IP> -oN ms01-initial.txt
nmap -p- -sVC -T4 -v <IP> -oN ms01-full.txt
nmap -sU <IP> -oN ms01-udp.txt
```
### Services
- SNMP
```sh
snmpwalk -c public -v1 -t 10 <IP> <value>
sudo nmap -sU -p161 --script "snmp-brute" <IP>
```
[SNMP Enumeration Tool](https://github.com/trailofbits/onesixtyone)

| Value                  | Description      |
| ---------------------- | ---------------- |
| 1.3.6.1.2.1.25.1.6.0   | System Processes |
| 1.3.6.1.2.1.25.4.2.1.2 | Running Programs |
| 1.3.6.1.2.1.25.4.2.1.4 | Processes Path   |
| 1.3.6.1.2.1.25.2.3.1.4 | Storage Units    |
| 1.3.6.1.2.1.25.6.3.1.2 | Software Name    |
| 1.3.6.1.4.1.77.1.2.25  | User Accounts    |
| 1.3.6.1.2.1.6.13.1.3   | TCP Local Ports  |
- NFS
```sh
showmount -e IP
sudo mount IP:/share /mnt/nfs
sudo umount -f -l /mnt/nfs
```
- SMB
```sh
crackmapexec smb targets.txt
smbmap -H IP
smbclient //IP/share -U "" -L -M
nmap --script "smb-vuln*" -p 139,445 IP
```
- FTP
```sh
nmap --script=ftp-* -p 21 IP
```
- MSSQL
```sh 
nmap -p 1433 -v -sVC -Pn --script="ms-sql*" IP
impacket-mssqlclient user:password@IP
impacket-mssqlclient user:password@IP -windows-auth
impacket-mssqlclient user@IP -H hash 
impacket-mssqlclient user@IP -H hash -windows-auth
```
- RPC
```sh
rpcclient -N IP -U ""

> srvinfo
> enumdomusers
> enumdomgroups
> getdompwinfo
```
### Web Applications
- Directory Enumeration
```sh
feroxbuster -u http://IP:PORT/ -t 10 -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories.txt  -x "txt,html,php,asp,aspx,jsp" -k -r
```
- Check for running software/libraries
```sh
whatweb URL
```
### Windows
```powershell
whoami /priv
```

- Check if we can access SAM and SYSTEM files
  - Windows.old folder
  - Registry Keys
    - \System32\config
    - \System32\config\RegBack
    - \repair
- tasklist /svc
- winpeas / privesccheck
- powershell history
	`AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`
#### Credential Dumping
- secretsdump (preferred)
```sh
impacket-secretsdump USER@IP -hashes :hash
impacket-secretsdump USER@IP 
impacket-secretsdump -sam SAM -system SYSTEM LOCAL
```
- mimikatz
```cmd
privilege::debug
token::elevate 
sekurlsa::logonpasswords
lsadump::secrets
lsadump::sam
```
### Linux
```sh
sudo -l
find / -perm -4000 2>/dev/null
```
- LinPEAS / Linux Smart Enumeration
- Check automated jobs
- Check for interesting files/scripts
- Check for PATH abuse
- Check the Capabilities
### Local Checks for Both Windows and Linux
- Ports listening only in the Machine
