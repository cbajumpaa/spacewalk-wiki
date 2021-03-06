# Spacewalk Installation Instructions



 * [Setting up Spacewalk repo ]()
 * [Additional repos & packages ]()
 * [Oracle Pre-Requisites ]()
 * [Installing Spacewalk ]()
 * [Managing Spacewalk ]()
 * [Known Issues ]()
## Prerequisites



 * Outbound open ports 80, 443, 4545 (only if you want to enable monitoring)
 * Inbound open ports 80, 443, 5222 (only if you want to push actions to client machines) and 5269 (only for push actions to a Spacewalk Proxy)
 * Storage for database: 128KB per client system + 64MB per channel
 * Storage for packages (default /var/satellite): Depends on what you're storing; Red Hat recommend 6GB per channel for their channels
 * 2GB RAM minimum, 4GB recommended
 * If you use LDAP as a central identity service and wish to pull user and group information from it, see [[https://fedorahosted.org/spacewalk/wiki/SpacewalkWithLDAP]]
# Setting up Spacewalk repo



RPM downloads of the project are available through a yum repository

  * http://spacewalk.redhat.com/yum - Binary RPMs
  * http://spacewalk.redhat.com/source - Source RPMs and tarballs

To use this repository install spacewalk-repo with commands below:
### Red Hat Enterprise Linux, CentOS




    rpm -Uvh http://spacewalk.redhat.com/yum/0.7/RHEL/5/i386/spacewalk-repo-0.7-4.el5.noarch.rpm
### Fedora 11




    rpm -Uvh http://spacewalk.redhat.com/yum/0.7/Fedora/11/i386/spacewalk-repo-0.7-4.fc11.noarch.rpm
### Fedora 12




    rpm -Uvh http://spacewalk.redhat.com/yum/0.7/Fedora/12/i386/spacewalk-repo-0.7-4.fc12.noarch.rpm
### Nightly builds



If you want to use the nightly builds, add a .repo file pointing to the nightly repository:


    cat > /etc/yum.repos.d/spacewalk.repo << 'EOF'
    [spacewalk]
    name=Spacewalk
    baseurl=http://miroslav.suchy.cz/spacewalk/nightly-candidate/$basearch/os/
    gpgkey=http://spacewalk.redhat.com/yum/RPM-GPG-KEY-spacewalk
    enabled=1
    gpgcheck=0
    EOF

*NOTE:* if yum install or update nags about conflict on "file /usr/bin/ipv4calc" enable the "epel-testing" repo in /etc/yum.repos.d/epel-testing.repo

*NOTE:* Nigthly repo contains developers snapshot and is not suitable in production environment.

Fedora 12 nightly repo is at:

{{{ 
http://miroslav.suchy.cz/spacewalk/nightly-candidate-f12/
}}}

Fedora 11 nightly repo is at:

{{{ 
http://miroslav.suchy.cz/spacewalk/nightly-candidate-f11/
}}}
# Additional repos & packages

### Fedora




For Spacewalk on Fedora, a couple of additional dependencies are needed from jpackage. Please create the following yum repository before beginning your Spacewalk installation:


    cat > /etc/yum.repos.d/jpackage.repo << EOF
    [jpackage]
    name=jpackage-f10
    baseurl=http://mirrors.dotsrc.org/jpackage/6.0/fedora-10/free/
    gpgkey=http://www.jpackage.org/jpackage.asc
    gpgcheck=1
    EOF

Note: The f10 is intentional here even if you have newer version of Fedora -- there simply are not newer jpackage repos.
### Red Hat Enterprise Linux 5



Spacewalk requires a Java Virtual Machine with version 1.6.0 or greater.  [EPEL - Extra Packages for Enterprise Linux](http://fedoraproject.org/wiki/EPEL) contains a version of the openjdk that works with Spacewalk.  To get packages from EPEL just install this RPM:


    rpm -Uvh http://download.fedora.redhat.com/pub/epel/5/i386/epel-release-5-3.noarch.rpm
or

    rpm -Uvh http://download.fedora.redhat.com/pub/epel/5/x86_64/epel-release-5-3.noarch.rpm

Please make sure you have only one java version installed. 
Otherwise you get error messeges like :
 * " bad major version at off set=6" or 
 * "INFO: validateJarFile(/usr/share/tomcat5/webapps/rhn/WEB-INF/lib/jspapi.jar) - jar not loaded."
### CentOS 5



Follow the instructions for *Red Hat Enterprise Linux 5* with the additions:
 * Necessary packages rhn-client-tools and rhnlib were removed from CentOS, they can be found in spacewalk-client repo. Setup it by installing spacewalk-client-repo package.

    rpm -Uvh http://spacewalk.redhat.com/yum/0.7/RHEL/5/i386/spacewalk-client-repo-0.7-4.el5.noarch.rpm
 *  Import Redhat's RPM GPG key:

    # wget -O /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release http://www.redhat.com/security/37017186.txt
    # rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
# Oracle Pre-Requisites



In order to get Spacewalk to run you need the Oracle client and an Oracle 10g database server. The easiest way to do this is to install Oracle XE and the Oracle Instant Client. Instructions on how to do this can be found on the [Oracle 10g Express Edition Setup](OracleXeSetup) page.
# Installing Spacewalk



Just ask yum to install the necessary packages. This will pull down and install the set of RPMs required to get Spacewalk to run.

*0.7 installation:*

If you tend to use the Oracle backend:

    yum install spacewalk-oracle 

If you prefer the PostgreSQL backend:

    yum install spacewalk-postgresql 

*Note:* PostgreSQL support is in early development stage. You may want to take a look at the [PostgreSql Project](PostgreSqlProject) page for further information.
# Configuring Spacewalk



*Note*: Your Spacewalk server should have a resolvable FQDN such as 'hostname.domain.com'.  If the installer complains that the hostname is not the FQDN, do not use the --skip-fqdn-test flag to skip !  

Also, this will not work unless the spacewalk account has a password.  You can set up a password by logging in to sqlplus with the sys account and using the command 


    PASSWORD user_name

where user_name = spacewalk.

Once the Spacewalk RPM is installed you need to configure the application, you can run:


    spacewalk-setup --disconnected



An example session is as follows:


    [root@fjs-0-02 yum.repos.d]# spacewalk-setup --disconnected
    * Setting up Oracle environment.
    * Setting up database.
    ** Database: Setting up database connection.
    DB User? spacewalk
    DB Password? 
    DB SID? XE
    DB hostname? localhost
    DB port [1521]? 
    DB protocol [TCP]? 
    ** Database: Testing database connection.
    ** Database: Populating database.
    *** Progress: ##########################################################
    * Setting up users and groups.
    ** GPG: Initializing GPG and importing key.
    ** GPG: Creating /root/.gnupg directory
    You must enter an email address.
    Admin Email Address? tito@example.com
    * Performing initial configuration.
    * Activating Spacewalk.
    ** Loading Spacewalk Certificate.
    ** Verifying certificate locally.
    ** Activating Spacewalk.
    * Enabling Monitoring.
    * Configuring apache SSL virtual host.
    Should setup configure apache's default ssl server for you (saves original ssl.conf) y/n? y
    ** /etc/httpd/conf.d/ssl.conf has been backed up to ssl.conf-swsave
    * Creating SSL certificates.
    CA certificate password? 
    Re-enter CA certificate password? 
    Organization? Tito Puente Solutions
    Organization Unit [fjs-0-02.rhndev.redhat.com]? 
    Email Address [jesusr@redhat.com]? 
    City? Raleigh
    State? NC
    Country code (Examples: "US", "JP", "IN", or type "?" to see a list)? US
    ** SSL: Generating CA certificate.
    ** SSL: Deploying CA certificate.
    ** SSL: Generating server certificate.
    ** SSL: Storing SSL certificates.
    * Deploying configuration files.
    * Update configuration in database.
    * Setting up Cobbler..
    Cobbler requires tftp and xinetd services be turned on for PXE provisioning functionality. Enable these services (y/n, default = 'y')?y
    * Restarting services.
    Installation complete.
    Visit https://fjs-0-02.rhndev.redhat.com to create the Spacewalk administrator account.

If you are on x86_64 and tomcat5 fails to restart you need todo a:


    cd /var/lib/tomcat5/webapps/rhn/WEB-INF/lib &&\
       rm ojdbc14.jar &&\
       ln -s /usr/lib/oracle/10.2.0.4/client64/lib/ojdbc14.jar ojdbc14.jar;

After `spacewalk-setup` is complete your application is ready to go!
*Be sure to use the FQDN of the host when you connect.*

*NOTE:* If you notice you missing some buttons and channels that have been created aren't visible in you UI edit `/etc/sysconfig/httpd` and change:

    export ORACLE_HOME=/opt/oracle
    export NLS_LANG=english.AL32UTF8
to

    export ORACLE_HOME=/usr/lib/oracle/xe/app/oracle/product/10.2.0/server
    export NLS_LANG=AMERICAN_AMERICA.AL32UTF8

Once you've made sure you can login to the Spacewalk web UI, you can then proceed to the next step: [Uploading Content](UploadFedoraContent).
# Managing Spacewalk

Spacewalk consists of several services. Each of them has its own init.d script to stop/start/restart. If you want manage all spacewalk services at once use 


    /usr/sbin/rhn-satellite [stop|start|restart].
# Known Issues



 * For issues with Oracle XE, see [Oracle XE Setup Known Issues](OracleXeSetup)