nagios-check-infections
=======================
Last modified: 2014-03-10 07:09
Author: Bernhard Brunner

Nagios plugin to check for infections of new, in-the-wild root kits not covered by chkrootkit, rkhunter etc.

Currently, this nagios plugin checks for the following rootkits:
- cdorked using an external c program. See the original blog post http://www.welivesecurity.com/2013/04/26/linuxcdorked-new-apache-backdoor-in-the-wild-serves-blackhole/ for more information. 
- ebury
- darkleech

Adding the check to nagios3
---------------------------
1. Prerequisite: set up check_by_ssh for the servers(s) you want to check
2. Copy the script and cdorked.c program to the server, e.g. in directory /usr/local/bin
3. Compile the cdorked.c program on the server, rename the file to "cdorked-`uname -m`" so that you have an exe for your cpu. 
4. Add command and service to your nagios configuration. In the configuration below, all servers belonging to the group linux-servers will automatically get checked using nagios-check-infections. Nagios configuration is beyond the scope of this readme. My configuration is based on hostgroups, so when adding a host just the hostgroups will specified, the service checks are triggered by the hostgroups.


Define a new command ssh_check_infections:

````
  # 'ssh_infections command definition
  # note: the host variable _SSHPORT is used to allow non-standard ssh ports
  define command
  →   command_name ssh_check_infections
  →   command_line /usr/lib/nagios/plugins/check_by_ssh -p $_HOSTSSHPORT$ -t 30 -H $HOSTADDRESS$ -C   "/usr/local/bin/nagios-check-infections"
  }
````

````
  # Define a service to check for infections
  define service{
          use                             generic-service         ; Name of service template to use
          hostgroup_name                  linux-servers
          service_description             Infections
          check_command                   ssh_check_infections
  }
````

