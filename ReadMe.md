Dynamic Routing Example
=======================

Overview
--------

This example shows how to use [Apache Camel](http://camel.apache.org), and its OSGi integration to dynamicly route
messages to newly deployed OSGi bundles. This allows you to extend **at runtime** to routing to new or updated services
without impacting running services.

This example combines use of Camel's [Recipient List](http://camel.apache.org/recipient-list.html), which allows you to
at runtime specify the Camel [Endpoint](http://camel.apache.org/endpoint.html) to route to, and use of the
[Camel VM Component](http://camel.apache.org/vm.html), which prodives a SEDA queue that can be accessed from different
OSGi bundles running in the same Java virtual machine.

Note: Extra steps, like use of Camel VM Component, need to be taken when accessing Camel Routes in different Camel Contextes,
and in different OSGi bundles, as you are dealing with classes in different ClassLoaders...

Alternative Approaches
----------------------

Instead of using [Camel VM Component](http://camel.apache.org/vm.html), use of
[ActiveMQ-Camel Component](http://camel.apache.org/activemq.html) would also work. You'd need to configure a single
ActiveMQ broker instance that all Camel routes can reference. To optimize performance, if the ActiveMQ broker runs
embedded within Fuse ESB, then you can use [ActiveMQ's VM Transport](http://activemq.apache.org/vm-transport-reference.html),
which is conceptually similar to Camel's VM Component in that it effectively uses direct method calls to enhance performance
within a single Java Virtual Machine.

You could also use the [Camel NMR Component](http://camel.apache.org/nmr), which leverages Fuse ESB / Apache ServiceMix's
NMR component. This provides a way to reference named endpoints across OSGi bundles. The issue with this approach is
that the NMR within Fuse ESB / ServiceMix is loosing favor within the Apache ServiceMix community, and may be
deprecated in the not to distant future.

----

Building
--------

To build this project use

    mvn install

To deploy this project into [Fuse ESB](http://fusesource.com/downloads)

Initial Base Service
--------------------

Start Fuse ESB

    <Fuse ESB Home>/bin/fuseesb

In the Fuse ESB console, use the following

    FuseESB:karaf@root> features:addurl mvn:org.fusesource.example.dynamic/base/0.0.1-SNAPSHOT/xml/features
    FuseESB:karaf@root> features:install dynamic-routing-base

Then in a different Command Prompt, change to the `client` sub-project, and run

    <dynamic-routing/client> $ mvn -Psimple

You should see log entries in both Fuse ESB console, and the Command Prompt that messages are flowing to the Simple
Route

    18:37:45,126 | INFO  | qtp317981251-233 | Base                             | 139 - org.apache.camel.camel-core - 2.9.0.fuse-70-084 | Processing: direct:simple
    18:37:45,127 | INFO  | qtp317981251-233 | Simple                           | 139 - org.apache.camel.camel-core - 2.9.0.fuse-70-084 | Processing: direct:simple
    18:37:46,043 | INFO  | qtp317981251-232 | Base                             | 139 - org.apache.camel.camel-core - 2.9.0.fuse-70-084 | Processing: direct:simple
    18:37:46,043 | INFO  | qtp317981251-232 | Simple                           | 139 - org.apache.camel.camel-core - 2.9.0.fuse-70-084 | Processing: direct:simple

You can also try the Other route to see that the recipient list is working correctly within the Base service

    <dynamic-routing/client> $ mvn -Pother

    18:39:59,483 | INFO  | qtp317981251-232 | Base                             | 139 - org.apache.camel.camel-core - 2.9.0.fuse-70-084 | Processing: direct:other
    18:39:59,484 | INFO  | qtp317981251-232 | Other                            | 139 - org.apache.camel.camel-core - 2.9.0.fuse-70-084 | Processing: direct:other
    18:40:00,399 | INFO  | qtp317981251-235 | Base                             | 139 - org.apache.camel.camel-core - 2.9.0.fuse-70-084 | Processing: direct:other
    18:40:00,400 | INFO  | qtp317981251-235 | Other                            | 139 - org.apache.camel.camel-core - 2.9.0.fuse-70-084 | Processing: direct:other

Installing and Testing `newservice`
-----------------------------------

In the Fuse ESB console, use the following

    FuseESB:karaf@root> features:addurl mvn:org.fusesource.example.dynamic/newservice/0.0.1-SNAPSHOT/xml/features
    FuseESB:karaf@root> features:install dynamic-routing-newservice

Then in a different Command Prompt, change to the `client` sub-project, and run

    <dynamic-routing/client> $ mvn -Pnewservice

You should see log entries in both Fuse ESB console, and the Command Prompt that messages are flowing to the New Service
Route

    18:41:51,479 | INFO  | qtp317981251-233 | Base                             | 139 - org.apache.camel.camel-core - 2.9.0.fuse-70-084 | Processing: vm:newservice
    18:41:51,480 | INFO  |  vm://newservice | newservice-route                 | 139 - org.apache.camel.camel-core - 2.9.0.fuse-70-084 | Message: vm:newservice
    18:41:51,489 | INFO  | umer[newservice] | newservice-consumer              | 139 - org.apache.camel.camel-core - 2.9.0.fuse-70-084 | Consumer: vm:newservice
    18:41:52,394 | INFO  | qtp317981251-232 | Base                             | 139 - org.apache.camel.camel-core - 2.9.0.fuse-70-084 | Processing: vm:newservice
    18:41:52,396 | INFO  |  vm://newservice | newservice-route                 | 139 - org.apache.camel.camel-core - 2.9.0.fuse-70-084 | Message: vm:newservice
    18:41:52,399 | INFO  | umer[newservice] | newservice-consumer              | 139 - org.apache.camel.camel-core - 2.9.0.fuse-70-084 | Consumer: vm:newservice

Notice that the original Base service did not reload or restart to start forwarding messages to the newly loaded routes.

`New Service` grabs the messages from the Camel VM component, and puts them onto (and off of) an ActiveMQ Queue. This
shows how to route to new endpoints and integration routes at runtime -- it does not have to be ActiveMQ, it could
easily be WS, REST, other JMS (Tibco, WMQ), etc.

----

Using Fuse Management Console and Fuse Fabric to Manage
-------------------------------------------------------


