Title: Symfony continuous integration with Jenkins
Date: 2014-07-03 10:20
Category: Symfony
Tags: symfony, continuous integration, jenkins, testing
Slug: symfony-jenkins
Authors: jandro_es
Summary: 
Status: draft

This post will try to be a guide on how to configure Jenkins for continous integration on big Symfony projects, even with custom configurations and dependencies. As an example, these are the requirements:
<h3>Requirements</h3>
- A server box with at least 1 core and 1GB of RAM. I'd make it to work on a Digital Ocean's 512MB droplet, but it's a bit tight.

- The server is running Ubuntu 14.04 x64. At least 12.04 is required to follow up this guide.

- The server has a functional Apache2 installation.

- A Symfony project hosted on Github (private or public) or on you own git hosting.

- The project is using a version of Symfony enabled for composer.
<h3> Server setup</h3>
&nbsp;
<h4>Installing Composer</h4>
Make sure you have all the requirements installed:

[cc lang="bash"]sudo apt-get install php5 git php5-curl[/cc]

Inside you home folder download the composer file:
[cc lang="bash"]curl -sS https://getcomposer.org/installer | php[/cc]

Now move the downloaded file to a global directory so it's available to any user:
[cc lang="bash"]sudo mv composer.phar /usr/local/bin/composer[/cc]

Now you can execute composer globally on the server.
<h4>Installing PHPUNIT</h4>
Following the instructions on it's site, we only have to execute the following commands:
[cc lang="bash"]
sudo wget https://phar.phpunit.de/phpunit.phar
sudo chmod +x phpunit.phar
sudo mv phpunit.phar /usr/local/bin/phpunit
[/cc]
<h4>Installing PHP Mess detector</h4>
The preferring method to install <strong>phpmd</strong> is to use it's PEAR channel. First make sure you have PEAR on you system.
[cc lang="bash"]sudo apt-get install php-pear[/cc]
Once PEAR is correctly installed, we execute the following commands:
[cc lang="bash"]
sudo pear channel-discover pear.phpmd.org
sudo pear channel-discover pear.pdepend.org
sudo pear install phpmd/PHP_PMD
[/cc]
<h4>Installing PHP Depend</h4>
Now we need to install <strong>PHPDepend</strong> to gather metrics about the software:
[cc lang="bash"]
sudo wget http://static.pdepend.org/php/latest/pdepend.phar
sudo chmod +x pdepend.phar
sudo mv pdepend.phar /usr/local/bin/pdepend
[/cc]
<h4>Installing PHPCPD</h4>
We install <strong>PHP Copy/Paste Detector</strong> to monitor C&amp;P excessive usage:
[cc lang="bash"]
sudo wget https://phar.phpunit.de/phpcpd.phar
sudo chmod +x phpcpd.phar
sudo mv phpcpd.phar /usr/local/bin/phpcpd
[/cc]
<h4>Installing PHPLOC</h4>
We install <strong>phploc </strong>to measure the size and analise the structure of the projects:
[cc lang="bash"]
sudo wget https://phar.phpunit.de/phploc.phar
sudo chmod +x phploc.phar
sudo mv phploc.phar /usr/local/bin/phploc
[/cc]
<h4>Installing PHP_CodeBrowser</h4>
It will generate a nice interface to explore all our code:
[cc lang="bash"]sudo composer global require "mayflower/php-codebrowser=~1.1"[/cc]
<h4>Installing PHP_CodeSniffer</h4>
[cc lang="bash"]sudo pear install PHP_CodeSniffer-2.0.0a2[/cc]
Once installed, we need to add Symfony2 coding standar following this instructions <a title="https://github.com/opensky/Symfony2-coding-standard#installation" href="https://github.com/opensky/Symfony2-coding-standard#installation" target="_blank">https://github.com/opensky/Symfony2-coding-standard#installation</a>
<h4>Installing Jenkins</h4>
We need to add to add the key and source list to apt. We execute the following two commands:
[cc lang="bash"]
sudo wget -q -O - http://pkg.jenkins-ci.org/debian/jenkins-ci.org.key | apt-key add -
sudo echo deb http://pkg.jenkins-ci.org/debian binary/ &gt; /etc/apt/sources.list.d/jenkins.list
[/cc]
Now we update apt sources:
[cc lang="bash"]sudo apt-get update[/cc]
And finally we install Jenkins:
[cc lang="bash"]apt-get install jenkins[/cc]
<h4>Redirecting Apache to port 8080 (optional)</h4>
Jenkins by default runs on the port 8080, supposing that we have a subdomain called <strong>ci.domain.com</strong> pointing to the server we need to follow the next steps to redirect it directly to Jenkins:
[cc lang="bash"]
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo a2dissite default
[/cc]
Now we create a file called jenkins.conf on /etc/apache2/sites-available with the following:
[cc lang="apache"]
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        ServerName jenkins.domain.com
        ServerAlias jenkins
        ProxyRequests Off
        <Proxy *>
                Order deny,allow
                Allow from all
        </Proxy>
        ProxyPreserveHost on
        ProxyPass / http://localhost:8080/
