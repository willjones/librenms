# Setting up syslog support

This document will explain how to send syslog data to LibreNMS.

### Syslog server installation

For Debian / Ubuntu:
```ssh
apt-get install syslog-ng
```

For CentOS / RedHat
```ssh
yum install syslog-ng
```

Once syslog-ng is installed, edit the relevant config file (most likely /etc/syslog-ng/syslog-ng.conf) and paste the following:

```ssh
@version: 3.5
@include "scl.conf"
@include "`scl-root`/system/tty10.conf"

# First, set some global options.
options {
        chain_hostnames(off);
        flush_lines(0);
        use_dns(no);
        use_fqdn(no);
        owner("root");
        group("adm");
        perm(0640);
        stats_freq(0);
        bad_hostname("^gconfd$");
};
 
########################
# Sources
########################
source s_sys {
       system();
       internal();
};
 
source s_net {
        tcp(port(514) flags(syslog-protocol));
        udp(port(514) flags(syslog-protocol));
};
 
########################
# Destinations
########################
destination d_librenms {
        program("/opt/librenms/syslog.php" template ("$HOST||$FACILITY||$PRIORITY||$LEVEL||$TAG||$YEAR-$MONTH-$DAY $HOUR:$MIN:$SEC||$MSG||$PROGRAM\n") template-escape(yes));
};
 
########################
# Log paths
########################
log {
        source(s_net);
        source(s_sys);
        destination(d_librenms);
};
 
###
# Include all config files in /etc/syslog-ng/conf.d/
###
@include "/etc/syslog-ng/conf.d/*.conf"
```

Next start syslog-ng:

```ssh
service syslog-ng restart
```

Add the following to your LibreNMS config.php file to enable the Syslog extension:

```ssh
$config['enable_syslog'] = 1;
```

### Client configuration

Below are sample configurations for a variety of clients. You should understand the config before using it as you may want to make some slight changes.

Replace librenms.ip with IP or hostname of your LibreNMS install.

Replace any variables in <brackets> with the relevant information.

#### syslog
```config
*.*     @librenms.ip
```

#### rsyslog
```config
*.* @librenms.ip:514
```

#### Cisco ASA
```config
logging enable
logging timestamp
logging buffer-size 200000
logging buffered debugging
logging trap notifications
logging host <outside interface name> librenms.ip
```

#### Cisco IOS
```config
logging trap debugging
logging facility local6
logging librenms.ip
```

#### Cisco NXOS
```config
logging server librenms.ip 5 use-vrf default facility local6
```

If you have permitted udp and tcp 514 through any firewall then that should be all you need. Logs should start appearing and displayed within the LibreNMS web UI.
