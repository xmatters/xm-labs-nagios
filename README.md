# Nagios Core
Nagios is the industry standard infrastructure monitoring tool. This integration expands the alerting capabilities of Nagios to leverage the xMatters communication protocols and integration platform for driving workflow across tools. 

---------

<kbd>
  <a href="https://support.xmatters.com/hc/en-us/community/topics">
  <img src="https://github.com/xmatters/xMatters-Labs/raw/master/media/disclaimer.png">
  </a>
</kbd>

---------

An updated version of this integration is available, supporting the latest version of Nagios and based on xMatters Flow Designer so you can easily connect other tools to your toolchain. Install it right from the Workflow Template directory within your xMatters instance. [Learn more](http://help.xmatters.com/integrations/#cshid=Nagios).

---------

# Pre-Requisites
* Nagios Core 4.3.x
* xMatters account - If you don't have one, [get one](https://www.xmatters.com)!

# Files
* [Nagios.zip](Nagios.zip) - Nagios Workflow with necessary the notification templates and integration scripts. 
* [xmatters.cfg](xmatters.cfg) - The contact and command entries for the xMatters notifications

# How it works
This integration uses a Nagios `command` to fire a curl request into one of the flow designer canvases, either Host or Service. 

# Installation


## xMatters set up
1. Create a new user called `nagios` and grant the `Standard User`, `Limited Developer` and `REST Web Services User` roles.
2. Click the Login as This User and nagivate to the Workflow page. Click the Import button and import the [Nagios.zip](Nagios.zip) file. If other users should be able to edit the scripts or forms going forward, click the Edit > Access Permissions on the Nagios Workflow and add the necessary users or roles. 
3. On the Flows tab, click inside the **New Host Notification** and double click (or click the edit pencil) next to the **New Host Notification** http trigger with the Nagios icon on it. In the dialog presented, copy the url at the bottom. 

<kbd>
  <img src="/media/triggersettings.png" width="500">
</kbd>
Repeat for the **New Service Notification** flow. Keep urls for later. 


## Nagios set up

1. Login to the Nagios host machine and navigate to the `NAGIOS_HOME` directory. 
2. Copy the [xmatters.cfg](xmatters.cfg) file to the `NAGIOS_HOME/etc` directory. Open this file and replace the `XMATTERS_SERVICE_URL_HERE` and `XMATTERS_HOST_URL_HERE` value with the inbound urls copied from above. *Note* If a different authentication method on the inbound integration was selected above, then the curl command will need to be updated. The `-u, --user` parameter will do basic auth or API Key auth. The syntax will be `--user "username:password"` See the curl help for more details. 
3. Open the `nagios.cfg` file and in the `OBJECT CONFIGURATION FILES` section, add this line, replacing `NAGIOS_HOME` with the full path to Nagios.

```
cfg_file=NAGIOS_HOME/etc/xmatters.cfg
```
4. Add the `xmatters` contact to the appropriate service by updating the service definition file. For example, to add xmatters to the ping service in the `NAGIOS_HOME/etc/objects/localhost.cfg`, add a `contacts` entry to make it look like this:

```cfg
define service{
    use                    local-service         ; Name of service template to use
    host_name              localhost
    service_description    PING
    check_command          check_ping!100.0,20%!500.0,60%
    contacts               xmatters
  }

```

5. Restart Nagios for the changes to take effect. 

## Adding xMatters to the template (optional)
Instead of adding the `xmatters` contact to each host and service entry, the contact can be added at the template level the host or service inherits from. In the `local-service` example above, instead of adding `xmatters` to the service, track down the template definition for `local-service`. 


# Testing
To test the integration, find a service that can be taken down for testing, or, alternatively, use the "Send custom service notification" on the service updated above. This is a screen shot of the PING service:

<kbd>
  <img src="media/ping_service.png">
</kbd>



# Troubleshooting
The `nagios.cfg` file contains debug logging settings and the debug log information. Setting the `debug_level` to `256` and inspecting the `debug_file` file should show debug level information and any errors associated with the curl command. 

Also check the activity stream in the host or service canvas for additional details.

## NSS error -5938 Error
Some curl installations have reported a `NSS error -5938` error when making the curl request to xMatters. This can be solved by adding the `-1, --tlsv1` parameter to force TLS v1. For example:

```
define command {
        command_name    notify_xmatters_service
        command_line    curl --tlsv1 -X POST -H "Content-Type: application/json" -d '{ "NAGIOS_CONTACTGROUPNAME": "$SERVICEDISPLAYNAME$", "NAGIOS_HOSTDISPLAYNAME": "$HOSTDISPLAYNAME$", "NAGIOS_HOSTNAME": "$HOSTNAME$", "NAGIOS_HOSTOUTPUT": "$HOSTOUTPUT$", "NAGIOS_HOSTSTATE": "$HOSTSTATE$", "NAGIOS_LASTHOSTSTATECHANGE": "$LASTHOSTSTATECHANGE$", "NAGIOS_LASTSERVICESTATECHANGE": "$LASTSERVICESTATECHANGE$", "NAGIOS_NOTIFICATIONAUTHOR": "$NOTIFICATIONAUTHOR$", "NAGIOS_NOTIFICATIONCOMMENT": "$NOTIFICATIONCOMMENT$", "NAGIOS_NOTIFICATIONTYPE": "$NOTIFICATIONTYPE$", "NAGIOS_SERVICEDESC": "$SERVICEDESC$", "NAGIOS_SERVICEOUTPUT": "$SERVICEOUTPUT$", "NAGIOS_SERVICESTATE": "$SERVICESTATE$", "NAGIOS_TIMET": "$TIMET$" }' "XMATTERS_INBOUND_URL_HERE"
}
```

