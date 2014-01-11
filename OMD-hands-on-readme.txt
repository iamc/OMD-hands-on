******************************************
HPCK'14 OMD hands-on virtual machine setup
******************************************

In this hands-on tutorial we will start from two bare CentOS VirtualBox virtual
machines and carry out a step by step procedure which will take us from a zero
configuration to a full monitoring system with email notifications, graphs,
trends, etc. where one of the machines will become the Nagios/OMD server
monitoring the status of both of the "target" machine and itself. Detailed
notes for the procedure will be provided.

For the hands-on sessions itself, for those interested in following along in
their laptops, I have already gone through part of the tutorial in order to
ease the initial steps and to avoid downloading packages etc. I have
specifically carried out the following steps:


In the OMD server (user/password: centos/reverse root/reverse):

  * setup internal network static IP address (192.168.56.10)
  * activate sshd service
  * setup posfix for Gmail SMTP service relay (with dummy credentials)
  * download and install OMD package and dependencies
  * enable http incoming connection in the firewall and set up selinux policy
  * download (but not installed) check_mk agent 


In the OMD "target" (user/password: root/reverse):

  * setup internal network static IP address (192.168.56.11)
  * download (but not installed) check_mk agent 


Anyway an overview of all the steps will be given.

You can download the virtual machines from http://goo.gl/rr4VrO


Also, a temporary Gmail account has been created for you to a)download the
pre-installed virtual machines and also to b)be used for those who do not have or
do not want to use another Gmail account for the email test notifications.
Credentials for this account  will be provided in the hands-on session.

If you plan to follow the tutorial along please test the virtual machines
making sure that you enable a "Host-only" network in your VirtualBox server and
that once booted machines can see each other eg. through ssh.


Bests,

Inigo Aldazabal
