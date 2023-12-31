# console-serial-disk
The CSD (Console Serial Disk) project is intended to allow a "Family of 8" computer
with an upgraded console serial port to be used for both the console and as the
interface to a general purpose OS/8 device handler.  A "Family of 8" computer is
one of the following:  
PDP-5  
PDP-8  
LINC-EIGHT  
PDP-8/s  
PDP-8/i  
PDP-8/l  
PDP-12  
PDP-8/e  
PDP-8/f  
PDP-8/m  
PDP-8/a  
Harris 6100 CPU  
Intersil 6120 CPU  
By upgraded console serial port I mean a higher baud rate (9600 or greater) and
RS-232 (not current loop).  The reason for this is that while extra serial ports
are still available for Omnibus based PDP-8's, extra serial ports on the earlier
machines (Negi and Posibus) are virtually non-existent and quite a bit more 
difficult to make.  An extra serial port is a prerequisite for Kyle Owen's Serial Disk
program.

In its most basic use case all you need is:

1) A PDP-8 with a 9600 baud or faster RS-232 console port.
2) A Raspberry Pi or any Posix compatible server with an RS-232 port.
3) The cables to connect them together.

The PDP-8 console port baud rate does not have to be set to something
really fast, but you will quickly wish that it was.  9600 baud is just
barely tolerable to me.

The most difficult part of using this is getting the serial port on the
server configured to match the PDP-8 and configuring the csd server to
talk to that port.

I have included two csd image files in the tarball so you can just
boot up.  You can also mount a simh RK05 or DECtape image. Kyle Owen's Serial
Disk images are image compatible with simh RK05 and can be mounted as an RK05.
You can only boot from a csd image.

Initial release Version 1.0 (tarball consd.tgz.20230707)

New in Version 1.1 (tarball consd.tgz.20230711)

Fixed the exit bug where the server would crash after saving the csd images.
Added support for simh DECtape images.
Added the ability to configure the stop bits in the server with the other serial port settings.

Unpacking the tarball:

I suggest you create a user account on the server specifically for this
but it can be run from anywhere as long as the server can read and write
the serial port devices.  For this you can place that user in the same
group as the serial device, or change the ownership of the serial device
to the user the server is running under.  I suppose you could also run
the server as superuser although this is not recommended.  Once you have
decided where to place the code you can unpack the tarball with the
command:

tar xzf consd.tgz

This will create a subdirectory consd in the current directory.  If you
have never used tar the x means extract, the z means use gzip compression, and
the f means the tarfile is the filename that follows the f.

You will find that there are several files and subdirectories in the
consd directory.

Anomalies: The anomalous behavior of the BUILD and RESORC commands are discussed.

Cables: A sort of guide to making RS-232 cables.  Also some info about console speed upgrades and conversion to RS-232 for the different machines.

Changelog: Tells what was worked on and when.

Config: More detailed instructions on how to configure everything.

handler: The OS/8 CSD handler subdirectory.

helpers: Small programs that were used during the build process.

Readme: This document.

server: The subdirectory that contains the server.

tsttty: Probably should be under helpers but code to open a Posix serial port for testing.

Upgrades: List of future potential upgrades.


Building the server:

The server sources are found in the subdir consd/server.  You can get
there by changing to it with cd consd/server command.

There are three lines in the csd.c file that may need to be changed to
match your installation.  They specify the serial port device, baud
rate and stop bits on the Unix server.  By default they are set to

#define PDP_SP          "/dev/ttyUSB0"  
#define PDP_BAUD        B9600  
#define PDP_STOP        0  

The first two are fairly self explanatory.  The PDP_STOP setting is somewhat obscure.
If you want 1 stop bit define this to 0.  If you want 2 stop bits then set this to CSTOPB.
Thise are the only choices.  Anything else and you will probably bug out the serial port
that talks to the PDP-8.

After correcting those just type make all.  No reason to install this in
a bin directory but you can if you want to.  To run it type

./csd -D

and it should start running in terminal mode emulating a really dumb ASR
33 or 35 Teletype.  The -D signals Debug mode and it will report what it
is doing.  Each debug line is prefixed by "SERVER:"

Booting:

The boot process is straight forward.  First you toggle the CSD boot loader
into the PDP-8:

0025    Load  
6031    Dep  
5025    Dep  
6036    Dep  
7012    Dep  
7010    Dep  
3001    Dep  
2032    Dep  
5025    Dep  

