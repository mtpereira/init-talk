init-talk
=========

R42: Talk about init systems.

# init

## Useful URLs

- The Linux Documentation Project: http://tldp.org
- Linux Standard Base - Init: http://refspecs.linuxbase.org/LSB_3.1.1/LSB-Core-generic/LSB-Core-generic/tocsysinit.html

## What is it?

Quote from tldp.org (http://www.tldp.org/LDP/sag/html/init-intro.html):

"[...] the first user level process started by the kernel. init has many important duties, such as starting getty (so that users can log in), implementing run levels, and taking care of orphaned processes."

So, what does this all mean?

- Starts your TTYs shells (F1-F12)
- Starts your services based on their runlevels
- A orphaned process (a process that was dettached from its parent) is automatically "adopted" by init 

## Why should I care?

- System configuration and administration
- Service development
- It's always good to know how your systems work 

## How does it do it?

As soon as the kernel finish its startup, it calls the `/sbin/init`, which will look into `/etc/inittab`.

A inittab entry looks like this:

  `id:runlevels:action:process`

Again, taken (partially) from tldp.org (http://www.tldp.org/LDP/sag/html/config-init.html)

- *id*: This identifies the line in the file. For getty lines, it specifies the terminal it runs on (the characters after /dev/tty in the device file name). For other lines, it doesn't matter (except for length restrictions), but it should be unique.

- *runlevels*: The run levels the line should be considered for. The run levels are given as single digits, without delimiters. (Run levels are described in the next section.)

- *action*: What action should be taken by the line, e.g., respawn to run the command in the next field again, when it exits, or once to run it just once
	man inittab.
	
- *process*: The command to run.

Lets take a look at a real `/etc/inittab`, taken from a Debian system:

	# The default runlevel.
	id:2:initdefault:
	
	# Boot-time system configuration/initialization script.
	# This is run first except when booting in emergency (-b) mode.
	si::sysinit:/etc/init.d/rcS
	
	# What to do in single-user mode.
	~~:S:wait:/sbin/sulogin
	
	# /etc/init.d executes the S and K scripts upon change
	# of runlevel.
	#
	# Runlevel 0 is halt.
	# Runlevel 1 is single-user.
	# Runlevels 2-5 are multi-user.
	# Runlevel 6 is reboot.
	
	l0:0:wait:/etc/init.d/rc 0
	l1:1:wait:/etc/init.d/rc 1
	l2:2:wait:/etc/init.d/rc 2
	l3:3:wait:/etc/init.d/rc 3
	l4:4:wait:/etc/init.d/rc 4
	l5:5:wait:/etc/init.d/rc 5
	l6:6:wait:/etc/init.d/rc 6
	# Normally not reached, but fallthrough in case of emergency.
	z6:6:respawn:/sbin/sulogin
	
	# What to do when CTRL-ALT-DEL is pressed.
	ca:12345:ctrlaltdel:/sbin/shutdown -t1 -a -r now
	
	# Action on special keypress (ALT-UpArrow).
	#kb::kbrequest:/bin/echo "Keyboard Request--edit /etc/inittab to let this work."
	
	# What to do when the power fails/returns.
	pf::powerwait:/etc/init.d/powerfail start
	pn::powerfailnow:/etc/init.d/powerfail now
	po::powerokwait:/etc/init.d/powerfail stop
	
	# /sbin/getty invocations for the runlevels.
	#
	# The "id" field MUST be the same as the last
	# characters of the device (after "tty").
	#
	# Format:
	#  <id>:<runlevels>:<action>:<process>
	#
	# Note that on most Debian systems tty7 is used by the X Window System,
	# so if you want to add more getty's go ahead but skip tty7 if you run X.
	#
	1:2345:respawn:/sbin/getty 38400 tty1
	2:23:respawn:/sbin/getty 38400 tty2
	3:23:respawn:/sbin/getty 38400 tty3
	4:23:respawn:/sbin/getty 38400 tty4
	5:23:respawn:/sbin/getty 38400 tty5
	6:23:respawn:/sbin/getty 38400 tty6
	
	# Example how to put a getty on a serial line (for a terminal)
	#
	#T0:23:respawn:/sbin/getty -L ttyS0 9600 vt100
	#T1:23:respawn:/sbin/getty -L ttyS1 9600 vt100
	
	# Example how to put a getty on a modem line.
	#
	#T3:23:respawn:/sbin/mgetty -x0 -s 57600 ttyS3

## sysvinit

- Used on Debian, Red Hat, openSUSE, etc.
- Run-level based
- Lets take a look at `/etc/init.d/`
- ...and at `/etc/rc?.d/`
- update-rc.d
- Messy, uh? Well, that's why sysv-rc-conf and such tools were created!

### Runlevels

From http://wiki.debian.org/RunLevel

	0 (halt the system) 
	1 (single-user / minimal mode), 
	2 through 5 (multiuser modes), and 
	6 (reboot the system). 

### Init scripts

#### Run time dependencies

	### BEGIN INIT INFO
	# Provides:        ntp
	# Required-Start:  $network $remote_fs $syslog
	# Required-Stop:   $network $remote_fs $syslog
	# Default-Start:   2 3 4 5
	# Default-Stop: 
	# Short-Description: Start NTP daemon
	### END INIT INFO

For more: http://wiki.debian.org/LSBInitScripts and http://refspecs.linuxfoundation.org/LSB\_3.1.0/LSB-Core-generic/LSB-Core-generic/tocsysinit.html

### Actions

- `start`
- `stop`
- `restart`
- `reload`
- `status`
- etc..
