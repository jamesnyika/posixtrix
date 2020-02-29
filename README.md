# posixtrix
Common queries I run and wish to remember

## 1. Searching for a specific string in a bunch of files 

````bash 

## Search for the string 3.3.8 recursively(r), with line number(n) and match whole word (w)
## also exclude certain directory names and file extensions 
grep -rnw --exclude-dir={assets} --exclude=*.{js,jar,png,jpeg,jpg,psd,svg,pack} '.' -e "3.3.8"

```` 
 
More examples here
````bash
## This will only search through those files which have .c or .h extensions:

grep --include=\*.{c,h} -rnw '/path/to/somewhere/' -e "pattern"

## This will exclude searching all the files ending with .o extension:

grep --exclude=*.o -rnw '/path/to/somewhere/' -e "pattern"
## For directories it's possible to exclude a particular directory(ies) through --exclude-dir parameter. For example, this will exclude the dirs dir1/, dir2/ and all of them matching *.dst/:

grep --exclude-dir={dir1,dir2,*.dst} -rnw '/path/to/somewhere/' -e "pattern"


````

## 2. Simulating Network Applications with SOCAT

Socat is a unix utility that can take input and push output from given ports. It is amazing.

To link serial port ttyS0 to another serial port:

````bash
socat /dev/ttyS0,raw,echo=0,crnl /dev/ttyS1,raw,echo=0,crnl

````

To get time from time server:

````bash
socat TCP:time.nist.gov:13 -

````

To forward local http port to remote http port:

````bash
socat TCP-LISTEN:80,fork TCP:www.domain.org:80

````

To forward terminal to the serial port COM1:

````bash
socat READLINE,history=$HOME/.cmd_history /dev/ttyS0,raw,echo=0,crnl

````

Simple file-transfer:

On the server-side: socat TCP-LISTEN:port filename
To send file fro the server: socat TCP:hostname:port filename

````bash
socat - TCP4:www.domain.org:80

````

Transfers data between STDIO (-) and a TCP4 connection to port 80 of host www.domain.org. This example results in an interactive connection similar to telnet or netcat. The stdin terminal parameters are not changed, so you may close the relay with ^D or abort it with ^C.

````bash
socat -d -d READLINE,history=$HOME/.http_history \
TCP4:www.domain.org:www,crnl

````

This is similar to the previous example, but you can edit the current line in a bash like manner (READLINE) and use the history file .http_history; socat prints messages about progress (-d -d). The port is specified by service name (www), and correct network line termination characters (crnl) instead of NL are used.

````bash
socat TCP4-LISTEN:www TCP4:www.domain.org:www

````

Installs a simple TCP port forwarder. With TCP4-LISTEN it listens on local port "www" until a connection comes in, accepts it, then connects to the remote host (TCP4) and starts data transfer. It will not accept a second connection.

````bash
socat -d -d -lmlocal2 \
TCP4-LISTEN:80,bind=myaddr1,su=nobody,fork,range=10.0.0.0/8,reuseaddr \
TCP4:www.domain.org:80,bind=myaddr2 

````

TCP port forwarder, each side bound to another local IP address (bind). This example handles an almost arbitrary number of parallel or consecutive connections by forking a new process after each accept(). It provides a little security by sudoing to user nobody after forking; it only permits connections from the private 10 network (range); due to reuseaddr, it allows immediate restart after master processes termination, even if some child sockets are not completely shut down. With -lmlocal2, socat logs to stderr until successfully reaching the accept loop. Further logging is directed to syslog with facility local2.

````bash
socat TCP4-LISTEN:5555,fork,tcpwrap=script \
EXEC:/bin/myscript,chroot=/home/sandbox,su-d=sandbox,pty,stderr

````

A simple server that accepts connections (TCP4-LISTEN) and forks a new child process for each connection; every child acts as single relay. The client must match the rules for daemon process name "script" in /etc/hosts.allow and /etc/hosts.deny, otherwise it is refused access (see "man 5 hosts_access"). For EXECuting the program, the child process chroots to /home/sandbox, sus to user sandbox, and then starts the program /home/sandbox/bin/myscript. Socat and myscript communicate via a pseudo tty (pty); myscripts stderr is redirected to stdout, so its error messages are transferred via socat to the connected client.

````bash
socat EXEC:"mail.sh target@domain.com",fdin=3,fdout=4 \
TCP4:mail.relay.org:25,crnl,bind=alias1.server.org,mss=512

````

mail.sh is a shell script, distributed with socat, that implements a simple SMTP client. It is programmed to "speak" SMTP on its FDs 3 (in) and 4 (out). The fdin and fdout options tell socat to use these FDs for communication with the program. Because mail.sh inherits stdin and stdout while socat does not use them, the script can read a mail body from stdin. Socat makes alias1 your local source address (bind), cares for correct network line termination (crnl) and sends at most 512 data bytes per packet (mss).