I suggest you then verify that it is correct before you run it.  The
reason for this is it self modifies and the OS/8 boot process overwrites
the bottom 1k of field zero.  When it is correct you start it with:

0025    Load

Start or Clear and Continue depending on the machine.

Now the PDP-8 is sitting there waiting for the csd server to send it the
boot codes.  You do this one of two ways.  You can start csd with the -B
option or start csd and once it announces it is in terminal mode you can
press shift F12.  If everything is working you will get an OS/8 dot
prompt after a couple of seconds.  OS/8 is now up and running.

In this version of csd the boot image is the file csd0.img which is
provided.  The disk images attached to csd will only be updated when you
leave the server by pressing the F1 key.  This means that all the work
you did in OS/8 has not been saved until you exit.  This was a decision
I made to prevent premature failure of the Micro SD card on the PI.  At
some point I will add an option to allow for immediate updates.

Supported Devices:

The file csd0.img is mapped to SYS: and CSD0: while the file csd1.img is
mapped to CSD1: and DSK: is pointed at that.
The server will also look for csd2.img and csd3.img and if found will be
available as devices CSD2: and CSD3: respectively.  The file rk05.img if
found is mapped to RKA: and RKB: and must be a simh RK05 or one of
Kyle's Serial Disk images.  A DECtape image is supported with the server
file name of tc0X.img and is mapped to the device DTA: in OS/8.

The csd images are maximum size OS/8 file systems.  They are 4095 blocks
long which on a non system platter gives a usable space of 4088 blocks.
This is 1046528 words or 1569752 bytes when packed 3 bytes every 2 words.
With extra code OS/8 could have mapped a device size of 0 as 4096 blocks
but this would have added extra complexity for little gain so I don't
fault the developers for making this decision.

Known to work on:

The server is known to work on a Raspberry Pi 3B+ running the Raspberry
Pi Linux 5.15.32. That is what I used to develop it.  I expect pretty
much any Linux distro will work.  It also runs on a Raspberry Pi 400
using the same microsd card.  It is running on an x86 based laptop running
Ubuntu.

It is expected that the server will also work under most of the BSD Unix variants.
I believe it will also work under Minux version 3.

The handler has been run on:  
A PDP-8/a with an M8315 CPU card and the console port on the M8316 Option 1 card at 9600 baud.  
An SBC6120 at 38400 baud.  
A PDP-8/e with an M8655 console port running at both 9600 and 19200 baud.  
A PDP-8/i.  
A PDP-12 using his Vince's replacement console port FlipChip modules at speeds up to 115.2k baud.  
On simh using socat to connect the two programs.  

It is expected that the handler will run on any of the Family of 8
machines except for the PDP-5.  More on that later.  It will not work
on a VT-78, or any of the DECMates because they don't actually have
a console port.  They do have a serial port which could probably be
used with Kyle's serial disk.

The server is known to not work on a very old version of Cygwin running
under Windows Vista (who would have guessed?)  The termios don't let
you put the serial port into raw mode.

Known Issues:

There is a problem with type ahead during server operations like read or
write.  If you type more than one character while the server is
processing a disk request those characters will not be sent on to the
PDP-8.  I have some ideas on how to fix this but that can wait for a
future version.  Also don't press cursor keys or any keys that don't exist
on a teletype as they can trigger this issue.  It happens because of the
way I chose to decode the function key escape sequences.

Control C is not yet decoded during handler operations.  Yes, this will
get added although it does not seem to be an issue that causes any
problems.

It won't work on a PDP-5.  In the PDP-5 the PC (Program Counter) is
stored in memory location 0 and the boot code currently uses that.  I
have completely re-written the boot code for version 1.2 and did make the new boot
compatible with the PDP-5.  I have since realized
that OS/8 will not run on a PDP-5 because when OS/8 loads its overlay it
overwrites the PC and would crash.  So the boot is fixed but it won't help with
the PDP-5.

Performance:

At 9600 baud a DIR takes about 20 seconds of activity before it starts
displaying.  At 19200 it takes about 10 seconds.  At 38400 it takes about 5 seconds.
At 115200 it takes about 1.6 seconds.  And yes, I know that a DIR is not normally
considered to be any kind of benchmark.  But it makes sense for this because it is
what you notice right away.

If you find any problems or just have questions or comments you can
email me.

doug.ingraham@gmail.com
