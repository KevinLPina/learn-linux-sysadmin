# How do I learn to be a Linux sysadmin?

These are some things that u/IConrad tells people to do, who ask him "how do I learn to be a Linux sysadmin?". Do these things and you will be fully exposed to every aspect of Linux Enterprise systems administation. Do them well and you will have the technical expertise required to seek "Senior" roles. If you go whole-hog crash-course full-time it with no other means of income, I would expect it would take between 3 and 6 months to go from "I think i'm good with computers" to achieving all of these -- assuming you're not afraid of IRC and google (and have neither friends nor family...).

Simply fork the repo, and interactively go through this list checking off as you go.

# The checklist

- [ ] Set up a KVM hypervisor.
- [ ] Inside of that KVM hypervisor, install a Foreman-Katello server. Use CentOS 8 as the distro for all work below. (For bonus points, set up errata importation on the CentOS channels, so you can properly see security update advisory information.)
- [ ] Create a VM to provide named and dhcpd service to your entire environment (FreeIPA). Set up the dhcp daemon to use the Foreman-Katello server as the pxeboot machine (thus allowing you to use Cobbler to do unattended OS installs). Make sure that every forward zone you create has a reverse zone associated with it. Use something like "internal.virtnet" (but not ".local") as your internal DNS zone.
- [ ] Use that Foreman-Katello server to automatically (without touching it) install a new pair of OS instances, with which you will then create a Master/Master pair of LDAP servers (FreeIPA). Make sure they register with the Foreman-Katello server. Do not allow anonymous bind, do not use unencrypted LDAP.
- [ ] Reconfigure all 3 servers to use LDAP authentication.
- [ ] Create two new VMs, again unattendedly, which will then be Postgresql VMs. Use pgpool-II to set up master/master replication between them. Export the database from your Foreman-Katello server and import it into the new pgsql cluster. Reconfigure your Foreman-Katello instance to run off of that server.
- [ ] Set up a Ansible Master. Plug it into the Foreman-Katello server for identifying the inventory it will need to work with.
- [ ] Deploy another VM. Install iscsitgt and nfs-kernel-server on it. Export a LUN and an NFS share.
- [ ] Deploy another VM. Install bakula on it, using the postgresql cluster to store its database. Register each machine on it, storing to flatfile. Store the bakula VM's image on the iscsi LUN, and every other machine on the NFS share.
- [ ] Deploy two more VMs. These will have httpd (Apache2) on them. Leave essentially default for now.
- [ ] Deploy two more VMs. These will have tomcat on them. Use JBoss Cache to replicate the session caches between them. Use the httpd servers as the frontends for this. The application you will run is JBoss Wiki.
- [ ] You guessed right, deploy another VM. This will do iptables-based NAT/round-robin loadbalancing between the two httpd servers.
- [ ] Deploy another VM. On this VM, install postfix. Set it up to use a gmail account to allow you to have it send emails, and receive messages only from your internal network.
- [ ] Deploy another VM. On this VM, set up a Nagios server. Have it use snmp to monitor the communication state of every relevant service involved above. This means doing a "is the right port open" check, and a "I got the right kind of response" check and "We still have filesystem space free" check.
- [ ] Deploy another VM. On this VM, set up a syslog daemon to listen to every other server's input. Reconfigure each other server to send their logging output to various files on the syslog server. (For extra credit, set up logstash or kibana or greylog to parse those logs.)
- [ ] Document every last step you did in getting to this point in your brand new Wiki.
- [ ] Now go back and setup Ansible Tower to ensure that every last one of these machines is authenticating to the LDAP servers, registered to the Foreman-Katello server, and backed up by the bakula server.
- [ ] Now go back, reference your documents, and set up a Puppet Razor profile that hooks into each of these things to allow you to recreate, from scratch, each individual server.
- [ ] Destroy every secondary machine you've created and use the above profile to recreate them, joining them to the clusters as needed.
- [ ] Bonus exercise: create three more VMs. A CentOS 7, and 8 machine. On each of these machines, set them up to allow you to create custom RPMs and import them into the Foreman-Katello server instance. Ensure your Ansible configurations work for all three and produce like-for-like behaviors.
