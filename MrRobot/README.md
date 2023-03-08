Bridged networking mode allows your virtual machine to appear as if it is directly connected to your physical network. This means that your virtual machine will receive its own IP address from your router or DHCP server, and can communicate with other devices on the network just as if it were a physical machine. In other words, the virtual machine will be able to access the internet and other devices on your network, as if it were a separate physical machine.

Host-only networking mode, on the other hand, creates a network that is only accessible between the host machine and the virtual machine. This means that the virtual machine can communicate with the host machine, but cannot access the internet or any other devices on the physical network. In other words, the virtual machine is isolated from the rest of the network and can only communicate with the host machine.

Em primeiro lugar, é necessário encontrar a máquina vulnerável na rede, neste caso dentro do intrevalo 192.168.56.0/24. Para isso, temos várias ferramentas que conseguem extrair essa informação:

sudo netdiscover -r 10.11.12.0/24 

Netdiscover is a network scanning tool that is used to identify hosts on a network. It sends ARP requests and analyzes the responses to create a list of all hosts on the network. Netdiscover can also identify the type of device and its MAC address. It is useful for network administrators and security professionals to monitor and analyze the devices on their network. It is an open source tool and is available for Linux and other Unix-based operating systems.


nmap -sP -vv 10.11.12.0/24

Nmap is a network exploration and security auditing tool that allows you to discover hosts and services on a computer network, and can also be used to identify security vulnerabilities. It uses different techniques, such as port scanning and version detection, to gather information about network hosts and services.

A ping sweep, on the other hand, is a basic network scanning technique that sends ICMP echo requests (pings) to a range of IP addresses in order to determine which hosts are active and responsive. It is a simple and quick way to identify hosts on a network, but does not provide detailed information about services or vulnerabilities.


gobuster dir -u 'http://10.11.12.7/' -w /usr/share/wordlists/dirb/common.txt -x php,txt.html | tee gobuster/common80.log
gobuster dir -u 'http://10.11.12.7/' -w /usr/share/wordlists/dirb/big.txt -x php,txt.html | tee gobuster/big80.log
gobuster dir -u 'http://10.11.12.7/' -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt | tee gobuster/medium80.log

Gobuster is a command-line tool used for directory and file enumeration on web servers. It uses a wordlist and makes requests to the target web server to discover hidden directories and files. Gobuster is often used for website vulnerability scanning, as it can identify unlinked pages, backup files, and other sensitive information that may be present on a web server. Gobuster is an open source tool and is available for Linux, Windows, and macOS. It supports different types of requests, such as GET and POST, and can be configured to use different user agents and cookies.

Analyzing the results, one can right away notice that the 'robots.txt' file is available. This is an easy file to analyze so we can start on that. Checking it we can find two interesting directories that seem like a dictionary and the first flag or key of the challenge. Visiting those pages indeed gives the first flag, but also a list that we should probably download to our system. The easiest way to download the file is by visiting the link, however, getting used to tools like wget or curl can immensely improve our abilities.

wget http://10.11.12.7/fsocity.dic -O fsocity.dic

wget is a tool that allows you to retrieve files from the web using HTTP, HTTPS, and FTP protocols. It supports recursive downloading, meaning that it can download an entire website or directory with just one command. It can also resume interrupted downloads and can be used to download files in the background.

curl http://10.11.12.7/fsocity.dic -o fsocity.dic

curl is a more versatile tool that not only allows you to download files, but also supports transferring data using various protocols, such as FTPS, LDAP, and SCP. It can also be used to send HTTP requests and receive responses, making it useful for testing web services and APIs. Additionally, curl supports a wide range of options and can be used to perform complex operations.


Continuing analyzing our previously acquired results, the admin pages seems quite interesting.
Unfortunately, as it happens quite often, rabbit holes exist and can waste our time tremendously.

Moving forward, there are other interesting endpoints such as /dashboard (among the others that show up).

Yes! A login page. This is good news. Specially when we have a dictionary list that was found in the previous step.
However, we do not know neither the username nor the password, which makes it impossible to login, or is it? 
There is a techin

