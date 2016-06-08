# minecraft-console
Bash scripts for managing Minecraft servers.

## Console

`console` manages the server's startup, shutdown, backups, and much more. It works as a manager, leaving the server running as the daemon. It is designed to copy the syntax of systemd.

Everything is configured by running `console config`. It supports multiple servers, multiple worlds (maps) per server, and infinite backups.

Syntax: `console [command] [profile]`

`all` is a special profile keyword that runs the command for all configured profiles.

### start
Start the specified profile. Will report if the server fails to start.

### stop
Stop the specified profile. Will report if the server refuses to close. Will issue a warning to online players before closing the server, and wait for `default_time`, unless a time is specified.

### restart
Restart the specified profile. If the profile is not running, it will be started. Will issue a warning and wait like `stop`.

### try-restart
Restarts the specified profile, if it is running. If it not, it will do nothing. Will issue a warning and wait like `stop`.

### status
Prints the status of the specified profile, if it is running or not.

### backup
Uses `rsync` to backup the server. Handles the server running while backing up. It using the diff backup feature of `rsync` to save space. Best used in a cron job or similar.

### say
Sends a message to the server that is printed to all online uses. This is designed to be used in automatic scripts. To talk to users directly, it would be better to use `see`.

### command
Sends a command to the server. This is designed to be used in automatic scripts. To use many commands or see their output, it would be better to use `see`.

### update
Updates the version of Minecraft installed in the specified profile. It downloads the jar files from Mojang using their interface to figure out what is the latest version. Optionlly, a version can be specified after the profile to update to that specific version. Will correctly handle running servers and also backup the server before updating.

### see
Creates a setup of screen to help chat and commands to the server. The server can be connected to directly with `screen -r profile-name`, but this can cause problems if there are automatic scripts running using the `say` or `command` commands to control the server, since they use same text buffer. Using the `see` command removes the problem, as well as adding helpful shortcuts to controlling the server.

### shell
Used by the `see` command for its command line. It can be used by itself if wanted to make many commands in a row easier to use.
Commands:
* `c|command`   runs the command on the server. Also puts the shell into command mode.
* `s|say`       sends the text as text to players
* `nick`        sets the nickname for showing when text is sent to players
* `help`        display a quick list of commands
* `exit`        exits the shell/console

### list
Lists the profiles that are configured

### config
Opens the config file for editing. After editing, it will check the config for errors and notify for any it finds.

### new
Adds a default profile to the config file. Takes a type argument for further customization.

### help
Displays a quick help text.


## Config
List of config options:
* `default_time`      the default time that stop will wait before closing the server
* `player_list_path`  the path where the minecraft.* files are stored for sharing with every server. If not specified, each server will use their own lists.
* `eula`              if true, console will set each server's eula to true to save time.
* `java`              the java command to use to launch the servers. Useful if you have a specific version of java for Minecraft.
* `profile_list`      the list of profiles used when `all` is specified instead of of profile name. If a profile is not in this list, it will work normally, but it will not be part of `all` nor scanned for config errors.
Profile specific:
* `type`              the type of server. Right now, only minecraft is supported.
* `autostart`         if the server should be started when `console start all` is run. This allows profiles to be configured without needing to run automaticlly.
* `server_path`       the location of the server directory. Where the server files are saved.
* `backup_path`       the location of the backup directory. If not specified, backups will be disabled for that profile.
* `world`             the name of the world directory. This allows multiple worlds (maps) to be used per profile. Put a different server.properties file in each world directory, then change the `world` option to select the different world to use.
* `jar_name`          the jar file to run for the profile. If the profile is of type vanilla or snapshot, you can specify the function `find_jar` instead to use the only jar file in the directory. This allows for updating without changing the config.
* `server_command`    the command to run when starting the server. Probably want to have `${java}` at the beginning and `${jar_name}` after the -jar flag.
* `updateable`        the type of server if updating is wanted, false otherwise. This is used to discover what version to update the profile.


## Listener

`listener` will watch a log file, and perform actions based on what the server prints to the file. To use it, run something like:

`screen -dmS listener listener vanilla`

This will start it in background mode. To kill it, connect to the screen (`screen -r listener`) and Ctrl-C.

It adds many nifty features, such as:
* Sending players messages when they login.
* Recording the logout times of players to tell them on login when they were last online.
* Responding to hello's, and simple questions
* Following simple commands, complete with error messages.
* Logging all chat to log/chat.log to have a log file without all of the crap.

Commands:
* !info - list info about the server
* !help - list server commands
* !playerlist - print the whitelist
* !playerinfo [playername] - show the last logout time of playername
