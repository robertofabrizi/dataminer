# [![Application icon](https://lh4.googleusercontent.com/OeT98FtkuEHTDrN6xr3m2nkpF04-p_U-k83d-nDl4mf-qgzK_q2floX_OG5ZGBEz3a97LdAYAO7bov0=w1920-h950)][icon]
[icon]: https://lh4.googleusercontent.com/OeT98FtkuEHTDrN6xr3m2nkpF04-p_U-k83d-nDl4mf-qgzK_q2floX_OG5ZGBEz3a97LdAYAO7bov0=w1920-h950

# Maas - Dataminer
[![Gem Version](https://img.shields.io/gem/v/t.svg)][gem]
[![Build Status](https://img.shields.io/travis/sferik/t.svg)][travis]
[![Dependency Status](https://img.shields.io/gemnasium/sferik/t.svg)][gemnasium]
[![Coverage Status](https://img.shields.io/coveralls/sferik/t.svg)][coveralls]
[![tip for next commit](https://tip4commit.com/projects/102.svg)](https://tip4commit.com/github/sferik/t)

[gem]: https://rubygems.org/gems/t
[travis]: https://travis-ci.org/sferik/t
[gemnasium]: https://gemnasium.com/sferik/t
[coveralls]: https://coveralls.io/r/sferik/t

#### The data extraction tool of the Maas platform.
The dataminer is composed of three main parts:
* One core: inspired by both the Swing and Spring frameworks, it manages the lifecycle of the application. In particular, it is in charge of:
  * reading the config.properties file and use it as its base configuration
  * read the config.xml file to decide which connectors to start
  * receive the data extracted from each connector and decide if and what to do with it (basically, if there is any model for it)
  * offers the following features to model instances: be persisted on a target database, be sent to a target Jms Topic, be sent via email, be sent to a jtec server
  * offers a Jmx and REST interface for management
  * ...
* N connectors: each connector knows how to extract data from a target source and technology, but doesn't know what to look for, therefore it can be reused
* M models: a model for each business even of interest. A model receives from the core an entry that it's in charge of managing, and must implement at the very least the **parse()** method, that takes such input and parses it assigning the insteresting parts to instance variables. It can then signal the core that it wants to be persisted, sent via email, etc. 

Usually, the workflow begins by creating the new model classes, then create the configuration files and go ahead with the deployment.

## Dependencies
First, make sure you have Java6+ JDK installed.

    java -version

If the output looks like this, you need to install Java or add it to your environment: 
http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html

    java: command not found

If your version of Java is before Java6 OR you only have the JRE installed, it must be upgraded at the very least to Java6 SDK.

Although not necessary, it is strongly recommended to download and install the latest version of the Netbeans IDE (Java SE version):
https://netbeans.org/downloads/

## Configuration
Create a new basic Java project, and import the following required compile time dependencies:
	
	log4j-1.2.16.jar or higher
	monitoraggioV4.0.2.jar (the dataminer core jar)
	
## Development of a very basic model class
Your model class should extend the basic model implementation provided by the core, called `BaseModel`.
`BaseModel` has an abstract `parse()` method that must be implemented; moreover, two constructors are needed, a noargs ctor used by Hibernate, and one that takes a String as the single input parameter.
This String is the data read by the core via a connector, and passed to this model.
Let's create a first, basic model class:

```
package com.rhad.dataminer.model.implementations.test;

import com.rhad.dataminer.model.superclasses.BaseModel;
import java.text.ParseException;

/**
 * This class doesn't model any table, it just prints what it has received from the core as its input.
 * @author Roberto Fabrizi
 */
public class TestModel extends BaseModel {

    private String inputToParse;    

    @Override
    public void parse() throws ParseException {  
        super.getLogger().debug("The received input is: "+this.inputToParse);
    }
    
    public TestModel() {
    }

    public TestModel(String input) {
        this.inputToParse=input;
    }
}
```

This is it for our very first model implementation!

## Configuring the dataminer
Usually, it is a good idea to start the configuration from the **config.properties** file.
It should look like this:
```
# The name of the class to use to create an InitialContext object. This parameter is mandatory if you want to use JMS publishing.
# If unspecified, it will default to org.jnp.interfaces.NamingContextFactory
#initial_context_factory=org.jnp.interfaces.NamingContextFactory

# The JNDI server URL. This parameter is mandatory if you want to use JMS publishing.
#provider_url=jnp://ec2-....eu-central-1.compute.amazonaws.com:1099

# The name of the persistence-unit to use (must be defined in the persistence.xml file). This parameter is mandatory.
#entity_manager_factory=mysqlamazontest

# The port on which the HTML-JMX is started. This parameter is optional. 
# The default is -1, which means that no HTML-JMX adapter should be started
#html_jmx_adapter_port=8087

# The username used by the HTML-JMX adapter. This parameter is optional.
#html_jmx_adapter_user=administrator

# The password used by the HTML-JMX adapter. This parameter is optional.
#html_jmx_adapter_password=klee.74@

# The maximum number of concurrent threads that the ResourceManager can start to parse Resource instances. This parameter is optional.
# The default is -1, which means that there is no limit to the number of concurrent threads that can be started.
# thread_limit=5

# The name of the application's certificate to use to determine if the application has expired. This parameter is mandatory.
certificate_file=dataminerV4.jks

# The path on the user's filesystem of the application's certificate to use to determine if the application has expired. This parameter is optional. 
# The default is the application's certificate folder
# certificate_path=/sw/monitoraggio/monitoraggioV4/certificate/dataminer.jks
```
The default values for basically each property should be fine for now. The next file to configure is the **config.xml** file. This configuration file is the most important one, because it tells the core what it should do. Let's see an example of it:

```
<?xml version="1.0" encoding="UTF-8"?>
<!-- Allowed tags
        <resources>, mandatory, must be the enclosing tags of this xml
                <resource>, at least one is mandatory, use this tag to create a new resource of interest
                        <resource-file>, each resource must have a resource-file, the source of the data to monitor
                        <properties-file>, optional, if the parser needs extra data to run
                        <pattern-file>, each parser on a resource must have a way to tell which data is of interest
                        <resource-name>, optional, the name of the parser
                        <resource-type>, whether the parser is a static one, or a watcher must be created
                        <parser-class>, the name of the class to use to parse the resource-file
			<xml-schema>, can be omitted when JMS isnt needed, the XML-Schema to use to marshall this Resource
			<jaxb-class>, can be omitted when JMS isnt needed, the class to be used by JAXB to marshall this Resource
-->
<resources>
	<resource>
		<resource-file>/your_base_path/java/monitoraggioV4/config/test.sql</resource-file>
		<pattern-file>/your_base_path/java/monitoraggioV4/config/test.xml</pattern-file>
		<properties-file>/your_base_path/java/monitoraggioV4/config/test.properties</properties-file>
		<resource-name>Db_TestQuerier</resource-name>
		<resource-type>static</resource-type>
		<parser-class>com.rhad.dataminer.parser.implementations.db.persistent.DbReader</parser-class>
	</resource>
</resources>
```

This is telling the core to start:
* one connector, of type **com.rhad.dataminer.parser.implementations.db.persistent.DbReader**
* this connector accepts some configuration properties (each connector supports or needs different properties, check its documentation to find them out), specified in the **/your_base_path/java/monitoraggioV4/config/test.properties** file. In this example, the following properties are defined:
	* PERSISTENCE_UNIT=amazon_mysql_testdb
	* TIME_BETWEEN_QUERIES=600000
* being a database connector that uses JPA to abstract itself from the specific DBMS vendor, there must be a **persistence.xml** file inside a **META-INF** folder. In said file there can be multiple persistence units defined, one of which has to be called **amazon_mysql_testdb**, as requested by the **/your_base_path/java/monitoraggioV4/config/test.properties** file. This is the target database instance for the connector.
* as per the **com.rhad.dataminer.parser.implementations.db.persistent.DbReader** connector documentation, it periodically reloads the file **/your_base_path/java/monitoraggioV4/config/test.sql** that contains one or more queries, and fires it/them.
* the connector passes the result of the query(ies) to the core, which uses the xml file **/your_base_path/java/monitoraggioV4/config/test.xml** to decide if the returned data is of interest. For example, let's add the following query to it: `SELECT SYSDATE();`
* the file **/your_base_path/java/monitoraggioV4/config/test.xml** is basically a collection of trees where each node is a String or regex against which the extracted data is compared. The leaves of the trees must be model classes. Assuming that a path from the node of a tree to one of its leaves is all matched agaist the input data, then this data is passed to the String ctor of the model class mapped by the leaf. For example:
```
<models>
	<root value="04">
		<node1 value="1999" />
                        <node2 value="01" />
                                <leaf value="com.rhad.dataminer.model.implementations.test.OlderTestModel" />
		<node1 value="2000" />
                        <node2 value="01" />
                                <leaf value="com.rhad.dataminer.model.implementations.test.AnotherOlderTestModel" />
	</root>
	<root value="2017">
		<leaf value="com.rhad.dataminer.model.implementations.test.TestModel" />
	</root>
	<root value="2018">
		<leaf value="com.rhad.dataminer.model.implementations.test.FutureTestModel" />
	</root>
</models>
```
* this means that the connector executes the query `SELECT SYSDATE()` and returns its result back to the core. The core compares the result of the query, agaist the root of the first tree. Let's assume that the query returned `14-04-2017`, this is compared against the String `04` first; `04` is included in the current String of interest, therefore the core goes one level below. `1999` isn't a substring of `14-04-2017`, therefore this part of the first tree is discarded, and the core never tests if `01` is a substring of the input String or not. At the same level of the already discarded `1999` there is `2000`, which is also discarded, therefore the whole first tree of root `04` is ignored. The core goes to the second tree, and tests `2017` against the input String. Since `2017` is included, it goes inside that tree, and reaches the one and only leaf right away, thus passing `14-04-2017` to its String ctor. Once the core finds one leaf, it considers the input String handled, therefore it never goes ahead to test it against the root of the third tree, `2018`.
* the core, after having instantiated a **com.rhad.dataminer.model.implementations.test.TestModel** passing to it the input String `14-04-2017`, proceeds to call its `parse()` method, that in our test example previously defined should end printing a log entry stating: *The received input is: 14-04-2017*.

Two more configuration files are needed, the **log4j** logging configuration and the **c3p0** connection pool configuration.

An example of the first could be:
```
log4j.rootLogger=INFO, rootAppender
log4j.appender.rootAppender=org.apache.log4j.DailyRollingFileAppender
log4j.appender.rootAppender.Append=false
log4j.appender.rootAppender.File=${log_path}/test.log
log4j.appender.rootAppender.threshold=TRACE
log4j.appender.rootAppender.layout=org.apache.log4j.PatternLayout
log4j.appender.rootAppender.layout.ConversionPattern=%d{ABSOLUTE} [%t] [%-5p] %C{2} %x - %m%n
log4j.logger.com.rhad=TRACE
```

while the second could look like this:
```
#Managing Pool Size and Connection Age 
#c3p0.maxConnectionAge=172800 
c3p0.maxConnectionAge=14400
#c3p0.maxIdleTime=1800

#Configuring Connection Testing: must be disabled if one of the queried databases has read only priviledges, 
#as c3p0 will try to create it on all target persistence units. 
#Use persistence unit specific preferredTestQuery that make sense instead (this property would override those and make them unused)
c3p0.automaticTestTable=C3P0TestTable
c3p0.testConnectionOnCheckin=true

#Configuring Statement Pooling: some databases and/or JDBC drivers, most notably Oracle, do not handle 
#the case well and freeze, leading to deadlocks. Setting this parameter to a positive value should eliminate the issue. 
#This parameter should only be set if you observe that attempts by c3p0 to close() cached statements freeze 
#(usually you'll see APPARENT DEADLOCKS in your logs). If set, this parameter should almost always be set to 1
c3p0.statementCacheNumDeferredCloseThreads=1
c3p0.maxStatementsPerConnection=50

#Configuring Recovery From Database Outages
c3p0.acquireRetryDelay=10000
#c3p0.acquireRetryDelay=300000

#Other DataSource Configuration  
#maxAdministrativeTaskTime Default: 0
#numHelperThreads Default: 3
c3p0.numHelperThreads=5
```

## Dataminer filesystem structure
This is the structure of the dataminer on a filesystem:
* **bin**: this folder contains the various scripts to start the dataminer, plus a properties file to set some configurations
* **certificate**: this folder contains the certificate that validates the dataminer. After its expiration date, the dataminer won't be able to run anymore unless this certificate is updated
* **config**: the folder containing all the configuration files
* **connector_lib**: the folder containing most/all the connector jar plus their required dependencies
* **deployment**: mostly unused
* **elab**: mostly unused
* **framework_lib**: the folder containing all the dataminer dependencies
* **installation_lib**: the folder containing the variable part of the code, for example the jar of the custom models plus their dependencies
* **log**: the folder that should contain the produced logs

Let's assume that, following this readme, we produced some new files, located as follows:
* c3p0.properties: **config/META-INF** folder
* persistence.xml: **config/META-INF** folder
* log4j.properties: **config** folder
* test.properties: **config** folder
* test.sql: **config** folder
* test.xml: **config** folder
* dataminer-plugin-model-test.jar: **installation_lib** folder

## Dataminer process management
The dataminer comes with a few management scripts in the **bin** folder.

**On a Linux/Unix system**, the scripts to manage the dataminer are the ones that contain the launcher substring, not the ones that contain the dataminer one. The reason is that the dataminer script starts the Java process **in foreground**, and then waits for it to terminate. If it terminates with a return code that is different from 0, it re-exports the classpath and restarts the Java process again. This is useful both in events of a crash, both to handle a stop/restart request via the Jmx/REST API. The launcher script is therefore in charge of management, and accepts a few input parameters, that can be listed by calling it without passing anything to it. For example, `./bash_launcher.sh` outputs *Usage: ./bash_launcher.sh { start | restart | stop | status | pid }*.

* `bash_launcher.sh`: manages the dataminer on a Linux/Unix system using the `/bin/bash` shell
* `bash_dataminer.sh`: fires the Java process. Do not use it directly
* `bash_run.properties`: the properties to use when launching the Dataminer via the `bash_launcher.sh` script
* `ksh_launcher.sh`: manages the dataminer on a Linux/Unix system using the `/usr/bin/ksh` shell
* `ksh_dataminer.sh`: fires the Java process. Do not use it directly
* `ksh_run.properties`: the properties to use when launching the Dataminer via the `ksh_launcher.sh` script

**On a Windows system running cygwin**, the scripts are almost identical to the Linux/Unix bash scripts, with some exceptions to make them work in a cygwin environment. For example, the ps command in a cygwin environment must be enforced to list Windows processes that run outside of it. Moreover, the classpath has to be exported in a Linux/Unix format even thought it is running on a Windows machine and on a virtual mount point, therefore the command `-cp "$CLASSPATH"` becomes `-cp "$(cygpath -pw "$CLASSPATH")"`.

* `cygwin_launcher.sh`: manages the dataminer on a Windows system and cygwin shell using the `/bin/bash` shell
* `cygwin_dataminer.sh`: fires the Java process. Do not use it directly
* `cygwin_run.properties`: the properties to use when launching the Dataminer via the `cygwin_launcher.sh` script

**On a Windows system**, there is only script at the moment, that must be edited with the proper configuration parameters before being fired.

* `windows_dataminer.bat`: fires the Java process

**Inside a Docker container**, Docker containers can run just one process (with PID = 1), and they MUST run one process, which means that starting the dataminer in background is not possible inside them; this is the reason why there is no separation between the launcher shell and the dataminer shell when it comes to Docker containers. The idea is that the functionalities like stop/restart are offered by the container itself, therefore the dataminer script and its properties file is all that is needed. Just firing the Java process in foreground would not be enough though, because signals like SIGTERM would be intercepted by the hanged calling script rather than the actual Java process, therefore the latter must replace the calling script and become the new (and only) running process inside the container (PID=1), which is obtained by prepending the `exec` command to the Java launch instruction.

* `docker_dataminer.sh`: fires the Java process. This is the script targeted by the CMD directive of the dataminer's Dockerfile base image.
* `docker_run.properties`: the properties to use when launching the Dataminer via the `docker_dataminer.sh` script
