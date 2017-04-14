# [![Application icon](https://lh4.googleusercontent.com/LZNc-LXQZkLS_wo2bSfORPnHmaurkF7ahI88irILnnV0qIU4ThHKS1rYltcTFTSW_liF7C9CXy8n9nk=w1920-h950)][icon]
[icon]: https://lh4.googleusercontent.com/LZNc-LXQZkLS_wo2bSfORPnHmaurkF7ahI88irILnnV0qIU4ThHKS1rYltcTFTSW_liF7C9CXy8n9nk=w1920-h950

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
Your model class should extend the basic model implementation provided by the core, called **BaseModel**.
**BaseModel** has an abstract **parse()** method that must be implemented; moreover, two constructors are needed, a noargs ctor used by Hibernate, and one that takes a String as the single input parameter.
This String is the data read by the core via a connector, and passed to this model.
Let's create a first, basic model class:

```
package yourpackage.model.implementations.test;

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
