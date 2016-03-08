# check_usolved_veeam_backups

## Overview

A PowerShell script that checks for failed Veeam backups and can be called by Nagios/Icinga with NRPE.
It's also possible to blacklist jobs that you don't want to check.

## Authors

Ricardo Klement ([www.usolved.net](http://usolved.net))

## Installation

### Prerequisites

1. Make sure to have the [NSClient++](https://www.nsclient.org) agent installed on your Veeam server. 
Refer to the NSClient++ documentation if you don't already have the agent installed.

2. The PowerShell [VeeamPSSnapIn](https://www.veeam.com/kb1489) is also required to use this script. 

Now copy the PowerShell script check_usolved_veeam_backups.ps1 into your NSClient++ folder of the Veeam server.
For example into this path: C:\nsclient\scripts

Execute check_usolved_veeam_backups.ps1 to see if the script is working fine.

3. Nagios plugin check_nrpe needs to be installed on your monitoring server.


### Add command to NSClient++ configuration

Open nsclient.ini and configure the following commands.

```
[/modules]

; NRPE server - A simple server that listens for incoming NRPE connection and handles them.
NRPEServer = enabled


[/settings/NRPE/server]

; COMMAND ARGUMENT PROCESSING - This option determines whether or not the we will allow clients to specify arguments to commands that are executed.
allow arguments = true

; COMMAND ALLOW NASTY META CHARS - This option determines whether or not the we will allow clients to specify nasty (as in |`&><'"\[]{}) characters in arguments.
allow nasty characters = true


[/settings/external scripts/scripts]

check_usolved_veeam_backups = cmd /c echo scripts\check_usolved_veeam_backups.ps1 "$ARG1$"; exit($lastexitcode) | powershell.exe -command -
```

Restart NSClient++ service to activate the changes.


## Usage

### Test on command line
If you are in the Nagios plugin directory execute this command:

```
./check_nrpe -H ip_address_of_veeam_server -p 5666 -c check_usolved_veeam_backups -t 60
```

The output could be something like this:

```
Backup Status - Failed: 1 / Warning: 2 / OK: 112 / None: 3 / Skipped: 0
Failed: Jobname1 (03.03.2016)
Warning: Jobname5 (04.03.2016), Jobname6 (04.03.2016)
```

Here are all arguments that can be used within this plugin:

```
-a <job names for blacklist>
Optional: Comma seperated list with Veeam backup job names that you don't want to check
```

### Install in Nagios

Example for checking Veeam backups with a backup job blacklist:

Edit your **commands.cfg** and add the following.

```
define command {
    command_name    check_usolved_veeam_backups
    command_line    $USER1$/check_nrpe -H $HOSTADDRESS$ -p 5666 -c check_usolved_veeam_backups -t 60 -a '$ARG1$'
}
```

Edit your **services.cfg** and add the following:

```
define service{
	host_name				Test-Veeam-Server
	service_description		Veeam-Backups
	use						generic-service
	check_command			check_usolved_veeam_backups!Jobname1,Jobname2
}
```

## What's new

v1.0 2016-03-08
Initial release