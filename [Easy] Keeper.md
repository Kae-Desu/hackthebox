![Keeper](https://github.com/Kae-Desu/hackthebox/assets/87841341/16af98a0-82e3-4078-9a9f-8cdf841765e9)

check if the machine is online using `curl 10.10.11.227`

![Pasted image 20230922233532](https://github.com/Kae-Desu/hackthebox/assets/87841341/f0c9f738-ae04-4f6d-be73-55e211b26c20)

it seems like to redirect me to `tickets.keeper.htb` with `keeper.htb` as the domain, lets add it to `/etc/hosts`

![Pasted image 20230922233640](https://github.com/Kae-Desu/hackthebox/assets/87841341/99830406-e905-4948-9470-4343f96040de)

after i found the domain name, i run nmap to scan the services running in the web `nmap -sVC -oN nmap keeper.htb`

![Pasted image 20230922233727](https://github.com/Kae-Desu/hackthebox/assets/87841341/3d208dd0-8386-4c04-931d-43ea1ef2c2c2)

after a while, the scan result were complete, it's shown that port 80(http) and 22(ssh) are open, lets try to visit the webpages

![Pasted image 20230922234251](https://github.com/Kae-Desu/hackthebox/assets/87841341/2eaa2145-6f87-4fed-b994-833665ddf987)

on the website its written that it runs a `request tracker version 4.4.4` software, lets try to find a default login credentials.
after a while of searching, i found this official documentation for the 4.4.4 version here https://docs.bestpractical.com/rt/4.4.4/README.html

![Pasted image 20230922234338](https://github.com/Kae-Desu/hackthebox/assets/87841341/90aa4107-ff46-4bd4-86b6-51787b3ead08)

found the default credentials `root:password`, lets try to login with these credentials

![Pasted image 20230922234426](https://github.com/Kae-Desu/hackthebox/assets/87841341/47bf3da4-1beb-4bd2-b11f-7ad0ee914407)

i could login with the default credentials, then i notice there's something interesting in the right pane a `queue list` lets read it, since all the other pane are empty

![Pasted image 20230922235119](https://github.com/Kae-Desu/hackthebox/assets/87841341/52e20667-6c35-41ca-af9f-4f660e5e429c)

there's 1 ticket made by a person named Lise Nørgaard, let's see the profile

![Pasted image 20230922235229](https://github.com/Kae-Desu/hackthebox/assets/87841341/7b08985c-3f75-421d-832d-cbe893c39c3d)

see the `comments about this user` part, it seems like the initial password was changed to `Welcome2023!` and username was already there `lnorgaard`, so i got another login credentials `lnorgaard:Welcom2023!`
let's try to login with this credential on the ssh (i've tried to login to the request tracker, nothing interesting)

![Pasted image 20230922235346](https://github.com/Kae-Desu/hackthebox/assets/87841341/e23b8f5b-87ac-4872-9b47-016dde586215)

machine user acquired

![Pasted image 20230922235413](https://github.com/Kae-Desu/hackthebox/assets/87841341/2cd08c7f-e813-4eae-ae55-d48c6acb6d39)

![Pasted image 20230922235413](https://github.com/Kae-Desu/hackthebox/assets/87841341/df26dc9f-359a-446d-8ac0-a058ca9a0820)

<details>
  <summary>User Flag</summary>
  3211684219e02c6f978ce47671dc7a3c
</details>

<h1>PrivEsc</h1>

for the privesc initial foothold is from a zip on the home folder `RT30000.zip`, move it to some directory such as `/tmp` and try to ectract the zip

![Pasted image 20230923001813](https://github.com/Kae-Desu/hackthebox/assets/87841341/ecbba73b-b733-4572-85f9-442fd00ebc56)

![Pasted image 20230923001846](https://github.com/Kae-Desu/hackthebox/assets/87841341/6720301b-aa04-4392-8e1e-bac65a31e34a)

unzipping the file return 2 file `KeePassDumpFull.dmp` and `passcodes.kbdx`, since this is a KeePass Dump lets try to open it using an online viewer, https://app.keeweb.info/
it ask for a password which i dont have. 

lets see what i could do with the dump. After a short browse i found an interesting article https://www.bleepingcomputer.com/news/security/keepass-exploit-helps-retrieve-cleartext-master-password-fix-coming-soon/
turns out that this is a CVE where an attacker could recover the master password from the dump which potentially could open the .kbdx

POC link: https://github.com/vdohney/keepass-password-dumper

lets move the file to the local machine for further operations

on the server, run python server

![Pasted image 20230923004047](https://github.com/Kae-Desu/hackthebox/assets/87841341/327823a9-985e-4315-b020-3791ffe4c0c5)

on the local machine, run a wget to retrieve the `RT30000.zip` from the server

![Pasted image 20230923140844](https://github.com/Kae-Desu/hackthebox/assets/87841341/7640bce3-0b4e-4679-9ef1-2f645030c7bd)

after the file is on the local machine, extract the zip and run the POC

![Pasted image 20230923141012](https://github.com/Kae-Desu/hackthebox/assets/87841341/ecb90925-448d-4bbd-9448-beb433ee98b6)

base on the above possibilities and extracted password, the password looks likev `[][]dgrød med fløde` missing the first 2 characters, just paste it on the google chrome

![Pasted image 20230923141151](https://github.com/Kae-Desu/hackthebox/assets/87841341/dc39ac65-af48-4f07-a57c-d6b9d6fbf7cb)

and the master password is `rødgrød med fløde`

back to the keepass viewer, lets open the .kbdx now

![Pasted image 20230923141458](https://github.com/Kae-Desu/hackthebox/assets/87841341/a8091335-aba2-46ae-821a-2ec50e90c5e7)

i got a PuTTY key, there should be a tools to convert this to SSH keys. after a quick browse i found an article using a tools called `puttygen` https://superuser.com/questions/232362/how-to-convert-ppk-key-to-openssh-key-under-linux

![Pasted image 20230923142041](https://github.com/Kae-Desu/hackthebox/assets/87841341/5c63fa53-9f96-453f-b30b-01b97e86c212)

after installing the tools, move the key in the .kbdx to a file with .ppk extension

![Pasted image 20230923142420](https://github.com/Kae-Desu/hackthebox/assets/87841341/652edbfc-afea-42f3-8a66-f8c63e8bb89c)

then convert the PuTTY key to SSH key with this command `puttyge puttyKey.ppk -O private-openssh -o id_rsa`

![Pasted image 20230923142559](https://github.com/Kae-Desu/hackthebox/assets/87841341/481b6c10-0825-4f1a-8f86-6dd025a07460)

with the id_rsa file created, try to login as root with the key with this command `ssh -i id_rsa root@keeper.htb`

![Pasted image 20230923142716](https://github.com/Kae-Desu/hackthebox/assets/87841341/46d25853-2417-4f5b-8363-aed8762ded6e)

root acquired

![Pasted image 20230923142745](https://github.com/Kae-Desu/hackthebox/assets/87841341/0f034ad8-995e-4d48-9127-a4768d099788)

<details>
  <summary>Root Flag</summary>
  141282f4204d8f702a522d88cb7c80ca
</details>

![Pasted image 20230923142844](https://github.com/Kae-Desu/hackthebox/assets/87841341/a5507e2a-c19f-4444-9434-506682882317)
