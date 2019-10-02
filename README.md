# Backdoorme 
 Tools like metasploit are great for exploiting computers, but what happens after you've gained access to a computer? Backdoorme answers that question by unleashing a slew of backdoors to establish persistence over long periods of time.

 Once an SSH connection has been established with the target, Backdoorme's strengths can come to fruition. Unfortunately, Backdoorme is not a tool to gain root access - only keep that access once it has been gained.

 Please only use Backdoorme with explicit permission - please don't hack without asking.
 
## Usage
Backdoorme is split into two parts: backdoors and modules. 

Backdoors are small snippets of code which listen on a port and redirect to an interpreter, like bash. There are many backdoors written in various languages to give variety.

Modules make the backdoors more potent by running them more often, for example, every few minutes or whenever the computer boots. This helps to establish persistence.

Demonstration:

<img src="https://cloud.githubusercontent.com/assets/14065974/21631075/2cd23a98-d205-11e6-811e-c3564b1ca55a.gif" alt="Bash Demonstration" height="500">

### Setup

To start backdoorme, first ensure that you have the required dependencies. 

For Python 3.5+:                                                                                  
```
$ sudo apt-get install python3 python3-pip python3-tk nmap                                 
$ cd backdoorme/
$ virtualenv --python=python3.5 env
$ source env/bin/activate
(env) $ pip install -r requirements.txt
```

For Python 2.7:
```
$ sudo python dependencies.py
```

### Getting Started

Launching backdoorme:
```
$ python master.py
```
To add a target:
``` 
>> addtarget
Target Hostname: 10.1.0.2
Username: victim
Password: password123
 + Target 1 Set!
>>
```
### Backdoors

To use a backdoor, simply run the "use" keyword. 
``` 
>> use shell/metasploit
 + Using current target 1.
 + Using Metasploit backdoor...
(msf) >>
```
From there, you can set options pertinent to the backdoor.  Run either "show options" or "help" to see a list of parameters that can be configured.  To set an option, simply use the "set" keyword. 
```
(msf) >> show options
Backdoor options:

Option		Value		Description		Required
------		-----		-----------		--------
name		initd		name of the backdoor		False
...
(msf) >> set name apache
 + name => apache
(msf) >> show options
Backdoor options:

Option		Value		Description		Required
------		-----		-----------		--------
name		apache		name of the backdoor		False
...
```
As in metasploit, backdoors are organized by category. 
- Auxiliary
  - **keylogger** - Adds a keylogger to the system and gives the option to email results back to you.
  - **simplehttp** - installs python's SimpleHTTP server on the client.
  - **user** - adds a new user to the target.
  - **web** - installs an Apache Server on the client.
- Escalation
  - **setuid** - the SetUID backdoor works by setting the setuid bit on a binary while the user has root acccess, so that when that binary is later run by a user without root access, the binary is executed with root access. By default, this backdoor flips the setuid bit on nano, so that if root access is ever lost, the attacker can SSH back in as an unprivileged user and still be able to run nano (or any chosen binary) as root. ('nano /etc/shadow'). Note that root access is initially required to deploy this escalation backdoor.
   - **shell** - the shell backdoor is a privilege escalation backdoor, similar to (but more specific than) it's SetUID escalation brother. It duplicates the bash shell to a hidden binary, and sets the SUID bit.  Note that root access is initially required to deploy this escalation backdoor. To use, while SSHed in as an unprivileged user, simply run ".bash -p", and you will have root access.
- Shell
  - **bash** - uses a simple bash script to connect to a specific ip and port combination and pipe the output into bash.
  - **bash2** - a slightly different (and more reliable) version of the above bash backdoor which does not prompt for the password on the client-side.
  - **sh** - Similar to the first bash backdoor, but redirects input to /bin/sh.
  - **sh2** - Similar to the second bash backdoor, but redirects input to /bin/sh.
  - **metasploit** - employs msfvenom to create a reverse_tcp binary on the target, then runs the binary to connect to a meterpreter shell.
  - **java** - creates a socket connection using libraries from Java and compiles the backdoor on the target.
  - **ruby** - uses ruby's libraries to create a connection, then redirects to /bin/bash.
  - **netcat** - uses netcat to pipe standard input and output to /bin/sh, giving the user an interactive shell.
  - **netcat_traditional** - utilizes netcat-traditional's -e option to create a reverse shell.
  - **perl** - a script written in perl which redirects output to bash, and renames the process to look less conspicuous.
  - **php** - runs a php backdoor which sends output to bash. It does not automatically install a web server, but instead uses the web module
  - **python** - uses a short python script to perform commands and send output back to the user.
  - **web** - ships a web server to the target, then uploads msfvenom's php reverse_tcp backdoor and connects to the host. Although this is also a php backdoor, it is not the same backdoor as the above php backdoor.
- Access
  - **remove_ssh** - removes the ssh server on the client. Often good to use at the end of a backdoorme session to remove all traces.
  - **ssh_key** - creates RSA key and copies to target for a passwordless ssh connection.
  - **ssh_port** - Adds a new port for ssh.
- Windows
  - **windows** - Uses msfvenom to create a windows backdoor.
  
### Modules
Every backdoor has the ability to have additional modules applied to it to make the backdoor more potent. To add a module, simply use the "add" keyword. 
```
(msf) >> add poison
 + Poison module added
```
Each module has additional parameters that can be customized, and if "help" is rerun, you can see or set any additional options. 
```
(msf) >> help
...
Poison module options:

Option		Value		Description		Required
------		-----		-----------		--------
name	    ls		  name of command to poison		False
location /bin		where to put poisoned files into		False
```
Currently enabled modules include:
 - Poison
  - Performs bin poisoning on the target computer - it compiles an executable to call a system utility and an existing backdoor.
  - For example, if the bin poisoning module is triggered with "ls", it would would compile and move a binary called "ls" that would run both an existing backdoor and the original "ls", thereby tripping a user to run an existing backdoor more frequently. 
 - Cron
  * Adds an existing backdoor to the root user's crontab to run with a given frequency.  
 - Web
  - Sets up a web server and places a web page which triggers the backdoor.
  - Simply visit the site with your listener open and the backdoor will begin.
 - User
  - Adds a new user to the target.
 - Startup
  - Allows for backdoors to be spawned with the bashrc and init files.
 - Whitelist
  - Whitelists an IP so that only that IP can connect to the backdoor.

### Targets
Backdoorme supports multiple different targets concurrently, organized by number when entered. The core maintains one "current" target, to which any new backdoors will default. To switch targets manually, simply add the target number after the command: "use metasploit 2" will prepare the metasploit backdoor against the second target. Run "list" to see the list of current targets, whether a connection is open or closed, and what backdoors & modules are available. 

## Contributing
Backdoorme is still very much in its infancy! Feel free to contribute to the project - simply fork it, make your changes, and issue a pull request. Have an idea for a killer backdoor, or something we could improve? Make an issue and we'll add it ASAP! Please email us at backdoormegit@gmail.com with any questions.

If you wish to add your own backdoor, follow the directions given in the backdoorme/backdoors/template.py file.

If you wish to add your own module, follow the directions given in the backdoorme/modules/template.py file.

# Flow Here 
https://github.com/alex14324