At this point we have two ways of approaching this problem. By using Burp or wpscan. Burp is much more versatile than wpscan, working for every web application that one faces. Wpscan is focused on the WordPress framework, but is also much easier to use. 

Burp Suite is a popular tool used for web application security testing. It includes a range of features such as intercepting and modifying HTTP requests and responses, scanning for vulnerabilities, and generating reports. It can be used for both manual and automated testing, and is popular among both security professionals and developers. Burp Suite has a user-friendly interface and can be extended with plugins, making it a versatile and powerful tool for web application security testing.

WPScan is a security scanning tool used to enumerate vulnerabilities in WordPress websites. It can be used to identify vulnerabilities in themes, plugins, and WordPress core files. WPScan uses a database of known vulnerabilities and can also perform brute force attacks to crack weak passwords. It is a command-line tool and is available for Linux and other Unix-based operating systems. WPScan is often used by security professionals to identify and mitigate security risks in WordPress websites.

Hydra is a brute-forcing tool used to perform login attacks against remote services. It works by attempting a large number of username and password combinations until it finds the correct one, or until a specified number of attempts is reached. Hydra can be used for a variety of protocols, such as SSH, FTP, HTTP, and more. It is often used by security professionals to test the strength of passwords and to identify vulnerabilities in systems. Brute-forcing is a method of guessing a password by trying many combinations until the correct one is found, and it can be an effective way to compromise a system if the password is weak or easily guessable.

Yes! The user Elliot was found and we have access to the dashboard panel. Here the following step is to either search online for exploit to possibly get a reverse shell, or simply try to find sensitive information such as credentials to possibly access the machine by other means. In this case the second one is not possible, so we must find a way to get a reverse shell.

Note: A reverse shell is a type of shell in which a remote host establishes a connection to a local host or client, allowing the remote host to execute commands on the local host. This is the opposite of a traditional shell, in which the local host initiates the connection to the remote host.

In exploitdb, nothing relevant is really showing up, so Google has to do it. A simple search shows different result seemingly quite promise. In fact, it seems that is possible to get remote code execution through several means. Let's try the simplest one.

Following the steps and overwritting the template.php with our code may result in code exeuction, let's try. For that I usually use the follow code to hopefully get a reverse shell: https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php

Got it! Triggering the code by entering a randon non-existing page makes our code execute and get the shell. Finally, in the context of daemon, we can explore way to privilege escalate our user to root (the entity with all the permissions). 

Looking around is not always easy, and most times quite frustrating, this way, we can use tools like linpeas (for linux) to automate this process. Note that not always this works. Therefore, always run your basic checks and make sure you do not miss anything.

As expected, something was found in MrRobot's directory which seems like a passowrd. In fact, that is not a password, it is an hash.

In computing, a hash (also known as a hash value or hash code) is a fixed-size string of characters that is generated from a variable-size input data. The input data can be anything, such as a text string, a file, or even an entire hard drive.

A hash function takes the input data and generates a unique fixed-size output string, called the hash value, which represents the original data in a compact and easily digestible form. Hash functions are designed to be one-way, meaning that it is very difficult (ideally impossible) to reverse engineer the original data from the hash value.

We cannot input the hash as Mr.Robot's password directly, so we must crack it first. For that, we can use Hashcat or John to bruteforce the cracking process. There are also online resources that can somtimes be faster.

Cracking the password, gives us Mr.Robot password, and we are one step away to get root. For that, run again the checks, or use linpeas and you will find that nmap is set as SUID. Well, even if you do not know how linux permissions work, know that SUID is very good news for the Hacker. Not always as there are programs that one cannot leverage directly such as ping, but other may drop root privs in some cases. Nmap is one of them.

Therefore, visting https://gtfobins.github.io/ which is a databse that contains specific commands to run in order to take advantage of certain conitions, gives us the key to get root. In fact, by running the following command


```
/usr/bin/nmap --interactive
nmap> !sh
```

Finally, root and we pwned the machine.