</VirtualHost>

[/cc]
We enable the new virtual host:
[cc lang="bash"]
sudo a2ensite jenkins
[/cc]
And finally we restart Apache:
[cc lang="bash"]sudo service apache2 restart[/cc]
<h4>STARTING Jenkins</h4>
Jenkins now should be available on the url <strong>jenkins.domain.com</strong> and we'll should see the following screen:

[caption id="attachment_46" align="aligncenter" width="550"]<a href="http://www.filtercode.com/wp-content/uploads/2014/07/1.png"><img class="size-full wp-image-46" src="http://www.filtercode.com/wp-content/uploads/2014/07/1.png" alt="Jenkins start screen" width="550" height="284" /></a> Jenkins start screen[/caption]

Keep in mind that at this moment Jenkins is <strong>unsecure</strong>, we need to <strong>setup security as soon as possible</strong>.

The best guideline is this article on Jenkins' wiki <a title="https://wiki.jenkins-ci.org/display/JENKINS/Securing+Jenkins" href="https://wiki.jenkins-ci.org/display/JENKINS/Securing+Jenkins" target="_blank">https://wiki.jenkins-ci.org/display/JENKINS/Securing+Jenkins</a>
<h4> Setup your Symfony2 project to work with Jenkins</h4>
We need to create a build.xml file in the root of our project, this file will tell <strong>ANT </strong>which steps to follow when building our application. The following is an example of <em>buils.xml </em>file for a Symfony project:
[cc lang="xml"]
<?xml version="1.0" encoding="UTF-8"?>

