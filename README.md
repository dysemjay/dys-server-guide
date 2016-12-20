dys-1.4.1-server-guide
==================================

This is the GitHub repository for a guide to setting up a Dystopia 1.4.1 dedicated server in the Linux environment. In addition to the guide, various supplementary files are included.

The work is licensed under the Creative Commons Attribution 4.0 License, which can be found here: https://creativecommons.org/licenses/by/4.0/ A plaintext copy of the license is also included in this repository, for further reference.

* guide.md  
  A copy of the server setup guide written in the markdown format. Should also be quite readable as plaintext.

* server.cfg  
  An example server.cfg file for SRCDS.

* sysctl.conf  
  An example /etc/sysctl.conf file for the server operating system. Includes TCP/IP stack optimization, and hardening, disabled IPv6, CPU scheduler optimization, decreased VM usage, ICMP related hardening, and a few other security related options. 

  I am aware that I have not described the contents of this file in great detail, or otherwise not included a major explanation of sysctl. My reasoning is that it does not directly relate to SRCDS, and that there is already a wealth of information for it, that I could not think to add to. Please do not simply load this file without understading its effects.

* srcds_run_m  
  A modified version of the standard srcds_run script used to start the server. The primary differences are: the addition of the -debugcmds parameter, a revised help message, and some minor refactoring. -debugcmds allows a newline separated file of commands for GDB to be specified. The default file is debug.cmds; if it is not found, then debugging will be disabled.

  I am aware that the Valve Corporation holds the copyright to the original script; if a legal representative from it requests that I remove this file, I will do so.

* debug.cmds  
  Debug commands file to be used with srcds_run_m. It contains the debug commands that would normally be used by the original srcds_run script.
