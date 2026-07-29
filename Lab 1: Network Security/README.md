This lab introduces some useful network exploit concepts and tools commonly used to get familiar with attack styles, as well as practice Linux commands from Lab 0 in the context. Also, this lab will give a better understanding about the underlying mechanisms of tools rather than just observing the outcomes of tools.

**1.0. Vulnerable VM (Metasploitable)**  
This is a purposely vulnerable target machine for practicing pen testing techniques.  

**1.1. Port Scan Using Bash**  
Creating a port scanner using a bash script.
'''
#!/bin/bash
if [ $# -ne 1 ]
then
    echo "Usage: `basename $0` {IP address or hostname}"
    exit 1
fi

# define a variable and set it to the value passed as the first argument ($1)
ip_address=$1
# write the current date to the output file
echo `date` >> $ip_address.open_ports

# for loop, where “i” starts at 1 and each time increments up to 65535
for port in {1..65535}
do
    # use a short timeout, and write to the port on the IP address
    timeout 1 echo >/dev/tcp/$ip_address/$port
    # if that succeeded (checks the return value stored in $?)
    if [ $? -eq 0 ]
    then
        # append results to a file named after the date and host
        echo "port $port is open" >> "$ip_address.open_ports"
    fi
done
'''

