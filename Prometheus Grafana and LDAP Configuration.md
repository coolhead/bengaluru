# Prometheus
**Prometheus download and installation steps.**

To download Prometheus, follow these steps.

1. Visit the [Prometheus downloads](https://prometheus.io/download/) and make a note of the most recent release. The most recent LTS release is clearly indicated on the site.
1. Use wget to download Prometheus to the monitoring server. The target link has the format https://github.com/prometheus/prometheus/releases/download/v[release]/prometheus-[release].linux-amd64.tar.gz. Replace the string [release] with the actual release to download. For example, the following command downloads release 2.37.6.

wget https://github.com/prometheus/prometheus/releases/download/v2.37.6/prometheus-2.37.6.linux-amd64.tar.gz

1. Extract the archived Prometheus files.

tar xvfz prometheus-\*.tar.gz

1. (**Optional**) After the files have been extracted, delete the archive, or move it to a different location for storage.

rm prometheus-\*.tar.gz

1. Create two new directories for Prometheus to use. The /etc/prometheus directory stores the Prometheus configuration files. The /var/lib/prometheus directory holds application data.

sudo mkdir /etc/prometheus /var/lib/prometheus

1. Move into the main directory of the extracted prometheus folder. Substitute the name of the actual directory in place of prometheus-2.37.6.linux-amd64.

cd prometheus-2.37.6.linux-amd64

1. Move the prometheus and promtool directories to the /usr/local/bin/ directory. This makes Prometheus accessible to all users.

sudo mv prometheus promtool /usr/local/bin/

1. Move the prometheus.yml YAML configuration file to the /etc/prometheus directory.

sudo mv prometheus.yml /etc/prometheus/prometheus.yml

1. The consoles and console libraries directories contain the resources necessary to create customized consoles. This feature is more advanced and is not covered in this guide. However, these files should also be moved to the etc/prometheus directory in case they are ever required.

sudo mv consoles/ console\_libraries/ /etc/prometheus/

1. Verify that Prometheus is successfully installed using the below command:

prometheus –version

![image](https://github.com/coolhead/bengaluru/assets/245313/5e328a7d-3542-444c-ac48-8c0487569142)


**Configure Prometheus as a Service**

Although Prometheus can be started and stopped from the command line, it is more convenient to run it as a service using the systemctl utility. This allows it to run in the background.

To configure Prometheus, follow the steps below.

1. Create a prometheus user. The following command creates a *system user*.

sudo useradd -rs /bin/false prometheus

1. Assign ownership of the two directories created in the previous section to the new prometheus user.

sudo chown -R prometheus: /etc/prometheus /var/lib/prometheus

1. To allow Prometheus to run as a service, create a prometheus.service file using the following command:

sudo vi /etc/systemd/system/prometheus.service

Enter the following content into the file:

File: /etc/systemd/system/prometheus.service

|||
| :- | :- |
1. The Wants and After options must be set to network-online.target.
1. The User and Group fields must both be set to prometheus.
1. The ExecStart parameter explains where to find the prometheus executable and defines the default options.
1. The config.file option defines the location of the Prometheus configuration file as /etc/prometheus/prometheus.yml.
1. storage.tsdb.path tells Prometheus to store application data in the /var/lib/prometheus/ directory.
1. web.listen-address is set to 0.0.0.0:9090, allowing Prometheus to listen for connections on all network interfaces.
1. The web.enable-lifecycle option allows users to reload the configuration file without restarting Prometheus.
1. Reload the systemctl daemon.

sudo systemctl daemon-reload

1. (**Optional**) Use systemctl enable to configure the prometheus service to automatically start when the system boots. If this command is not added, Prometheus must be launched manually.

sudo systemctl enable prometheus

1. Start the prometheus service and review the status command to ensure it is active.

Note: 

If the prometheus service fails to start properly, run the command journalctl -u prometheus -f --no-pager and review the output for errors.

sudo systemctl start prometheus

sudo systemctl status prometheus

1. Access the Prometheus web interface and dashboard at http://localhost:9090. Replace localhost with the address of the monitoring server. Because Prometheus is using the default configuration file, it does not display much information yet.


1. The default prometheus.yml file contains a directive to scrape the local host. Click **Status** and **Targets** to list all the targets. Prometheus should display the local Prometheus service as the only target.


**Install and Configure Node Exporter**

To install Node Exporter, follow below mentioned steps. 

1. Consult the [Node Exporter section of the Prometheus downloads page](https://prometheus.io/download/#node_exporter) and determine the latest release.
1. Use wget to download this release. The format for the file is https://github.com/prometheus/node\_exporter/releases/download/v[release\_num]/node\_exporter-[release\_num].linux-amd64.tar.gz. Replace [release\_num] with the number corresponding to the actual release. For example, the following example demonstrates how to download Node Exporter release 1.5.0.

wget https://github.com/prometheus/node\_exporter/releases/download/v1.5.0/node\_exporter-1.5.0.linux-amd64.tar.gz

1. Extract the application.

tar xvfz node\_exporter-\*.tar.gz

1. Move the executable to usr/local/bin so it is accessible throughout the system.

sudo mv node\_exporter-1.5.0.linux-amd64/node\_exporter /usr/local/bin

1. (**Optional**) Remove any remaining files.

rm -r node\_exporter-1.5.0.linux-amd64\*

1. There are two ways of running Node Exporter. It can be launched from the terminal using the command node\_exporter. Or, it can be activated as a system service. Running it from the terminal is less convenient. But this might not be a problem if the tool is only intended for occasional use. To run Node Exporter manually, use the following command. The terminal outputs details regarding the statistics collection process.

node\_exporter

1. It is more convenient to run Node Exporter as a service. To run Node Exporter this way, first, create a node\_exporter user.

sudo useradd -rs /bin/false node\_exporter

1. Create a service file for systemctl to use. The file must be named node\_exporter.service and should have the following format. Most of the fields are similar to those found in prometheus.service, as described in the previous section.

sudo vi /etc/systemd/system/node\_exporter.service

File: /etc/systemd/system/node\_exporter.service

|||
| :- | :- |
1. (**Optional**) If you intend to monitor the client on an ongoing basis, use the systemctl enable command to automatically launch Node Exporter at boot time. This continually exposes the system metrics on port 9100. If Node Exporter is only intended for occasional use, do not use the command below.

sudo systemctl enable node\_exporter

1. Reload the systemctl daemon, start Node Exporter, and verify its status. The service should be active.

sudo systemctl daemon-reload

sudo systemctl start node\_exporter

sudo systemctl status node\_exporter


1. Use a web browser to visit port 9100 on the client node, for example, http://localhost:9100. A page entitled Node Exporter is displayed along with a link reading Metrics. Click the Metrics link and confirm the statistics are being collected. For a detailed explanation of the various statistics, see the [Node Exporter Documentation](https://prometheus.io/docs/guides/node-exporter/).

**Configure Prometheus to Monitor Client Nodes**

The client node (localhost) is now ready for monitoring. To add clients to prometheus.yml, follow the steps below:

1. On the monitoring server running Prometheus, open prometheus.yml for editing.

sudo vi /etc/prometheus/prometheus.yml

1. Locate the section entitled scrape\_configs, which contains a list of jobs. It currently lists a single job named prometheus. This job monitors the local Prometheus task on port 9090. Beneath the prometheus job, add a second job having the job\_name of node\_exporter. Include the following information.
   1. A scrape\_interval of 10s.
   1. Inside static\_configs in the targets attribute, add a bracketed list of the IP addresses to monitor. Separate each entry using a comma.
   1. Append the port number :9100 to each IP address.
   1. To enable monitoring of the local server, add an entry for localhost:9100 to the list.

The entry should resemble the following example. Replace localhost with the actual IP address of the client.

File: /etc/prometheus/prometheus.yml

|<p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p></p>|<p>...</p><p>- job\_name: "node\_exporter"</p><p>`  `scrape\_interval: 10s</p><p>`  `static\_configs:</p><p>`    `- targets: **[**"localhost:9100"**]**</p>|
| :- | :- |

1. To immediately refresh Prometheus, restart the prometheus service.

sudo systemctl restart prometheus

1. Using a web browser, revisit the Prometheus web portal at port **9090** on the monitoring server. Select **Status** and then **Targets**. A link for the node\_exporter job is displayed, leading to port **9100** on the client. Click the link to review the statistics.






### Grafana

Prometheus is now collecting statistics from the clients listed in the scrape\_configs section of its configuration file. However, the information can only be viewed as a raw data dump. The statistics are difficult to read and not very useful.

Grafana provides an interface for viewing the statistics collected by Prometheus. Install Grafana on the same server running Prometheus and add Prometheus as a data source. Then install one or more panels for interpreting the data. 

To install and configure Grafana, follow these steps.

1. Install some required utilities using apt.

sudo apt-get install -y apt-transport-https software-properties-common

1. Import the Grafana GPG key.

sudo wget -q -O /usr/share/keyrings/grafana.key https://apt.grafana.com/gpg.key

1. Add the Grafana “stable releases” repository.

echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

1. Update the packages in the repository, including the new Grafana package.

sudo apt-get update

1. Install the open-source version of Grafana.

sudo apt-get install grafana

Note

To install the Enterprise edition of Grafana, use the command: 

sudo apt-get install grafana-enterprise instead.

1. Reload the systemctl daemon.

sudo systemctl daemon-reload

1. Enable and start the Grafana server. Using systemctl enable configures the server to launch Grafana when the system boots.

sudo systemctl enable grafana-server.service

sudo systemctl start grafana-server

1. Verify the status of the Grafana server and ensure it is in the active state.

sudo systemctl status grafana-server



**Integrate Grafana and Prometheus**

To integrate Grafana and Prometheus, follow the steps below:

1. Using a web browser, visit port 3000 of the monitoring server. For example, enter http://localhost:3000, replacing localhost with the actual IP address. Grafana displays the login page. Use the user name “admin” and the default password “password”. Change the password to a more secure value when prompted to do so.


1. After a successful password change, Grafana displays the Grafana Dashboard.

1. To add Prometheus as a data source, click the connections drop-down menu, then select 

**Data sources**.


1. At the next display, click the **Add data source** button.


1. Choose **Prometheus** as the data source.


















1. For a local Prometheus source, as described in this guide, set the URL to http://localhost:9090. Most of the other settings can remain at the default values. However, a non-default Timeout value can be added here.

1. select the **Save & test** button at the bottom of the screen.

1. If all settings are correct, Grafana confirms the Data source is working.


# LDAP Configuration for Grafana

The LDAP integration in Grafana allows your Grafana users to login with their LDAP credentials. You can also specify mappings between LDAP group memberships and Grafana Organization user roles.

**Supported LDAP Servers**

Grafana uses a [third-party LDAP library](https://github.com/go-ldap/ldap) under the hood that supports basic LDAP v3 functionality.

**Enable LDAP**

In order to use LDAP integration you’ll first need to enable LDAP in the [main config file](https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/) as well as specify the path to the LDAP specific configuration file (default: /etc/grafana/ldap.toml).

After enabling LDAP, the default behavior is for Grafana users to be created automatically upon successful LDAP authentication. If you prefer for only existing Grafana users to be able to sign in, you can change allow\_sign\_up to false in the **[auth.ldap]** section.

[auth.ldap]

\# Set to `true` to enable LDAP integration (default: `false`)

enabled = true

\# Path to the LDAP specific configuration file (default: `/etc/grafana/ldap.toml`)

config\_file = /etc/grafana/ldap.toml

\# Allow sign-up should be `true` (default) to allow Grafana to create users on successful LDAP authentication.

\# If set to `false` only already existing Grafana users will be able to login.

allow\_sign\_up = true

**Disable org role synchronization**

If you use LDAP to authenticate users but don’t use role mapping, and prefer to manually assign organizations and roles, you can use the skip\_org\_role\_sync configuration option.

[auth.ldap]

\# Set to `true` to enable LDAP integration (default: `false`)

enabled = true

\# Path to the LDAP specific configuration file (default: `/etc/grafana/ldap.toml`)

config\_file = /etc/grafana/ldap.toml

\# Allow sign-up should be `true` (default) to allow Grafana to create users on successful LDAP authentication.

\# If set to `false` only already existing Grafana users will be able to login.

allow\_sign\_up = true

\# Prevent synchronizing ldap users organization roles

skip\_org\_role\_sync = true

**Grafana LDAP Configuration**

Depending on which LDAP server you’re using and how that’s configured your Grafana LDAP configuration may vary. See [configuration examples](https://grafana.com/docs/grafana/latest/setup-grafana/configure-security/configure-authentication/ldap/#configuration-examples) for more information.

**LDAP specific configuration file (ldap.toml):**

\# To troubleshoot and get more log info enable ldap debug logging in grafana.ini

\# [log]

#filters = ldap:debug

[[servers]]

\# Ldap server host (specify multiple hosts space separated)

host = "code1.emi.philips.com"

\# Default port is 389 or 636 if use\_ssl = true

port = 636

\# Set to true if LDAP server should use an encrypted TLS connection (either with STARTTLS or LDAPS)

use\_ssl = true

\# If set to true, use LDAP with STARTTLS instead of LDAPS

start\_tls = false

\# set to true if you want to skip ssl cert validation

ssl\_skip\_verify = true

\# set to the path to your root CA certificate or leave unset to use system defaults

\# root\_ca\_cert = "/path/to/certificate.crt"

\# Authentication against LDAP servers requiring client certificates

\# client\_cert = "/path/to/client.crt"

\# client\_key = "/path/to/client.key"

\# Search user bind dn

bind\_dn = "CN=320169160,OU=Users,OU=INGBTCPIC5,OU=CODE,DC=code1,DC=emi,DC=philips,DC=com"

\# Search user bind password

\# If the password contains # or ; you have to wrap it with triple quotes. Ex """#password;"""

bind\_password = "\*\*\*\*\*\*"

\# We recommend using variable expansion for the bind\_password, for more info https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/#variable-expansion

\# Timeout in seconds (applies to each host specified in the 'host' entry (space separated))

timeout = 10

\# User search filter, for example "(cn=%s)" or "(sAMAccountName=%s)" or "(uid=%s)"

search\_filter = "(cn=%s)"

\# An array of base dns to search through

search\_base\_dns = ["OU=Users,OU=INGBTCPIC5,OU=CODE,dc=code1,dc=emi,dc=philips,dc=com"]

\## For Posix or LDAP setups that does not support member\_of attribute you can define the below settings

\## Please check grafana LDAP docs for examples

#group\_search\_filter = "(&(objectClass=posixGroup)(memberUid=%s))"

group\_search\_base\_dns = ["OU=Groups,OU=INGBTCPIC5,OU=CODE,DC=code1,DC=emi,DC=philips,DC=com"]

group\_search\_filter\_user\_attribute = "uid"

\# Specify names of the ldap attributes your ldap uses

[servers.attributes]

name = "givenName"

surname = "sn"

username = "sAMAccountName"

member\_of = "memberOf"

email =  "email"

\# Map ldap groups to grafana org roles

[[servers.group\_mappings]]

group\_dn = "CN=ggINGBTCPIC5-synergy-grafana-Owners, OU=Groups,OU=INGBTCPIC5,OU=CODE,DC=code1,DC=emi,DC=philips,DC=com"

org\_role = "Admin"

\# To make user an instance admin (Grafana Admin) uncomment line below

grafana\_admin = true

\# The Grafana organization database id, optional, if left out the default org (id 1) will be used

org\_id = 1

[[servers.group\_mappings]]

group\_dn = "CN=ggINGBTCPIC5-synergy-grafana,OU=Groups,OU=INGBTCPIC5,OU=CODE,DC=code1,DC=emi,DC=philips,DC=com"

org\_role = "Viewer"