<project name="Your project name" default="build">
  <property name="workspace" value="${basedir}" />
  <property name="sourcedir" value="${basedir}/src" />
  <property name="builddir" value="${workspace}/app/build" />

  <target name="build"
  depends="prepare,parameters,vendors,lint,phploc,pdepend,phpcpd,phpmd-ci,phpcs-ci,phpunit,phpcb"/>

  <target name="build-parallel" depends="prepare,lint,tools-parallel,phpunit,phpcb"/>

  <target name="tools-parallel" description="Run tools in parallel">
    <parallel threadCount="2">
      <sequential>
        <antcall target="pdepend"/>
        <antcall target="phpmd-ci"/>
      </sequential>
      <antcall target="phpcpd"/>
      <antcall target="phpcs-ci"/>
      <antcall target="phploc"/>
      <antcall target="phpdoc"/>
    </parallel>
  </target>

  <target name="clean" description="Cleanup build artifacts">
    <delete dir="${builddir}/api"/>
    <delete dir="${builddir}/code-browser"/>
    <delete dir="${builddir}/coverage"/>
    <delete dir="${builddir}/logs"/>
    <delete dir="${builddir}/pdepend"/>
    <delete dir="${builddir}/docs/*"/>
  </target>

  <target name="prepare" depends="clean" description="Prepare for build">
    <mkdir dir="${builddir}/api"/>
    <mkdir dir="${builddir}/code-browser"/>
    <mkdir dir="${builddir}/coverage"/>
    <mkdir dir="${builddir}/logs"/>
    <mkdir dir="${builddir}/pdepend"/>
  </target>

  <target name="lint" description="Perform syntax check of sourcecode files">
    <apply executable="php" failonerror="true">
    <arg value="-l" />
    <fileset dir="${sourcedir}">
      <include name="**/*.php" />
      <exclude name="Tests/**" />
      <exclude name="Resources/**" />
      <modified />
    </fileset>
    <fileset dir="${basedir}/src/">
      <include name="**/*Test.php" />
      <modified />
    </fileset>
    </apply>
  </target>

  <target name="phploc" description="Measure project size using PHPLOC">
    <exec executable="phploc">
    <arg value="--log-csv" />
    <arg value="${builddir}/logs/phploc.csv" />
    <arg value="--exclude"/>
    <arg value="${sourcedir}/Resources/**"/>
    <arg path="${sourcedir}" />
    </exec>
  </target>

  <target name="pdepend" description="Calculate software metrics using PHP_Depend">
    <exec executable="pdepend">
      <arg value="--jdepend-xml=${builddir}/logs/jdepend.xml" />
      <arg value="--jdepend-chart=${builddir}/pdepend/dependencies.svg" />
      <arg value="--overview-pyramid=${builddir}/pdepend/overview-pyramid.svg" />
      <arg value="--ignore=/Resources/**"/>
      <arg path="${sourcedir}" />
    </exec>
  </target>

  <target name="phpmd" description="Perform project mess detection using PHPMD and print human readable output. Intended for usage on the command line before committing.">
    <exec executable="phpmd">
      <arg path="${basedir}/src" />
      <arg value="text" />
      <arg value="${workspace}/app/phpmd.xml" />
    </exec>
  </target>

  <target name="phpmd-ci" description="Perform project mess detection using PHPMD creating a log file for the continuous integration server">
    <exec executable="phpmd">
      <arg path="${sourcedir}" />
      <arg value="xml" />
      <arg value="${workspace}/app/phpmd.xml" />
      <arg value="--reportfile" />
      <arg value="${builddir}/logs/pmd.xml" />
    </exec>
  </target>

  <target name="phpcs" description="Find coding standard violations using PHP_CodeSniffer and print human readable output. Intended for usage on the command line before committing.">
    <exec executable="phpcs">
      <arg value="--standard=Symfony2" />
      <arg path="${sourcedir}" />
    </exec>
  </target>

  <target name="phpcs-ci" description="Find coding standard violations using PHP_CodeSniffer creating a log file for the continuous integration server">
    <exec executable="phpcs" output="/dev/null">
      <arg value="--report=checkstyle" />
      <arg value="--report-file=${builddir}/logs/checkstyle.xml" />
      <arg value="--standard=Symfony2" />
      <arg path="${sourcedir}" />
    </exec>
  </target>

  <target name="phpcpd" description="Find duplicate code using PHPCPD">
    <exec executable="phpcpd">
      <arg value="--log-pmd" />
      <arg value="${builddir}/logs/pmd-cpd.xml" />
      <arg path="${sourcedir}" />
    </exec>
  </target>

  <target name="phpdoc" description="Generate API documentation using phpDox">
    <exec executable="phpdoc">
      <arg line="-d '${sourcedir}' -t '${builddir}/docs' --title='Tempo' " />
    </exec>
  </target>

  <target name="phpunit" description="Run unit tests with PHPUnit">
    <exec executable="phpunit" failonerror="true">
      <arg value="-c" />
      <arg path="${basedir}/app/phpunit.xml" />
    </exec>
  </target>

  <target name="phpcb" description="Aggregate tool output with PHP_CodeBrowser">
    <exec executable="phpcb">
      <arg value="--log" />
      <arg path="${builddir}/logs" />
      <arg value="--source" />
      <arg path="${sourcedir}" />
      <arg value="--output" />
      <arg path="${builddir}/code-browser" />
    </exec>
  </target>

  <target name="parameters" description="Copy parameters">
    <exec executable="cp" failonerror="true">
        <arg path="app/config/parameters.ini.dist" />
        <arg path="app/config/backend/parameters.ini" />
    </exec>
  </target>

  <target name="vendors" description="Update vendors">
    <exec executable="php" failonerror="true">
        <arg value="composer.phar" />
        <arg value="install" />
    </exec>
  </target>
