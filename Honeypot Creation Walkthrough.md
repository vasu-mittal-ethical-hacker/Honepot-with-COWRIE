I have recently finished creating a honeypot by using COWRIE as a vulnerable SSH server. There are a few prerequisites like Kali linux being installed and updated. Reminder to switch to root user at the beginning until you need to run cowrie so sudo is not reqired. Following are the steps for the project.

Step 1: Updating the System

First, we need to update the Kali Linux System to ensure we have the latest repositories.
`apt update && apt upgrade`
Step 2: Installing Cowrie

Cowrie is not in Kali’s default repository, so we’ll download it from GitHub.

: Clone the Cowrie repository:
`git clone https://github.com/cowrie/cowrie.git /home/kali/Honeypot`
![[clone repo.png]]

: List Contents to double check the clone after navigating to the directory:
`cd /home/kali/Honeypot`
`ls`
![[list out honeypot dir.png]]

Step 3: Setting up the python Virtual Environment

:Installing the Python virtual environment package:
	Cowrie requires Python 3 virtual environment for smooth running.
`apt install python3-venv -y`
: Create a virtual environment within the Honeypot directory:
`python3 -m venv cowrie-env`
![[setup venv.png]]

: Activate the virtual environment:
`source cowrie-env/bin/activate`
![[activate venv.png]]

Step 4: Installing Dependencies

:We should see (cowrie-env) in our command prompt

:Installing all the Python packages that Cowrie needs to function:
`pip install -r requirements.txt`
![[install requirements.png]]

Step 5: Configure Cowrie.

: After  installing all the dependencies Cowrie has to be configured.

: Copying the sample configuration file:

 The file has to be copied from cowrie.cfg.dist to cowrie.cfg because cowrie needs this specific file to run. This file allows us to make many changes like adjusting settings, without the original template file being affected.
 `cp etc/cowrie.cfg.dist etc/cowrie.cfg`
![[copy file.png]]

:Open the configuration file using a text editor like nano or mousepad to adjust the settings if needed.

:Various settings like SSH port, logging preferences, and more can be customized.

:Next step is to change the hostname and SSH settings.

:The main reasons for doing this are:

{a} Deceiving Attackers.

{b} Making it more real.

{c} Better Log analysis.

{d} Avoiding automated detection by automated tools.

Scroll down and change the host name to an inconspicuous name like ssh_srv04.

Ctrl+ W for telnet

:Change enabled = False

:To ENABLED =TRUE

:Save ( Ctrl+O) press enter and exit (Ctrl+ X)

Step 6: Setting up Cowrie to listen on port 22

Cowrie, the fake SSH server (honeypot), listens by default on port **2222**, not the usual SSH port, **22**.

Most real SSH servers use **port 22**, so attackers usually try that port first.

 By making Cowrie look like it’s listening on port 22, we trick attackers into thinking it’s a real server.

But since Cowrie is set to listen on port **2222**, we use a command to **redirect traffic**.

 This command sends anything coming to port **22** over to port **2222** where Cowrie is actually listening.

So, when an attacker connects to port **22**, they’re secretly connecting to Cowrie on port **2222** without realizing it!

 This setup can help capture more realistic attacks.
`iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222`
![[rerouting.png]]

**: iptables:** This is the tool used to create rules for managing network traffic.

**: -t nat:** Specifies the NAT (Network Address Translation) table, used for traffic redirection.

**: -A PREROUTING:** Adds a rule to alter the packet before it reaches its destination.

**: -p tcp:** Indicates that this rule applies to TCP traffic (which SSH uses).

· — dport 22: The port we want to redirect (22, the default SSH port).

**· -j REDIRECT:** Tells the system to forward the traffic.

**· — to-port 2222:** Specifies the new port to which the traffic will be sent.

Redirection has been used to send all connection attempts on port 22 to Cowrie’s actual port 2222 to look like attackers are connecting to real SSH server

For Telnet – *Optional*:
`iptables -t nat -A PREROUTING -p tcp --dport 23 -j REDIRECT --to-port 2223`

Tip: We can use different ports for Testing.

In the configuration, you can choose to set Cowrie to listen on different ports besides the common port 22. This can help make the honeypot look more realistic or test how attackers respond to different setups.

Step 7: Setting up Permissions:

{a}Changing Ownership:
`chown -R kali:kali /home/kali/Honeypot/var/run`
![[changing ownership.png]]

:**/var/run**: Temporary, runtime data, usually PID files and other temporary information needed for the application’s active session.

`chown -R kali:kali /home/kali/Honeypot/var/lib/cowrie`
![[changing own var lib.webp]]

:Setting Permissions:
`chmod -R 755 /home/kali/Honeypot/var/run`

:Verifying Ownership:
`ls -l /home/kali/Honeypot/var/lib/cowrie`
`ls -l /home/kali/Honeypot/var/run`
![[verify own.webp]]

{b}Changing to the non root user
	This is important as the root user cannot run cowrie due to inbuilt restrictions.
`su {INSERT USERNAME}`

Step 8: Starting the Honeypot
:Running Cowrie to start capturing any attempted connections:
`bin/cowrie start`

The Programme has started without showing any errors regarding permissions or multiple instances. The deprecation warnings about TripleDES are not critical issues and can be ignored, as they are related to older encryption algorithms that will eventually be removed in future updates.

: Check Active Processes

ps aux: This command lists all running processes on your system.
`ps aux | grep cowrie`
·       a shows processes for all users, not just the current user.

·       u displays the user who owns each process.

·       x includes processes not attached to a terminal (background processes, like services).

·       | (Pipe): The pipe takes the output of ps aux and sends it to the next command (grep cowrie).

·       grep cowrie: This part searches the output for any lines that contain the word “cowrie.” This filters out the list so that only processes with “cowrie” in their description are shown.

The Cowrie honeypot is running. The output shows a Cowrie process with the twisted command, which is used to start Cowrie, along with the relevant path and parameters.

Viewing the Logs:
`tail -f var/log/cowrie/cowrie.log`

To confirm that Cowrie is capturing activity, we will check the logs for any login attempts or other interactions. Logs are records of everything that happens in Cowrie — like login attempts or commands attackers try. Reading the logs helps us to see what attackers are doing and understand their behavior. The tail command below will show you the latest entries in the logs.

The log output:
![[log output.webp]]

: CowrieSSHFactory starting on 2222: This means Cowrie is ready to accept SSH connections on port 2222.

:HoneyPotTelnetFactory starting on 2223: Cowrie is also set up to accept Telnet connections on port 2223.

*To stop the honey pot simply enter the command* `bin/cowrie stop` *whenever you feel the honeypot doesn't need to run.*

Now since no one will attack this specific honeypot the logs will not show anything of too much interest. However if required an attack can be simulated from a secondary virtual machine.

*Note: the attack is out of scope for this walkthrough.*