````bash
socat - /dev/ttyS0,raw,echo=0,crnl

````

Opens an interactive connection via the serial line, e.g. for talking with a modem. raw and echo set ttyS0's terminal parameters to practicable values, crnl converts to correct newline characters. Consider using READLINE instead of `-'.

````bash
socat UNIX-LISTEN:/tmp/.X11-unix/X1,fork \
SOCKS4:host.victim.org:127.0.0.1:6000,socksuser=nobody,sourceport=20

````

With UNIX-LISTEN, socat opens a listening UNIX domain socket /tmp/.X11-unix/X1. This path corresponds to local XWindow display :1 on your machine, so XWindow client connections to DISPLAY=:1 are accepted. Socat then speaks with the SOCKS4 server host.victim.org that might permit sourceport 20 based connections due to an FTP related weakness in its static IP filters. Socat pretends to be invoked by socksuser nobody, and requests to be connected to loopback port 6000 (only weak sockd configurations will allow this). So we get a connection to the victims XWindow server and, if it does not require MIT cookies or Kerberos authentication, we can start work. Please note that there can only be one connection at a time, because TCP can establish only one session with a given set of addresses and ports.

````bash
socat -u /tmp/readdata,seek-end=0,ignoreeof -

````


This is an example for unidirectional data transfer (-u). Socat transfers data from file /tmp/readdata (implicit address GOPEN), starting at its current end (seek-end=0 lets socat start reading at current end of file; use seek=0 or no seek option to first read the existing data) in a "tail -f" like mode (ignoreeof). The "file" might also be a listening UNIX domain socket (do not use a seek option then).

````bash
(sleep 5; echo PASSWORD; sleep 5; echo ls; sleep 1) |
socat - EXEC:'ssh -l user server',pty,setsid,ctty

````

EXECutes an ssh session to server. Uses a pty for communication between socat and ssh, makes it ssh's controlling tty (ctty), and makes this pty the owner of a new process group (setsid), so ssh accepts the password from socat.

````bash
socat -u TCP4-LISTEN:3334,reuseaddr,fork \
OPEN:/tmp/in.log,creat,append

````


Implements a simple network based message collector. For each client connecting to port 3334, a new child process is generated (option fork). All data sent by the clients are appended to the file /tmp/in.log. If the file does not exist, socat creats it. Option reuseaddr allows immediate restart of the server process.

````bash
socat READLINE,noecho='[Pp]assword:' EXEC:'ftp ftp.server.com',pty,setsid,ctty

````


Wraps a command line history (READLINE) around the EXECuted ftp client utility. This allows editing and reuse of FTP commands for relatively comfortable browsing through the ftp directory hierarchy. The password is echoed! pty is required to have ftp issue a prompt. Nevertheless, there may occur some confusion with the password and FTP prompts.

````bash
socat PTY,link=$HOME/dev/vmodem0,raw,echo=0,waitslave exec:'
````


Generates a pseudo terminal device (PTY) on the client that can be reached under the symbolic link $HOME/dev/vmodem0. An application that expects a serial line or modem can be configured to use $HOME/dev/vmodem0; its traffic will be directed to a modemserver via ssh where another socat instance links it with /dev/ttyS0.

````bash
socat TCP4-LISTEN:2022,reuseaddr,fork \
PROXY:proxy:www.domain.org:22,proxyport=3128,proxyauth=user:pass

````

starts a forwarder that accepts connections on port 2022, and directs them through the proxy daemon listening on port 3128 (proxyport) on host proxy, using the CONNECT method, where they are authenticated as "user" with "pass" (proxyauth). The proxy should establish connections to host www.domain.org on port 22 then.

````bash
echo |socat -u - file:/tmp/bigfile,create,largefile,seek=100000000000

````
creates a 100GB sparse file; this requires a file system type that supports this (ext2, ext3, reiserfs, jfs; not minix, vfat). The operation of writing 1 byte might take long (reiserfs: some minutes; ext2: "no" time), and the resulting file can consume some disk space with just its inodes (reiserfs: 2MB; ext2:16KB).

````bash
   socat tcp-l:7777,reuseaddr,fork system:filan -i 0 -s >&2,nofork

````

listens for incoming TCP connections on port 7777. For each accepted connection, invokes a shell. This shell has its stdin and stdout directly connected to the TCP socket (nofork). The shell starts filan and lets it print the socket addresses to stderr (your terminal window).

````bash
echo -e
````
functions as primitive binary editor: it writes the 4 bytes 000 014 000 000 to the executable /usr/bin/squid at offset 0x00074420 (this is a real world patch to make the squid executable from Cygwin run under Windows, actual per May 2004).

````bash
socat - tcp:www.blackhat.org:31337,readbytes=1000

````

connect to an unknown service and prevent being flooded.