</project>
[/cc]
&nbsp;
You'll need to adjust it to the requirements of your project, nevertheless remember to have parameters.ini.dist within the project configuration on you repository.
Next we need to create a folder on app/ called <strong>build</strong> where the generated files from the different analyzers will be generated.
We also need to create two more files <strong>phpunit.xml</strong>, the configuration for PHPUnit and <strong>phpmd.xml</strong> for PHPMessDetector. We'll place these files on the folder <i>app/</i>.
&nbsp;
<strong>phpunit.xml</strong>
[cc lang="xml"]
<?xml version="1.0" encoding="UTF-8"?>

<!-- http://www.phpunit.de/manual/current/en/appendixes.configuration.html -->
<phpunit
  backupGlobals               = "false"
  backupStaticAttributes      = "false"
  colors                      = "true"
  convertErrorsToExceptions   = "true"
  convertNoticesToExceptions  = "true"
  convertWarningsToExceptions = "false"
  processIsolation            = "false"
  stopOnFailure               = "false"
  syntaxCheck                 = "false"
  bootstrap                   = "bootstrap.php.cache" >

    <testsuites>
        <testsuite name="Project Test Suite">
            <directory>../src/*/*Bundle/Tests</directory>
            <directory>../src/*/Bundle/*Bundle/Tests</directory>
        </testsuite>
    </testsuites>

    <logging>
        <log type="coverage-html" target="build/coverage" title="BankAccount" charset="UTF-8" yui="true" highlight="true"
       lowUpperBound="35" highLowerBound="70"/>
        <log type="coverage-clover" target="    build/logs/clover.xml"/>
        <log type="junit" target="build/logs/junit.xml" logIncompleteSkipped="false"/>
    </logging>


    <filter>
        <whitelist>
            <directory>../src</directory>
            <exclude>
                <directory>../src/*/*Bundle/Resources</directory>
                <directory>../src/*/*Bundle/Tests</directory>
                <directory>../src/*/Bundle/*Bundle/Resources</directory>
                <directory>../src/*/Bundle/*Bundle/Tests</directory>
            </exclude>
        </whitelist>
    </filter>
</phpunit>
[/cc]
&nbsp;
<strong>phpmd.xml</strong>
[cc lang="xml"]
<?xml version="1.0"?>
<ruleset name="Symfony2 ruleset" xmlns="http://pmd.sf.net/ruleset/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://pmd.sf.net/ruleset/1.0.0 http://pmd.sf.net/ruleset_xml_schema.xsd" xsi:noNamespaceSchemaLocation="http://pmd.sf.net/ruleset_xml_schema.xsd">
    <description>
        Custom ruleset.
    </description>

    <rule ref="rulesets/design.xml" />
    <rule ref="rulesets/unusedcode.xml" />
    <rule ref="rulesets/codesize.xml" />
    <rule ref="rulesets/naming.xml/ShortVariable">
        <property name="xpath">
            <value>
                //VariableDeclaratorId[(string-length(@Image) &lt; 3) and (not (@Image='id')) and (not (@Image='em'))]
                [not(ancestor::ForInit)]
                [not((ancestor::FormalParameter) and (ancestor::TryStatement))]
            </value>
        </property>
    </rule>

</ruleset>
[/cc]
&nbsp;
With those files and folders, we're ready to configure our first job on Jenkins.

<h4>Setting up Jenkins for our project</h4>
In <strong>New job</strong> select the option <strong>Build a free-style software project</strong>, in there configure the name, description, per project security if you need it, etc..
&nbsp;
Now the "tricky" part:
<h5>Configure GITHUB</h5>
Similar steps need to be done for other SCMs platforms like Bitbucket, etc..
