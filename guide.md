Dystopia 1.4.1 Server Setup Guide
==================================

1.  [Introduction](#introduction)
2.  [Preparation](#preparation)
3.  [Getting SteamCMD](#getting-steamcmd)
4.  [Server Installation](#server-installation)
5.  [server.cfg (The primary configuration file)](#servercfg-the-primary-configuration-file)
6.  [Map Management](#map-management)
7.  [Starting the Server](#starting-the-server)
8.  [Working From the Server Console/Common Commands](#working-from-the-server-consolecommon-commands)
9.  [Banning Players](#banning-players)
10. [Rate Settings](#rate-settings)
11. [Firewall Hardening Tips](#firewall-hardening-tips)
12. [Miscellaneous Optimizations](#miscellaneous-optimizations)
13. [Known Bugs](#known-bugs)
14. [Useful Links](#useful-links)
15. [Credits](#credits)
16. [Changelog](#changelog)

Introduction
----------------------------------

This guide is meant to provide instructions for installing and operating a dedicated server for Dystopia 1.4.1 in a Linux environment. Although these instructions are written specifically for, and tested in Linux, many of them should also be applicable in Windows. Many of the steps are also likely to remain the same in the event that Dystopia is updated again. The following is a link to sample configuration, and other supplementary files for this tutorial: https://github.com/dysemjay/dys-server-guide At this point, the primary discussion for the guide is held here: http://steamcommunity.com/app/17580/discussions/0/152390014785349924/

A specific notation will be used for specifying variable options for commands in this guide.  
'<' and '>' will surround the description that is necessary for a command. Example: `cd <Name of directory>`  
'[' and ']' will surround a specific option that will alter the functionality of a command. Example: `app_update <number of Steam App> [validate]`

Next is a list of platforms the guide has currently been tested on:

* Slackware 14.2 - 3.13.0-32-generic - #57-Ubuntu SMP Tue Jul 15 03:51:08 UTC 2014 - x86_64  
  This was actually my desktop distribution. Note that I used the multilib packages maintained by Alien BOB; more information can be found about them here: https://docs.slackware.com/slackware:multilib


Preparation
----------------------------------

1. Create a new user to install and operate the server. This will help to limit the amount of damage that can be done, should SRCDS be compromised. Running SRCDS as root, or a superuser is very inadvisable. In fact, the start script for SRCDS will likely display a warning message if run by root.

   With root privileges, or sudo: `adduser <name of user>`

2. You should have been prompted for the password, if not, you should definitely set it manually.

   With root privileges, or sudo: `passwd <name of user>`

3. Now we can log in as that new user, and change to the home directory.

        su <name of user>
        cd ~

4. You might want to create a new directory for SteamCMD to go in, but this is optional.

        mkdir <name of SteamCMD directory>
        cd <name of SteamCMD directory>

5. Create a directory for the server itself to be installed to, but don't enter it yet.

        mkdir <Name of server directory>

6. If you are running a 64 bit system, you will need multilib support to run both SteamCMD and SRCDS. For some, this will ship by default with the operating system. The description for each tested OS in the introduction should contain more information regarding multilib support.


Getting SteamCMD
----------------------------------

1. If you are using a Debian, RedHat/CentOS, or Arch based distribution, you may simply get SteamCMD and its dependencies from a repository. SteamCMD should be installed to /usr/games/steamcmd, and symbolic linked to the current directory. Otherwise, continue to step 2 for manual installation.

   If necessary, change to an account with the appropriate privileges: `su <name of account>`

   * Debian: `sudo apt-get install steamcmd`
   * RedHat/CentOS: `yum install steamcmd`
   * Arch: `pacman install steamcmd`

        ln -s /usr/games/steamcmd steamcmd

   If we changed to another account, log out of it: `exit`

2. Please check the tested operating systems section of the introduction for more information on special dependencies. Instructions to install dependencies for the common Debian, RedHat/CentOS, or Arch based systems will be detailed here.

   If necessary, change to an account with the appropriate privileges: `su <name of account>`

   * Debian: `sudo apt-get install lib32gcc1`
   * RedHat/CentOS: `yum install glibc libstdc++`
   * Arch: `yum install glibc.i686 libstdc++.i686`

   If we changed to another account, log out of it: `exit`

3. Now we download the actual SteamCMD package, and extract it.

        wget https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz
        tar zxvf steamcmd_linux.tar.gz


Server Installation
----------------------------------

1. Now we start the newly installed SteamCMD. It should automatically update itself if necessary. We should come to a prompt resembling `Steam>`

  * If installed from repository: `./steamcmd`  
  * If installed manually: `./steamcmd.sh`

2. Install Source SDK 2013 Server to the server directory we previously created. The validate option is used to verify the integrity of the downloaded files; note that this includes configuration files. THIS OPTION WILL OVERWRITE YOUR CUSTOMIZED CONFIGURATION FILES. So you should back them up before using it.

        login anonymous
        force_install_dir <Name of server directory>
        app_update 244310 [validate]

3. Now that Source SDK 2013 Server has been installed, we must exit SteamCMD, then start it again to install the Dystopia game content to the same directory, from a real Steam account. At this point, it is necessary to specify the "Previous" beta option, as the most recent version was broken by an update to Source SDK. You can find more information about this here: http://steamcommunity.com/app/17580/discussions/0/357286119111069868/

        quit
        `./steamcmd` OR `./steamcmd.sh`
        login <username of chosen Steam account>
        force_install_dir <Name of server directory>
        app_update 17580 -beta Previous [validate]


server.cfg (The primary configuration file)
----------------------------------

server.cfg is the primary configuration file for SRCDS, and will control most of it's function. This section will describe common variables, and detail a way of managing configurations. Comments in any config file are preceded by "//" (Similar to C++ style comments).

The following is a list of common ConVars (console variables), and a description (Note there are certain ConVars that will be covered in specialized sections):

* `exec <Name of file in <Name of server directory>/dystopia/cfg>`  
  Executes a configuration file (in a similar way to server.cfg). This can be useful for separating configuration in to multiple files.

* `hostname "<Name of server>"` Default: ""  
  Sets the name of the server. If set to "", then it will be listed as "Dystopia" in the server browser. The maximum length is uncertain, however, for practical purposes, it should probably be no greater than 63 characters, as successive characters will be truncated when it is listed in the server browser.

* `sv_pure <-1, 0, 1, or 2>` Default: 0  
  Determines to what extent a client will be permitted to use files that differ from those included with the vanilla game. The behavior of the ConVar seems bugged in many ways. It appears that values 1, and 2 cause equivalent behavior to 0: from a client perspective, it is possible to use non-whitelisted custom content, and sv_pure will actually be reported as 0. More information regarding its behavior can be found in the known bugs section.

  -1: No restrictions on clientside files. Apply no rules.  
   0: Minimal restriction on clientside files. Apply rules in `<Name of server directory>/dystopia/cfg/trusted_keys_base.txt`.  
   1: Strict restriction on clientside files. Apply rules in `<Name of server directory>/dystopia/cfg/pure_server_full.txt`, `<Name of server directory>/dystopia/cfg/trusted_keys_base.txt`, and in `<Name of server directory>/dystopia/pure_server_whitelist.txt`. This does not appear to work.  
   2: Strictest restriction on clientside files. Apply rules in `<Name of server directory>/dystopia/cfg/pure_server_full.txt` and in `<Name of server directory>/dystopia/cfg/trusted_keys_base.txt`. This does not appear to work.

* `sv_lan <0 or 1>` Default: 0  
  If set to 1, the server will be run in LAN mode, which should prevent users from joining, or seeing the server if they are not in your local area network. However, that feature appears to be bugged; please check the known bugs section for more information. This will also prevent banning of players by SteamID, and disable VAC security. Note that if you intend to set this to 1, then it should also be set from the command prompt. Otherwise, it will not take effect until the next map load, because the default value of 0 will have been processed before server.cfg is first loaded.

* `rcon_password "<password>"` Default: ""  
  Sets the password for RCON authentication.

* `sv_rcon_min_failuretime <decimal value>` Default: 30 Min: 1  
  Time in seconds to track failed RCON authentications for sv_rcon_minfailures.

* `sv_rcon_minfailures <number>` Default: 5 Min: 1 Max: 20  
  Number of times an IP address can fail RCON authentication within the period specified by sv_rcon_min_failuretime before the next failed attempt results in a ban.

* `sv_rcon_maxfailures <number>` Default: 10 Min: 1 Max: 20  
  Number of times an IP address can fail RCON authentication before the next failed attempt will result in a ban. The failure counter for this ConVar is persistent until server shutdown, and remains after a player is banned. (So if a player waits out the ban period, then performs another failed RCON authentication, they are automatically banned again)

* `sv_rcon_banpenalty <decimal value>` Default: 0 Min: 0  
  Time in minutes to ban IP addresses that fail RCON authentication.

* `sv_rcon_whitelist_address "<IP Address>"`  
  Specifies a SINGLE IP address from which failed RCON authentications can never result in a ban.

* `sv_password "<password>"` Default: ""  
  Sets the password to join the server.

* `sv_allowdownload <0 or 1>` Default: 1  
  Allows downloading of content (such as maps) directly from the server if set to 1. The rate for this seems to be capped at 25 kB/s or sv_maxrate, if it is nonzero; whichever is lower. If sv_downloadurl is set, then the source specified by it will take precedence.

* `sv_allowupload <0 or 1>` Default: 1  
  Allows uploading of content from the client, to the server if set to 1. Primarily used for sprays.

* `sv_downloadurl "<url>"` Default: ""  
  Download link for separate server to retrieve content from. Should work for HTTP, and HTTPS (Note that in testing, problems were experienced with a self signed certificate.). Clients should also follow HTTP (status code 3xx) redirects.

* `log on`  
  Enables server logs.

* `sv_logbans <0 or 1>` Default: 0  
  Enables logging of bans from the server if set to 1.

* `sv_logecho <0 or 1>` Default: 1  
  If set to 1, then whenever information is written to server logs, it will also be printed to the console. This can potentially make the console output quite messy.

* `sv_logfile <0 or 1>` Default: 1  
  Enables logging to files if set to 1. The other case where server logs are written, is to a remote server.

* `sv_log_onefile <0 or 1>`  Default: 0  
  All server logs are written to a single file if set to 1.

* `sv_rcon_log <0 or 1>` Default: 1  
  Enable logging of RCON connections if set to 1.

* `sv_logsdir <Name of directory in <Name of server directory>/dystopia>` Default: "logs"  
  Set directory for log files to be stored to.

* `dys_log_stat_events <0 or 1>` Default: 1  
  Enable logging of events pertaining to Dystopia Stats, if set to 1.

* `con_logfile <Name of file in <Name of server directory>>` Default: ""  
  If a file name is specified, console output will be logged to that file.

* `con_timestamp <0 or 1>` Default: 0  
  Information in the file specified by con_logfile will be prefixed with timestamps.

* `logaddress_add <ip:port>`  
  Send logging information to the server specified by ip and port.

* `sv_cheats <0 or 1>` Default: 0  
  Enables various cheat commands, and should also allow for a more detailed net_graph to be shown to a client, if set to 1. Can be useful for server debugging.

* `maxplayers <number>` Default: 16  
  Maximum number of players on the server. This number is usually 16 for Dystopia.

* `mp_autoteambalance <0 or 1>` Default: 1  
  If set to 1, then the server can automatically balance the teams. This will occur if one team has two or more players than the other. The player that most recently joined the offending team will be moved to the other upon death. It also appears that players that were moved forcibly within the last 300 seconds will automatically take lower priority than those that were not. This detail was gleaned from the following Source SDK 2013 functions: https://github.com/ValveSoftware/source-sdk-2013/blob/0d8dceea4310fde5706b3ce1c70609d72a38efdf/mp/src/game/shared/teamplayroundbased_gamerules.cpp#L2992, and https://github.com/ValveSoftware/source-sdk-2013/blob/0d8dceea4310fde5706b3ce1c70609d72a38efdf/mp/src/game/server/basemultiplayerplayer.cpp#L205

* `mp_rounds <number>` Default: 2  
  Number of rounds before a map change. This number is usually 2 for Dystopia. Setting it to 0 should be equivalent to setting it to 1. (Only 1 round before the map changes)

* `mp_startdelay <number>` Default: 40  
  Specifies the delay before players spawn, after a map has loaded. A common purpose for this is to provide time for all players to have connected before a round will begin.

* `mp_timelimit <number>` Default: 0  
  Timelimit per round in minutes. This number is usually 20 for Dystopia. Setting this to 0 will also cause the timelimit to be 20.

* `mp_stopwatch <0 or 1>` Default: 0  
  If set to 1, then there will be twice the number of rounds specified by mp_rounds per map. When a round ends, the timelimit of the next round is set to the time taken to complete the previous. This setting is usually used in pick up games, and other competitive settings.

* `mp_scalespawntime <0 or 1>` Default: 1  
  This is thought to have no effect since Dystopia 1.4. The primary reason I include it is that it is set to 1 by default, and its behavior when set to 0 is best understood.

* `mp_friendlyfire <0 or 1>` Default: 1  
  Enables friendly fire, if set to 1. Friendly fire is usually on for Dystopia.

* `mp_ceasefire <0 or 1>` Default: 0  
  Prevents players from attacking, if set to 1.

* `dys_melee_only <0 or 1>` Default: 0  
  If set to 1, then prevent players from firing, and make melee do no damage to health (it can still damage armor however).

* `mp_chattime <Decimal value>` Default: 30 Min: 1.000000 Max: 120.000000  
  Specifies a waiting period in seconds after a round ends, in which players can chat.

* `sv_alltalk <0 or 1>` Default: 0  
  If set to 1, then voice chat can be heard by all players, otherwise voice chat can only be heard by team mates.

* `mp_punishvotes <number>` Default: 5  
  Number of punish votes by a teamkilled players, before the teamkiller is banned from the server. Disabled if set to 0.

* `mp_punishbantime <number>` Default: 30  
  Time in minutes for a player to be banned by mp_punishvotes. They will only be kicked from the server if this is set to 0.

* `mp_allowvoting <0 or 1>` Default: 1  
  Players may use callvote commands if this is set to 1.

* `mp_allowvoting_map <0 or 1>` Default: 1  
  If set to 1, players may callvote maps.

* `mp_allowvoting_balanceteams <0 or 1>` Default: 1  
  If set to 1, players may callvote team balance. The balance performed by this callvote should arrange players so that the difference between the sum of their ranks for each team is as low as possible.

* `mp_allowvoting_swapteams <0 or 1>` Default: 1  
  If set to 1, players may callvote to swap their teams.

* `mp_allowvoting_maprestart <0 or 1>` Default: 1  
  If set to 1, players may callvote map restart.

* `mp_allowvoting_kick <0 or 1>` Default: 1  
  If set to 1, players may callvote to kick, or ban a player.

* `mp_allowvoting_forcespec <0 or 1>` Default: 1  
  If set to 1, players may callvote to force a player onto the spectators team.

* `mp_allowvoting_meleeonly <0 or 1>` Default: 0  
  If set to 1, players may callvote to toggle the dys_melee_only ConVar.

* `mp_allowvoting_stopwatch <0 or 1>` Default: 1  
  If set to 1, players may callvote to toggle the mp_stopwatch ConVar.

* `mp_votepct <number>` Default: 51  
  Specifies the percentage of players necessary for a callvote to pass.

* `mp_votekick_bantime <number>` Default: 30  
  Specifies time in minutes for player to be banned by callvote ban. If set to 0, then the player will only be kicked from the server.

* `sv_idlekick <0, 1, or 2>` Default: 1  
  Determines whether players should be kicked for idling.  
  0: Never kick players for idling.  
  1: Kick non-spectator players for idling.  
  2: Kick any player for idling.

* `sv_idlekick_timer <decimal value>` Default: 300 Min: 60  
  Time in seconds a player can idle before they are kicked.

Finally, I will describe a method that I frequently use for managing server configuration files for different purposes. Let's say that I have the configuration files: standard.cfg, and pug.cfg for public games, and pickup games respectively. In order to use one, it's contents would be written to server.cfg with a simple `cat <filename> > server.cfg`, then the server would be restarted with `map <name of map>` from the server console, to completely load the new settings.


Map Management
----------------------------------

This section will describe the method of installing, and managing maps for the server.

* The map files should be stored in `<Name of server directory>/dystopia/maps`, and should have the extension ".bsp". Note that serverside map files CANNOT be compressed, unlike the clientside copies.

* It has been confirmed by a Dystopia developer that `<Name of server directory>/dystopia>/maplist.txt` essentially has no effect. At this point, it is basically unremoved content from a previous version of the game. It can likely be ignored, or deleted if present. Available maps will be directly determined from the contents of `<Name of server directory>/dystopia>/maps`.

* Finally, you must specify a mapcycle file. This is done by setting the "mapcyclefile" ConVar; by default it is set to "mapcycle.txt". The mapcycle file controls the order in which maps are loaded by the server. If the mapcycle file does not exist, or is empty, then whatever map is currently loaded, will simply be reloaded. Names of maps are stored in a similar fashion to the maplist file, and are loaded in descending order from top to bottom. If the end of the file is reached, or the current map is not included in the file, then the first map is chosen again. Finally, if a map listed in the file cannot be found, then it will be skipped.


Starting the Server
----------------------------------

This section will describe a procedure for starting the server, and some command line parameters that may be passed during this process. SRCDS should be started with a shell script called "srcds_run", located in `<Name of server directory>`. In my opinion, the original script has some features that do not serve much of a purpose, or do not work properly. A revised version that I wrote can be found here: https://github.com/dysemjay/dys-server-guide/blob/master/srcds_run_m The primary differences are: reliance on the -debugcmds parameter to pass arguments to GDB, a revised help message, and a little refactoring. More information can be found in the readme of the repository.

When SRCDS is started, the server command line interface will opened. This can pose a problem when attempting to switch between the server console, and the shell prompt. A common method of circumventing this is to start SRCDS with the screen command. The session can be detached from (to return to the shell prompt) by pressing Ctrl-a, then Ctrl-d. The session can be reattached with the command `screen -r`

If your server is behind NAT (Such as in the case of a SOHO network with one public IP address, and an internal network in a private addressing scheme), then you will probably want to bind SRCDS to the private address of the system, the forward the port it is listening on from your router.

* Descriptions for some parameters can be found by running `./srcds_run -help` in `<Name of server directory>`.

* ConVars can also be specified from the command line by preceding them with a '+' instead of a '-'. Note that ConVars specified in server.cfg will take precedence over these.

* If a map is not specified with `+map <name of map>`, then the server will start without having loaded one. In this state, it will not be listed in the server browser, cannot be joined by a player, and will not have loaded server.cfg.

In my experience, the commonly used ConVars are:

* `-game <Name of game directory>`  
  In this case, it should always be "dystopia"

* `-ip <IP Address>`  
  This is the IP address for SRCDS to bind to.

* `-port <number>` Default: 27015  
  This is the port number for SRCDS to bind to. TCP is used for RCON, UDP is used for general game traffic. Note that SRCDS can also bind to different ports; this will be further discussed in iptables Hardening Tips.

* `-debug`  
  This will cause GDB to be run to collect some debugging information in the event of a crash. It actually will not slow down the server itself, or otherwise cause a change in its behavior.

* `-debuglog <Name of file in <Name of server directory>>`  
  This specifies the file for debugging output generated by the -debug parameter to be stored in.

Finally, an example command to start SRCDS is:

    screen ./srcds_run -game dystopia -ip 192.168.1.103 -port 27025 -debug +map dys_broadcast


Working From the Server Console/Common Commands
----------------------------------

The purpose of this section is to explain the layout and behavior of the SRCDS command line interface, and some commonly issued commands. The interface is quite simple and minimalist; output is echoed to the terminal, as is buffered input. Input is submitted by a linebreak (or pressing enter). Due to this behavior, the display of buffered input can be interrupted by other information, which can make the process of typing out a command tedious, and error prone. As far as I know, there is no elegant way of suppressing, or filtering output. Please inform me if you happen to know a way of doing this.

ConVars can be set from the command line interface in a similar manner to in server.cfg. These will take precedence over those passed as arguments when starting the server, but not those explicitly set in server.cfg. So if the ConVar is set in server.cfg, your changes will be lost when it is reloaded (Such as in the case of a map change). If the ConVar is typed without a specified value, then the current value, along with a description of the ConVar will be printed.

The following is a list of common commands, and their descriptions (Again, certain commands may be listed in a more specific section):

* `find <string>`  
  Search for commands and ConVars which contain `<string>` in their name or description, then list them.

* `cvarlist`  
  Dumps a list of all console commands and variables. Can also be used like `cvarlist log <Name of file in <Name of server directory>/dystopia>` to dump them to a file, in the CSV format.

* `status`  
  List general information about the server, including hostname, version, IP addressing, SteamID, current map, and players.

* `stats`  
  List general performance information about the server, including CPU usage, bandwidth usage, uptime, number of map changes, FPS, current number of players, and total number of connections.

* `maps <substring>`  
  Displays available maps which contain `<substring>` in their name. Entering `maps *` will cause all maps to be displayed.

* `map <name of map>`  
  Load a map of a certain name (Not including the filename extension). Note that this should also effectively restart the server, and reload configuration. It appears that the command will also load a map if the argument is a substring of its name. If there are multiple names that contain the substring, then the one that comes first, in alphabetical order will take precedence.

* `map_restart`  
  Restart the current map, note that this does not restart the server in the same way that `map <name of current map>` would.

* `balanceteams`  
  Balances teams through a similar algorithm to that used by `callvote balanceteams`.

* `swap_teams`  
  Swaps player teams. The effect is equivalent to that performed by callvote swap_teams.

* `kick <Name of player>`  
  Kicks a player from the server by name. Note that the name matching is case insensitive.

* `kickid <userid>`  
  Kick a player from the server by UserID.

* `say <string>`  
  Causes a message to be printed to global chat in game. In this case, the source will be listed as "Console". Note that, unlike the client side say command, the server side say command, will include quotation marks in the printed message. 


Banning Players
----------------------------------

The purpose of this section is to describe procedures for managing player bans from the command line, and in files. There are two principle ways a player can be banned from the server: by IP address, and by Steam ID. The respective .cfg files associated with these methods are: banned_ip.cfg, and banned_user.cfg. (Both are located in `<Name of server directory>/dystopia/cfg` and can be made if they do not already exist.)

Note that for banning IP addresses, you might also consider completely blocking them at the firewall level. On short notice, this can be performed by the command: `iptables -I INPUT -s <IP Address> -j DROP`. Replacing '-I' with '-D' will remove the IP Address ban. The argument of the command, with '-I' replaced by '-A', can also be placed at the beginning of the INPUT chain in the rules file. One caveat about IP address banning is that many IP addresses are dynamically allocated, and are quite subject to change as a result.

* banned_ip.cfg should essentially be populated by a series of addip commands.
* banned_user.cfg should essentially be populated by a series of banid commands.
* Without being loaded from a file, these commands are only effective until the server shuts down.

Finally, the following should be added to your server.cfg to ensure that both of these files are loaded:

```
exec banned_ip.cfg
exec banned_user.cfg
```

The following is a list of commands associated with banning players:

* `banid <time> <SteamID or UserID of player> [kick]`  
  Ban the account specified a given SteamID or UserID for `<time>` minutes. If the kick argument is added, the account will also be kicked if connected. This will not work if sv_lan is set to 1.

* `addip <time> <IP Address of player>`  
  Ban the IP address for `<time>` minutes.

* `removeip <IP Address of player>`  
  Remove an IP address from the ban list.

* `removeid <Steam ID of player>`  
  Remove a Steam ID from the ban list.

* `writeip`  
  Write the currently, permanently banned IP addresses to banned_ip.cfg. Note that this will completely overwrite the
file.

* `writeid`  
  Write the currently banned Steam IDs to banned_user.cfg. Note that this will
completely overwrite the file.

* `listid`  
  List currently banned Steam IDs.

* `listip`  
  List currently banned IP addresses.


Rate Settings
----------------------------------

The purpose of this section is to list ConVars associated with rate settings, and describe how they might be configured for your system and connection.

* `sv_mincmdrate <number>` Default: 10  
  Sets the minimum number of command packets that can be sent to the server each second. 0 is unlimited.

* `sv_maxcmdrate <number>` Default: 66  
  Sets the maximum number of command packets that can be sent to the server each second. 0 is unlimited.

* `sv_minupdaterate <number>` Default: 10  
  Sets the minimum number of update packets that can be sent from the server each second. 0 is unlimited.

* `sv_maxupdaterate <number>` Default: 66  
  Sets the maximum number of update packets that can be sent from the server each second. 0 is unlimited.

* `sv_client_cmdrate_difference <number>` Default: 20  
  Specifies the number of packets the difference between a client's command rate and update rate will fall within.

* `sv_minrate <number>` Default: 3500  
  Minimum upload bandwidth usage for each connection in bytes. 0 is unlimited

* `sv_maxrate <number>` Default: 0  
  Maximum upload bandwidth usage for each connection in bytes. 0 is unlimited

So note that these ConVars are closely related to the clientside ConVars: cl_cmdrate, cl_updaterate, and rate. They allow the server to negotiate the parameters of a player's connection. The updaterate, and cmdrate should not exceed the server's tickrate, so the default maximum should be sufficient in most situations. An important consideration when setting the minimum is that many players will just use the default rate settings. When setting sv_client_cmdrate_difference, bear in mind that a common player's upload bandwidth might be lower than their download bandwidth, so allowing for there to be some difference between their command rate and update rate can be beneficial.

Finally, we will discuss a method of calculating an appropriate value for sv_maxrate, based on the number of slots, and upload bandwidth. Suppose the server has 16 slots, and an upload bandwidth of 5 Mbit/s (This should be common for a consumer cable connection, remember that the bandwidth might also be consumed by other traffic).

1. Convert 5 Mbit/s to B/s.  
   5 Mbit = 5,000,000 bits  
   5,000,000 bits / 8 = 625000 bytes

2. Divide this by the number of slots.  
   625000 / 16 = ~39062 bytes

So we have calculated the maximum bandwidth per connection to be 39062 bytes. This a value we can set sv_maxrate to. It might be preferable to estimate below the precise number, given that the ISP may not guarantee consistent bandwidth.


Firewall Hardening Tips
----------------------------------

The purpose of this section is to describe various procedures for hardening the Linux kernel firewall for traffic associated with the Dystopia Dedicated server. Though this section specifically describes configuration of the Linux Kernel firewall, through iptables, much of that information may be applicable to other management utilities.

In order to illustrate how stateful packet inspection in the Linux kernel works, I will briefly list the common connection states:

* NEW - This is a connection which is newly initiated, or otherwise has not seen packets to and from the destination.
* RELATED - This is a new connection associated with an already existing connection. A practical example of this might be the data transfer for FTP on port 20, associated with a connection on port 21.
* ESTABLISHED - This connection has seen packets to and from the destination.
* INVALID - This is not a known connection. Usually used when one does not match the other states.

The server (you can normally recognize it by the process associated with the file srcds_linux) will communicate on a number of ports:

* `<User specified listening port>` - TCP, UDP - By default, this is 27015. This port listens with TCP for RCON, and with UDP for general game traffic. TCP can be blocked completely if you do not intend to use the standard RCON, otherwise the port should be open to unsolicited connections for both TCP, and UDP.

* `<32768 - 61000?> to 192.198.217.32:27055` - TCP - This connection is associated with Dystopia Stats. It is initiated by the server, so it is only necessary to allow outgoing traffic, and established incoming traffic for it. To the best of my knowledge, the outbound port should simply be randomly chosen from the standard ephemeral port range, like in a typical TCP connection. (This seems to be a general consensus with the developers)

* `26901 to <IP Address registered to Valve corporation>:<27016 - 27021?>` - UDP - This connection is most likely associated with VAC, and the Steam server browser. It is initiated by the server, so it is only necessary to allow outgoing traffic, and established incoming traffic for it. My description of the port range used by this connection is based on observation, rather than formal documentation. Please feel free to offer correction or clarification.

So, the primary considerations for firewall configuration are:

* SRCDS listens for unsolicited connections for TCP, and UDP on a user specified port.
* SRCDS may initiate connections itself.
* It is possible for incoming packets to be flooded to the listening port.
* SRCDS will only accept packets of a minimum and maximum size for game traffic.
* The maximum packet size can be calculated from ConVars (The default value should be 2521 bytes): net_maxroutable + net_splitrate * net_maxfragments
* The minimum packet size should be 32 bytes

With these considerations, this is an example iptables chain to be included in a rules file:

```
#SRCDS CHAIN

-N SRCDS

#block undersized packets to srcds
-A SRCDS -m length --length 0:32 -j LOG --log-prefix "SRCDS-XSQUERY " --log-ip-options -m limit --limit 1/m --limit-burst 1
-A SRCDS -m length --length 0:32 -j DROP

#block oversized packets to srcds
-A SRCDS -m length --length 2521:65535 -j LOG --log-prefix "SRCDS-XLFRAG " --log-ip-options -m limit --limit 1/m --limit-burst 1
-A SRCDS -m length --length 2521:65535 -j DROP

#accept established connections
-A SRCDS -m state --state RELATED,ESTABLISHED -j ACCEPT

#log new packets
-A SRCDS -j LOG

#block excessive unsolicited udp packets to srcds
-A SRCDS -m hashlimit --hashlimit-mode srcip --hashlimit-name srcds-newflood --hashlimit 2/s --hashlimit-burst 6 -j ACCEPT

#Drop unaccepted traffic
-A SRCDS -j DROP
```

* `-A <name of chain> -p udp -i <listen interface> --destination-port <listen port> -j SRCDS` can be added to a file where you want it to be processed.
* `-A <name of chain> -i <listen interface> -p tcp --dport <listen port> -m limit --limit 1/s --limit-burst 3 -j ACCEPT` can be added to handle RCON.
* The standard `-A <name of chain> -m state --state RELATED,ESTABLISHED -j ACCEPT` can be added before the rules for new connections to handle those initiated by SRCDS, or otherwise already established.


Miscellaneous Optimizations
----------------------------------

The purpose of this section is to describe a number of optimizations to improve the performance of SRCDS in various areas. Please use these optimizations with care, and consider your own situation when implementing them. It is possible for some optimizations to be detrimental, or otherwise have little effect on performance in specific circumstances. Note that I might not go in to great details regarding the implementation of these optimizations, as an entire guide could be written on them alone.

* Dystopia dedicated server will start with a tickrate of 66, by default. While it is technically possible to change this, doing so is inadvisable. The in game physics were not made to properly support other tickrates, and in my experience, setting a higher tickrate essentially results in the gameplay being sped up from the player perspective.

* The ConVar "host_timer_spin_ms" can possibly improve the performance of SRCDS if the timing mechanism implemented in the operating system is imprecise. It is set to time in milliseconds, with a default value of zero. Internally, there is a delay between each server tick, or update (this delay is affected by the tickrate).

  During the delay, normally a system call is made for the process to "sleep"; this allows for it to relinquish CPU time that can be used by another process, however, the operating system must eventually "wake" the process so that it will run again. If the operating system does not wake the process on time, server performance can be negatively impacted. host_timer_spin_ms addresses this issue by causing the process to wake a specified number of milliseconds before it normally would, thus compensating for the lack of precision. The downside of it, is that the CPU usage of the server will increase. In my experience, the Linux kernel timing mechanism is pretty precise, and this is not necessary, especially with improvements in hardware clock sources.

* Server performance may benefit by the process being run at a lower nice value, which will increase the priority of it. Running the server with realtime priority is inadvisable, as this would cause it to compete with system processes for resources.

* If your system has a lot of RAM, you might consider moving server content to a RAM disk. The primary focus should probably be on files that consume a large quantity of space, but do not change often (maps, materials, models, sounds, etc), or temporary files that might be frequently changed or accessed (downloads).

  This optimization is inadvisable if you notice RAM being consumed to the point that swap space is used, as a result. The point of moving content to a RAM disk is to avoid drive IO. Another thing to note is that, as RAM is volatile memory, the content will need to be copied back to it's respective location on each boot. Backups are a good idea regardless of this optimization.

  The ramdisk can be created by adding an entry to /etc/fstab (You will need to restart, or manually mount the RAM disk with: `mount -t tmpfs -o defaults,nodev,nosuid,noexec,size=<Desired size of ramdisk> tmpfs <Absolute path to directory>` for this to take effect):

  `tmpfs <Absolute path to directory> tmpfs defaults,nodev,nosuid,noexec,size=<Desired size of ramdisk> 0 0`

  The permissions specified here should be sufficient for the maps directory. It should not be necessary to create device files, or  run any executables from it. The size of the RAM disk is in kilobytes by default, however, appending 'g' or 'm' to the value, will set the unit to gigabytes, and megabytes, respectively. '%' can also be used to set the size to a percentage of the system RAM. Note the size of the ramdisk cannot be greater than 50 percent of the installed RAM.


Known Bugs
----------------------------------

The purpose of this section is to list a number of known bugs, or quirks in SRCDS, and possible strategies for mitigating them.

* The sv_lan ConVar does not prevent the server being accessed from outside the local area network. Binding SRCDS to a LAN IP address, then forwarding the port being listened on, will cause the server to be accessible from outside the LAN, and visible from the internet tab in the server browser. A more sane way for sv_lan to behave, is to prevent the server from being accessed by a host outside the subnet of the bound IP address.

* The functionality of the sv_pure ConVar is very unclear. Many of the files that it attempts to load are not present with a clean install of Dystopia; the only present one is `<Name of server directory>/dystopia/pure_server_whitelist.txt`. It also appears that this file can be loaded from `<Name of server directory>/dystopia/cfg` as well. Generally speaking, the files loaded by a given sv_pure setting do not match those specified in the description of the ConVar provided by the server.

  Another oddity is that sv_pure seems to always be set to 0 upon server start. This occurs after ConVars passed from the command line are processed, but before server.cfg (and the map) is loaded. As far as I know, sv_pure must be set before a map loads, in order to take effect. The result is that sv_pure will effectively be 0 for the very first map loaded by the server; it should work correctly for subsequent maps.

  Finally, as previously mentioned in the description for this ConVar: a setting of 1 or 2, appears to have equivalent effect to a setting of 0. These settings seem to have no effect on the ability of a client to use custom content. Further, although the server console will report that it is set to the correct value, the client console will report that it is set to 0. A setting of -1 is correctly reported however.

* The mp_scalespawntime ConVar is thought to take no effect, due to changes in the spawn timer code for Dystopia 1.4. Please feel free to mention any behavior you have seen to the contrary.

* As mentioned in the Map Management section `<Name of server directory>/dystopia>/maplist.txt` is basically unremoved content from a previous version of the game, and now has no effect on the ability to load, or to list maps with the `maps <substring>` command. Instead, the contents of `<Name of server directory>/dystopia>/maps` will directly determine the available maps.

* There is an issue with string parsing for client side chat (the say and say_team commands) in Dystopia that will cause a server to crash; it does not manifest itself when using the say command, server side. To explain it, I will first describe a feature of the client side version of the commands: if the first character of the passed argument is ", then the first, and final characters will be truncated. I imagine the rationale for this design choice is so that chat messages can be surrounded by quotes (this will be done to any message submitted through the chat prompt).

  So, if the submitted string is only two characters long, or shorter, this essentially creates an empty string. I believe the empty string condition is what crashes the server. This can work in unexpected ways to cause crashes, even from the chat prompt, which surrounds all messages with quotes. Consider the input ";. The chat prompt would cause this to be submitted as `say "";"`. ; has the syntactical significance of indicating the end of a command, so it would be interpreted as: `say "" <END OF COMMAND> "`. This produces the same empty string condition.

  Further evidence to support this is that the SourceMod server addon will prevent the crash if `say ""` is entered. I believe the section of code responsible for this is near the comment that starts with "The server normally won't display empty say commands, but in this case it does" here: https://github.com/alliedmodders/sourcemod/blob/master/core/ChatTriggers.cpp The reason SourceMod does not block all incidences of the empty string condition, is that the check only occurs if the first and last characters are quotes, and the processing performed by SourceMod occurs before that performed by the game.

  One possible method of fixing this bug is by a SourceMod plugin that I wrote (sorry for the self plug). You can find the source code, and some documentation for it here: https://github.com/dysemjay/SayCommandFix

* You may notice a number of intermittent server crashes for no clear reason. These do not happen exceptionally often, and usually do not cause any long term problem, as the server can simply be started again by the srcds_run script. In my experience, they might occur once or twice a week, and the debugging information describes a segmentation fault occurring in `<Name of server directory>/dystopia/bin/server_srv.so`. Please feel free to offer any further insight on this.


Useful Links
----------------------------------

* Pure Servers - https://developer.valvesoftware.com/wiki/Sv_pure  
  Explains standard functionality of the sv_pure ConVar in detail.

* SRCDS Hardening - https://wiki.alliedmods.net/Srcds_hardening  
  A thorough list of security practices, and bugs for SRCDS.

* SSDOptimization - https://wiki.debian.org/SSDOptimization  
  Detailed instructions for creating an optimal environment for an SSD drive . Much of this information should carry over to other operating systems, to some extent.

* sysctl - https://wiki.archlinux.org/index.php/sysctl  
  Contains an introduction to passing arguments to the Linux kernel through sysctl, and links to more detailed documentation.

* Tuning the Task Scheduler - https://doc.opensuse.org/documentation/leap/tuning/html/book.sle.tuning/cha.tuning.taskscheduler.html  
  Excellent documentation on the Linux kernel task scheduler.


Credits
----------------------------------

* Valve Developer Wiki - https://developer.valvesoftware.com/wiki/Main_Page  
  Detailed general source of information regarding SRCDS, and the Source engine in general.

* Dystopia Wiki - http://www.dystopia-game.com/wiki/index.php?title=Main_Page  
  Much of the documentation here is very dated, however, it still provided some insight in to features specific to the Dystopia dedicated server.

* Steam user forums - http://forums.steampowered.com/forums/  
  Place for administrators to discuss SRCDS, and the Source engine in general. A good number of bugs are reported here as well.

* Source Dedicated Server website - http://www.srcds.com/  
  Most of the useful information was gleaned from forum posts. Discussions by other administrators did much to clarify SRCDS behavior.

* AlliedModders forum - https://forums.alliedmods.net/index.php  
  Excellent source of information regarding source engine internals. Another place for administrators to discuss SRCDS.

* Linux man pages - https://linux.die.net/man/  
  General source of documentation regarding the Linux operating system, and common programs for it.

* Tom Sawyer - http://steamcommunity.com/profiles/76561197983306799/  
  Wrote an initial guide for installing Dystopia 1.4, back when it was first released, and documented the fix for Dystopia 1.4.1 being broken by an SDK update. Confirmed that maplist.txt has no effect. Explained behavior of mp_autoteambalance, and mp_scalespawntime ConVars. Described algorithm used by callvote balanceteams, and the serverside balanceteams command. Directed my attention to Source SDK 2013 code associated with team balance.

* Salty Peasant - http://steamcommunity.com/profiles/76561198041115308/  
  Corrected my basic description of the sv_pure ConVar. Also brought it to my attention that sv_pure seemed to have no effect on the ability of a client to load custom content.

* Source SDK 2013 Source Code - https://github.com/ValveSoftware/source-sdk-2013  
  Provided a good deal of insight into Dystopia internals; especially regarding the algorithm associated with the mp_autoteambalance ConVar.

* And players like YOU!


Changelog
----------------------------------

Version numbers should be composed of 3 decimal places, each separated by a '.'. If a decimal place is incremented, all lower decimal places should be reset to 0. The purpose of each (listed from leftmost place to rightmost place) is:

1. Count large scale changes, usually affecting many sections. These should majorly alter the content of the guide.
2. Count major changes. These might be adding, removing, or otherwise rewriting substantial portions of sections.
3. Count minor changes. These might be to correct typographical errors, or to make other localized changes.

V2.1.2 11-01-2017

* Clarify phrasing in description of sv_allowdownload ConVar.
* Correct two typos in description of sv_allowdownload ConVar.
* Explain issue with self signed certificates for HTTPS, in description of sv_downloadurl ConVar.
* Include that HTTP redirects will be followed, in description of sv_downloadurl ConVar.

V2.1.1 10-31-2017

* Remove unnecessary end of line spaces in Introduction section.
* Clarify that 'ConVar' is a contraction of 'Console Variable' in the server.cfg (The primary configuration file) section.
* Remove unnecessary end of line spaces in server.cfg (The primary configuration file) section.
* Specify direct download rate in description of sv_allowdownload ConVar.
* Fix formatting in server.cfg (The primary configuration file) section.
* Remove unnecessary end of line spaces in Miscellaneous Optimizations section.
* Correct typo in Changelog section.

V2.1.0 06-07-2017

* Add table of contents with links to each section.
* Add bullet points in Server Installation section.
* Remove superfluous space in Miscellaneous Optimizations section.
* Precede single digits in dates with 0 in Changelog section.
* Fix typo in Changelog section.

V2.0.2 02-24-2017

* Correct typo in Getting SteamCMD section.
* Correct typo in description of sv_lan ConVar.
* Correct typo in description of mp_autoteambalance ConVar.
* Correct typo in Starting the Server section.
* Change 'minimalistic' to 'minimalist' in Working From the Server Console/Common Commands section.
* Correct typo in description for kickid command.
* Add Markdown code section in Banning Players section.
* Correct typo in Banning Players section.
* Correct two typos in Rate Settings section.
* Correct three typos in Miscellaneous Optimizations section.
* Correct two typos in Known Bugs section.
* Correct typo in Credits section.

V2.0.1 01-25-2017

* Describe default hostname displayed in the server browser.
* Describe character limit for hostname in the server browser.

V2.0.0 12-09-2016

* Remove redundant Description section.
* Include repository, and Steam discussion page links in Introduction section.
* Explain mp_autoteambalance behavior.
* Add description for mp_scalespawntime ConVar.
* Describe algorithm used by team balance performed by callvote balanceteams.
* Describe verified maplist.txt behavior.
* Describe algorithm used by the balanceteams command.
* Clarify description for Dystopia stats outgoing port range.

V1.1.2 12-02-2016

* Correct typo in Map Management section.
* Describe behavior for mapcycle.txt, if an entry cannot be found.

V1.1.1 11-25-2016

* Fix shell script to populate maplist.txt.
* Add period to end of sentence in Map Management section.
* Add Markdown code sections in Known Bugs section.

V1.1.0 11-21-2016

* Fix formatting in Changelog section.
* Correct sv_pure description, and better describe its bugs. Thanks to Salty Peasant!
* Add caveat to sv_lan, about not setting it from the command line.

V1.0.0 11-19-2016

* Initial Release.


