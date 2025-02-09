# Puppet Installation {#puppet-installation-1}

## Puppet Server Installation

### Enable Root Acess

``` bash
sudo su -
```

-   Root access needed for ssl certificate access at puppetmaster

### Set hostname to *server-name*

``` bash
hostnamectl set-hostname <server-name>
```

### Enable the Puppet platform repo

``` bash
rpm -ihv https://yum.puppetlabs.com/puppet-release-el-8.noarch.rpm
```

-   The following steps are especially for puppet version 7.31.0, and
    the above repo was used.
-   For more available repo visit: <http://yum.puppetlabs.com/>

### Install puppetserver

``` bash
yum -y install puppetserver
```

### Add PATH variable for puppet

*This helps in accessing puppet commands from anywhere*

-   Go to /etc/profile.d
-   Open puppet-agent.sh file
-   Replace with the below code

``` bash
if [ -z "${PATH-}" ] ; then
  export PATH=/opt/puppetlabs/bin
  export PATH=/opt/puppetlabs/puppet/bin
elif ! echo "${PATH}" | grep -q /opt/puppetlabs/bin ; then
  export PATH="${PATH}:/opt/puppetlabs/bin"
  export PATH="${PATH}:/opt/puppetlabs/puppet/bin"
fi

if ! echo "${MANPATH-}" | grep -q /opt/puppetlabs/puppet/share/man ; then
  export MANPATH="${MANPATH-}:/opt/puppetlabs/puppet/share/man"
fi
```

-   Logout from root and login again and

``` bash
exec bash
```

, then check by running

``` bash
puppet
```

, if you are able to access puppet commands or not.

### Add hostname and ip configurations

-   Go to /etc/puppetlabs/puppet directory
-   Open puppet.conf file and add the following lines

``` bash
[main]
server=<server-name>
```

-   Go to /etc and open the hosts file and add the following line

``` bash
<server-ip-address> <server-name>
```

### Start puppetserver service, check status is active and check if port 8140 is listening

``` bash
systemctl start puppetserver
systemctl status puppetserver
netstat -ntlp
```

### Firewall port permission

*TCP connection to port 8140 needed firewall to allow permission*

-   Add the port and check in the list if it is allowed

``` bash
firewall-cmd --zone=public --add-port=8140/tcp --permanent
firewall-cmd --reload
firewall-cmd --list-all
exec bash
```

### Building Manifests

*Manifests are files which provide conifgurations or instructions to
puppetmaster to execute them on the puppet agents*

-   Go to /etc/puppetlabs/code/environments/production/manifests and
    create a site.pp file
-   Add the following lines in it

``` bash
node /abcd/ {
    include webserver
}
```

-   The \'*abcd*\' name depicts that any agent that includes \'abcd\' in
    its hostname will be provided with this catalog(configuration) when
    it requests for catalog
-   You can also execute specifically for a hostname like below

``` bash
node "puppet-agent-26" {
    include webserver
}
```

-   \'webserver\' is the module whose catalog will be provided
-   To create the webserver module, we go to
    /etc/puppetlabs/code/environments/production/modules and create a
    directory with the name \'webserver\'
-   Inside this webserver directory, we create another directory named
    \'manifests\', and in the mainfests directory we create an init.pp
    file
-   This init.pp defines the configurations of this specific webserver
    module
-   Add the following lines in this file

``` bash
class webserver {
        if $::osfamily == 'RedHat' {
                package { 'httpd' :
                        ensure => present
                }
        } elsif $::osfamily == 'Debian' {
                package { 'apache2' :
                        ensure => present
                }
        } else {
                notify { 'unsupported_os':
                        message => 'This OS is not supported by the webserver module.',
                        loglevel => 'warning',
                }
        }
}
```

-   This installs a httpd web server in your puppet agent
-   For other applications, include them in the respective agents witin
    the site.pp file and create its own modules directory, create a
    manifests directory in it and create an init.pp in that.
-   Now your Puppetserver is ready to sign the requests from puppet
    agents.

## Puppet Agent Installation (Manually)

### Enable Root Acess

``` bash
sudo su -
```

### Set hostname to *agent-name*

``` bash
hostnamectl set-hostname <agent-name>
```

### Enable the Puppet platform repo

``` bash
rpm -ihv https://yum.puppetlabs.com/puppet-release-el-8.noarch.rpm
```

-   Use the same repo as used in the puppet server to avoid conflicts

### Install puppet-agent

``` bash
yum -y install puppet-agent
```

### Add PATH variable for puppet

*This helps in accessing puppet commands from anywhere*

-   Go to /etc/profile.d
-   Open puppet-agent.sh file
-   Replace with the below code

``` bash
if [ -z "${PATH-}" ] ; then
  export PATH=/opt/puppetlabs/bin
  export PATH=/opt/puppetlabs/puppet/bin
elif ! echo "${PATH}" | grep -q /opt/puppetlabs/bin ; then
  export PATH="${PATH}:/opt/puppetlabs/bin"
  export PATH="${PATH}:/opt/puppetlabs/puppet/bin"
fi

if ! echo "${MANPATH-}" | grep -q /opt/puppetlabs/puppet/share/man ; then
  export MANPATH="${MANPATH-}:/opt/puppetlabs/puppet/share/man"
fi
```

-   Logout from root and login again and

``` bash
exec bash
```

, then check by running

``` bash
puppet
```

, if you are able to access puppet commands or not.

### Add hostname and ip configurations

-   Go to /etc/puppetlabs/puppet directory
-   Open puppet.conf file and add the following lines

``` bash
[main]
server=<server-name>
```

-   Go to /etc and open the hosts file and add the following line

``` bash
<server-ip-address> <server-name>
```

### Firewall port permission

*TCP connection to port 8140 needed firewall to allow permission*

-   Add the port and check in the list if it is allowed

``` bash
firewall-cmd --zone=public --add-port=8140/tcp --permanent
firewall-cmd --reload
firewall-cmd --list-all
exec bash
```

### Start puppet-agent service, check status is active and check if port 8140 is listening

``` bash
systemctl start puppet
systemctl status puppet
netstat -ntlp
```

## Puppet Agent Installation (Automated using script)

-   Copy the file -------- into all systems where agents should be
    installed.
-   Dont login to root, script will do that
-   Give 755 permission to the file and run it with sudo

``` bash
chmod 755 ./-----
sudo ./-----
```

## Signing Certificates

-   Run the below command in puppet agent to request for signing its
    certificate

``` bash
puppet agent --test --server <server-name>
```

-   An RSA SSL key should be created

-   On the puppet server, list the requested certificates and sign them
    using the below commands

``` bash
puppetserver ca list --all
puppetserver ca sign --certname <agent-name>
```

-   Puppetserver should display that it has been successfully signed

```{=html}
<!-- -->
```
-   On the puppet agents, run the test command to execute the manifests
    on the puppet agents

``` bash
puppet agent --test
```

-   This should receive catalogs and compile them
