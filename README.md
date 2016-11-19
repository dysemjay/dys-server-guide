dys-1_4_1-server-guide
==================================

This is the GitHub repository for a guide to setting up a Dystopia 1.4.1 dedicated server in the Linux environment. In addition to the guide, various supplementary files are included.

* guide.md  
  A copy of the server setup guide written in the markdown format. Should also be quote readable as plaintext.

* server.cfg  
  An example server.cfg file for SRCDS.

* sysctl.conf  
  An example /etc/sysctl.conf file for the server operating system. Includes TCP/IP stack optimization, and hardening, disabled IPv6, CPU scheduler optimization, decreased VM usage, ICMP related hardening, and a few other security related options.

* srcds_run_m  
  A modified version of the standard srcds_run script used to start the server. The primary differences are: the addition of the -debugcmds parameter, a revised help message, and some minor refactoring. -debugcmds allows a newline separated file of commands for GDB to be specified. The default file is debug.cmds; if it is not found, then debugging will be disabled.

* debug.cmds
  Debug commands file to be used with srcds_run_m. It contains the debug commands that would normally be used by the original srcds_run script.
