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
** reading the config.properties file and use it as its base configuration
** read the config.xml file to decide which connectors to start
** receive the data extracted from each connector and decide if and what to do with it (basically, if there is any model for it)
** offers the following features to model instances: be persisted on a target database, be sent to a target jms topic, be sent via email, be sent to a jtec server
** offers a jmx and rest interface for management
** ...
* N connectors: each connector knows how to extract data from a target source and technology, but doesn't know what to look for, therefore it can be reused
* M models: a model for each business even of interest. A model receives from the core an entry that it's in charge of managing, and must implement at the very least the parse() method, that takes such input and parses it assigning the insteresting parts to instance variables. It can then signal the core that it wants to be persisted, sent via email, etc. 

Usually, the workflow is to start by creating the new model classes, then create the configuration files and go ahead with the deployment.
