Securing Apache 2 With ModSecurity

What is ModSecurity?
ModSecurity is a free and open source web application that started out as an Apache module and grew to a fully-fledged web application firewall. It works by inspecting requests sent to the web server in real time against a predefined rule set, preventing typical web application attacks like XSS and SQL Injection.

rerequisites & Requirements
In order to install and configure ModSecurity, you need to have a Linux server with the following services running:

Apache 2

Installing ModSecurity

1- ModSecurity can be installed by running the following command in your terminal:

            sudo apt install libapache2-mod-security2 -y

2- Alternatively, you can also build ModSecurity manually by cloning the official ModSecurity Github repository.

3- After installing ModSecurity, enable the Apache 2 headers module by running the following command:

           sudo a2enmod headers

4- After installing ModSecurity and enabling the header module, you need to restart the apache2 service, this can be done by running the following command:

            sudo systemctl restart apache2

You should now have ModSecurity installed. The next steps involves enabling and configuring ModSecurity and the OWASP-CRS.

                      ####  Configuring ModSecurity

ModSecurity is a firewall and therefore requires rules to function. This section shows you how to implement the OWASP Core Rule Set. First, you must prepare the ModSecurity configuration file.

1- Remove the .recommended extension from the ModSecurity configuration file name with the following command:

            sudo cp /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf

2- With a text editor such as vim, open /etc/modsecurity/modsecurity.conf and change the value for SecRuleEngine from DetectionOnly to On:

File: /etc/modsecurity/modsecurity.conf
SecRuleEngine On
Restart Apache to apply the changes:

                 sudo systemctl restart apache2

ModSecurity should now be configured to run. The next step in the process is to set up a rule set to actively prevent your web server from attacks.

Setting Up the OWASP ModSecurity Core Rule Set

The OWASP ModSecurity Core Rule Set (CRS) is a set of generic attack detection rules for use with ModSecurity or compatible web application firewalls. The CRS aims to protect web applications from a wide range of attacks, including the OWASP Top Ten, with a minimum of false alerts. The CRS provides protection against many common attack categories, including SQL Injection, Cross Site Scripting, and Local File Inclusion.

To set up the OWASP-CRS, follow the procedures outlined below.

1- First, delete the current rule set that comes prepackaged with ModSecurity by running the following command:

            sudo rm -rf /usr/share/modsecurity-crs

2- Ensure that git is installed:

            sudo apt install git

3- Clone the OWASP-CRS GitHub repository into the /usr/share/modsecurity-crs directory:

            sudo git clone https://github.com/coreruleset/coreruleset /usr/share/modsecurity-crs

4- Rename the crs-setup.conf.example to crs-setup.conf:

            sudo mv /usr/share/modsecurity-crs/crs-setup.conf.example /usr/share/modsecurity-crs/crs-setup.conf

5- Rename the default request exclusion rule file:

            sudo mv /usr/share/modsecurity-crs/rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf.example /usr/share/modsecurity-crs/rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf

You should now have the OWASP-CRS setup and ready to be used in your Apache configuration.

                                      Enabling ModSecurity in Apache 2

To begin using ModSecurity, enable it in the Apache configuration file by following the steps outlined below:

1- Using a text editor such as vim, edit the /etc/apache2/mods-available/security2.conf file to include the OWASP-CRS files you have downloaded:

        File: /etc/apache2/mods-available/security2.conf

                SecDataDir /var/cache/modsecurity
                Include  /usr/share/modsecurity-crs/crs-setup.conf
                Include /usr/share/modsecurity-crs/rules/*.conf

In /etc/apache2/sites-enabled/example.com.conf file VirtualHost block, include the SecRuleEngine directive set to On.

             File: /etc/apache2/sites-enabled/example.com.conf

<VirtualHost \*:80> ServerAdmin webmaster@localhost DocumentRoot /var/www/html

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    SecRuleEngine On

If you are running a website that uses SSL, add `SecRuleEngine` directive to that website's configuration file as well. See our guide on [SSL Certificates with Apache on Debian & Ubuntu](/docs/guides/ssl-apache2-debian-ubuntu/#configure-apache-to-use-the-ssl-certificate) for more information.

Restart the apache2 service to apply the configuration:
sudo systemctl restart apache2

ModSecurity should now be configured and running to protect your web server from attacks. You can now perform a quick test to verify that ModSecurity is running.

Testing ModSecurity

        curl http://<SERVER-IP/DOMAIN>/index.html?exec=/bin/bash


        ```output

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access this resource.</p>
<hr>
<address>Apache/2.4.41 (Ubuntu) Server at 172.105.53.220 Port 80</address>
</body></html>
```
