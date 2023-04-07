# LAPSDumper

* Dump laps passwords if you have a user account with the rights to dump the laps passwords
* Enumerate users that can read the laps passwords with bloodhound

Usage:

Basic:

`$ python laps.py -u user -p password -d domain.local`

Pass the Hash, specific LDAP server:

`$ python laps.py -u user -p e52cac67419a9a224a3b108f3fa6cb6d:8846f7eaee8fb117ad06bdd830b7586c -d domain.local -l dc01.domain.local`
