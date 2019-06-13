Jboss CLI snippets compilation
==============================

## Description

A compilation of interesting code snippets useful to manage Jboss via CLI. It is intended as a point of reference to know how to do certain tasks, but remember you will have to adapt them to your environment! (datasource names, usernames...)

In the snippets, I will use certain conventions:

User -> jota

Password -> mypassword

Datasource -> jotaDS

In some sections of the content I will place a subsection for standalone and another for domain. If not said otherwise, a command should work both in standalone and domain mode.

## Table of contents

1. [Connecting and interacting with the CLI](#connecting-and-interacting-with-the-cli)
2. [Platform and subsystems information and monitoring](#platform-and-subsystems-information-and-monitoring)
3. [Snapshots](#snapshots)
4. [Start and stop components](#start-and-stop-components)
5. [Deployments](#deployments)
6. [Datasources](#datasources)
7. [System properties](#system-properties)
8. [Domain specific tips and tricks](#domain-specific-tips-and-tricks)
9. [Network](#network)
10. [Naming and JNDI](#naming-and-jndi)
11. [Logging](#logging)
12. [License](#license)


## Connecting and interacting with the CLI

* There are many ways to connect and interact with the CLI:

    ```
    $JBOSS_HOME/bin/jboss-cli.sh -c
    $JBOSS_HOME/bin/jboss-cli.sh -c --controller=192.168.2.101:9990 --user=jota --password=mypassword
    $JBOSS_HOME/bin/jboss-cli.sh -c --controller=192.168.2.101:9990 --user=jota --password=mypassword --command=":shutdown"
    $JBOSS_HOME/bin/jboss-cli.sh -c --controller=192.168.2.101:9990 --user=jota --password=mypassword --file=mybatch.cli
    ```


## Platform and subsystems information and monitoring

* View codename of the Jboss version

    ```
    :read-attribute(name=release-codename)
    ```
    
* Check product version

    ```
    :read-attribute(name=product-version)
    ```
    
* Check the mode Jboss is running

    ```
    :read-attribute(name=launch-type)
    ```
    
* Check platform attributes, we can see many:

    * We can see many (press TAB)
    
    ```
    [standalone@localhost:9990 /] /core-service=platform-mbean/type=
    buffer-pool  class-loading  compilation  garbage-collector  memory  memory-manager  memory-pool  operating-system  runtime  threading
    ```
    
    * For example in standalone, to see the operating system:
    
    ```
    /core-service=platform-mbean/type=operating-system:read-resource(include-runtime=true,include-defaults=true)
    ```
    
    * And in domain to see the operating system:
    
    ```
    /host=master/server=server-one/core-service=platform-mbean/type=operating-system:read-resource(include-runtime=true,include-defaults=true)
    ```
    
* Check runtime attributes

    * For example, JVM version in standalone:
    
    ```
    /core-service=platform-mbean/type=runtime:read-attribute(name=spec-version)
    ```
    
    * And in domain:
    
    ```
    /host=master/server=server-one/core-service=platform-mbean/type=runtime:read-attribute(name=spec-version,include-defaults=true)
    ```

* List subsystems

    * In standalone:
    
    ```
    /:read-children-names(child-type=subsystem)
    ```

    * In domain, list subsystems of a particular server instance:
    
    ```
    /host=master/server=server-one:read-children-names(child-type=subsystem)
    ```


## Snapshots

* Take snapshot

    ```
    :take-snapshot
    ```
    
* Load snapshot

    ```
    $JBOSS_HOME/bin/standalone.sh --server-config=standalone_xml_history/snapshot/20110913-164449522standalone.xml
    ```
    
* Delete snapshot
    
    ```
    :delete-snapshot(name="20110630-165714239standalone.xml")
    ```
    
* Delete all snapshots
    
    ```
    :delete-snapshot(name="all")
    ```
    
* List snapshots
    
    ```
    :list-snapshots
    ```


## Start and stop components

### Standalone

* Start standalone
    
    ```
    JBOSS_HOME/bin/standalone.sh (additional options, see --help)
    ```
    
* Stop standalone

    * In the CLI:
    
    ```
    :shutdown()
    ```
    
    * Passing as a command:
   
    ```
    $JBOSS_HOME/bin/jboss-cli.sh -c --controller=127.0.0.1:9999 --command=":shutdown()"
    $JBOSS_HOME/bin/jboss-cli.sh -c --controller=127.0.0.1:9999 --command=":shutdown()" --user=jota --password=mypassword
    ```

* Restart
   
    ```
    :shutdown(restart=true)
    ```
   
    * Also passing as a command:
   
    ```
    $JBOSS_HOME/bin/jboss-cli.sh -c --controller=127.0.0.1:9990 --command=":shutdown(restart=true)"
    ```

### Domain

* Start domain
   
    ```
    JBOSS_HOME/bin/domain.sh (additional options)
   
    ```
* Shutdown domain
   
    ```
    $JBOSS_HOME/bin/jboss-cli.sh --connect --controller=localhost:9999 command="/host=master:shutdown"
    ```
    
    * Or if user and password needed:
    
    ```
    $JBOSS_HOME/bin/jboss-cli.sh --connect --controller=localhost:9999 command=/host=master:shutdown  --user=jota --password=mypassword
    ```
    
    * Also once logged in the CLI:
    
    ```
    /host=master:shutdown()
    
    ```
* Start and stop server instances from a server group
    
    ```
    /server-group=main-server-group:start-servers
    /server-group=main-server-group:stop-servers
    ```
    
* Start and stop a particular Jboss instance
    
    ```
    /host=master/server-config=server-one:start
    /host=master/server-config=server-one:stop
    ```
    
* Restart servers from server group
    
    ```
    /server-group=main-server-group:restart-servers()
    ```


## Deployments

### Standalone

* Deploy an application
    
    ```
    deploy /opt/jboss/example_apps/appbinaries/example.war
    ```
    
* Redeploy (force deployment)
    
    ```
    deploy /opt/jboss/example_apps/appbinaries/example.war --force
    ```
    
* Check deployments
  
    ```
    deployment-info
    undeploy -l
    ls deployment
    ```
    
* Undeploy application
    
    ```
    undeploy example.war
    ```

* Disable application
    
    ```
    undeploy example.war --keep-content
    ```

* Enable a disabled application
    
    ```
    deploy --name=example.war
    ```

### Domain

* Deploy in a specific server group
    
    ```
    deploy /opt/jboss/example_apps/appbinaries/example.war --server-groups=main-server-group
    ```
    
* Redeploy/force deployment. Note that no server-group is passed. This is because the content of the application will be redeployed in the server-groups where it is already present
    
    ```
    deploy /opt/jboss/example_apps/appbinaries/example.war --force
    ```
    
* Undeploy in a specific server group
    
    ```
    undeploy example.war --server-groups=main-server-group
    ```
    
* Check deployments in a specific server group
    
    ```
    deployment-info --server-group=main-server-group
    ```
    
* Check information about one deployment
    
    ```
    deployment-info --name=jboss-as-kitchensink.war
    ```
    
* Deploy, undeploy or check all server groups
    
    ```
    deploy /opt/jboss/example_apps/appbinaries/example.war --all-server-groups
    undeploy example.war --all-relevant-server-groups
    ```


## Datasources

### Standalone

* Datasource/pool stastics
    
    ```
    /subsystem=datasources/data-source=jotaDS/statistics=pool:read-resource(include-runtime=true)
    /subsystem=datasources/data-source=jotaDS/statistics=jdbc:read-resource(include-runtime=true)
    /subsystem=datasources/data-source=jotaDS:read-resource(include-runtime=true,recursive=true)
    ```
    
* List description of available attributes and childs of a datasource
    
    ```
    /subsystem=datasources/data-source=jotaDS:read-resource-description
    ```
    
* Add datasource
    
    ```
    data-source add \
        --name=jotaDS \
        --driver-name=oracle \
        --connection-url=jdbc:oracle:thin:@localhost:1521:XE \
        --jndi-name=java:jboss/jdbc/jotaDS \
        --user-name=databaseuser \
        --password=password \
        --use-ccm=false \
        --max-pool-size=25 \
        --blocking-timeout-wait-millis=5000 \
        --enabled=true
    ```
    
* Add XA datasource
    
    ```
    xa-data-source add \
        --name=jotaDS \
        --driver-name=postgresql \
        --jndi-name=java:jboss/jdbc/jotaDS \
        --user-name=demouser \
        --password=password \
        --recovery-username= jota \
        --recovery-password = mypassword \
        --use-ccm=false \
        --max-pool-size=25 \
        --blocking-timeout-wait-millis=5000 \
        --enabled=true"
    /subsystem=datasources/xa-data-source=jotaDS/xa-datasource-properties=ServerName:add(value=localhost)
    /subsystem=datasources/xa-data-source=jotaDS/xa-datasource-properties=PortNumber:add(value=5432)
    /subsystem=datasources/xa-data-source=jotaDS/xa-datasource-properties=DatabaseName:add(value=jotaDatabase)
    ```
    
* Remove, enable, disable datasource
    
    ```
    data-source remove --name=jotaDS
    data-source enable --name=jotaDS
    data-source disable --name=jotaDS
    ```
    
* Remove database driver
    
    ```
    /subsystem=datasources/jdbc-driver=h2:remove
    ```
    
* Add JDBC driver
    
    ```
    /subsystem=datasources/jdbc-driver=postgresql:add(driver-name=postgresql,driver-module-name=org.postgresql.jdbc,driver-xa-datasource-class-name=org.postgresql.xa.PGXADataSource)
    ```
    
* Flush datasources
    
    ```
    /subsystem=datasources/data-source=jotaDS:flush-idle-connection-in-pool
    /subsystem=datasources/data-source=jotaDS:flush-all-connection-in-pool
    ```
    
* Enable statistics (Jboss EAP 6.3+)
    
    ```
    /subsystem=datasources/data-source=jotaDS/statistics=pool:write-attribute(name=statistics-enabled,value=true)
    /subsystem=datasources/data-source=jotaDS/statistics=jdbc:write-attribute(name=statistics-enabled,value=true)
    /subsystem=datasources/data-source=jotaDS:write-attribute(name=statistics-enabled,value=true)
    ```
    
* See statistics
    
    ```
    /subsystem=datasources/data-source=jotaDS/statistics=pool:read-resource(include-runtime=true)
    /subsystem=datasources/data-source=jotaDS/statistics=jdbc:read-resource(include-runtime=true)
    /subsystem=datasources/data-source=jotaDS:read-resource(include-runtime=true,recursive=true)
    ```
    
* List all available datasources
    
    ```
    /subsystem=datasources:read-resource
    ```
    
* Write, remove attributes
    
    ```
    /subsystem=datasources/data-source=jotaDS:write-attribute(name=query-timeout,value=300)
    /subsystem=datasources/data-source=jotaDS:undefine-attribute(name=query-timeout)
    ```

### Domain

* Flush datasources
    
    ```
    /host=master/server=server1/subsystem=datasources/data-source=jotaDS:flush-idle-connection-in-pool
    /host=master/server=server1/subsystem=datasources/data-source=jotaDS:flush-all-connection-in-pool
    ```
    
* Enable statistics in a profile datasource (Jboss EAP 6.3+)
    
    ```
    /profile=full-ha/subsystem=datasources/data-source=jotaDS:write-attribute(name=statistics-enabled,value=true)
    ```
    
* See statistics
    
    ```
    /host=master/server=server1/subsystem=datasources/data-source=jotaDS/statistics=pool:read-resource(include-runtime=true)
    /host=master/server=server1/subsystem=datasources/data-source=jotaDS/statistics=jdbc:read-resource(include-runtime=true)
    ```
    
* Information regarding timeouts, max/min pool, enabled/disabled...
    
    ```
    /host=master/server=server1/subsystem=datasources/data-source=jotaDS:read-resource(include-runtime=true,recursive=true)
    ```
    
    * Also at profile level, information:
    
    ```
    /profile=full-ha/subsystem=datasources/data-source=jotaDS:read-resource(include-runtime=true,recursive=true)
    ```
    
* List all available datasources in a given profile
    
    ```
    /profile=full-ha/subsystem=datasources:read-resource
    ```
    
* Write, remove attributes
    
    ```
    /profile=myprofile-ha/subsystem=datasources/data-source=jotaDS:write-attribute(name=query-timeout,value=300)
    /profile=myprofile-ha/subsystem=datasources/data-source=jotaDS:undefine-attribute(name=query-timeout)
    ```
    
* Add datasource to a profile (in this example,full-ha)
    
    ```
    data-source add \
        --name=jotaDS \
        --driver-name=oracle \
        --connection-url=jdbc:oracle:thin:@localhost:1521:XE \
        --jndi-name=java:jboss/jdbc/jotaDS \
        --user-name=databaseuser \
        --password=password \
        --use-ccm=false \
        --max-pool-size=25 \
        --blocking-timeout-wait-millis=5000 \
        --profile=full-ha \
        --enabled=true
    ```
    
* Add XA datasource to a profile (in this example,full-ha)
    
    ```
    xa-data-source add \
        --name=jotaDS \
        --driver-name=postgresql \
        --jndi-name=java:jboss/jdbc/jotaDS \
        --user-name=demouser \
        --password=password \
        --recovery-username= jota \
        --recovery-password = mypassword \
        --use-ccm=false \
        --max-pool-size=25 \
        --blocking-timeout-wait-millis=5000 \
        --enabled=true" \
        --profile=full-ha
    ```
    
    * After adding it, add the connection URL:
    
    ```
    /profile=full-ha/subsystem=datasources/xa-data-source=ApplicationXADS/xa-datasource-properties=ServerName:add(value=localhost)
    /profile=full-ha/subsystem=datasources/xa-data-source=ApplicationXADS/xa-datasource-properties=PortNumber:add(value=5432)
    /profile=full-ha/subsystem=datasources/xa-data-source=ApplicationXADS/xa-datasource-properties=DatabaseName:add(value=jotaDB)
    ```
    
* Remove, disable, enable datasource
    
    ```
    data-source --profile=full-ha remove --name=jotaDS
    data-source --profile=full-ha disable --name=jotaDS
    data-source --profile=full-ha enable --name=jotaDS
    ```
    
* Read datasource
    
    ```
    data-source --profile=full-ha read-resource --name=jotaDS
    ```
    
    * Also at profile level:
    
    ```
    /profile=full-ha/subsystem=datasources/data-source=jotaDS:read-resource()
    ```


## System properties

### Standalone

* Add, read, remove, write-attribute
    
    ```
    /system-property=foo:add(value=bar)
    /system-property=foo:read-resource
    /system-property=foo:remove
    /system-property=foo:write-attribute(name="value", value="newValue")
    ```
    
* View all system properties
    
    ```
    /core-service=platform-mbean/type=runtime:read-attribute(name=system-properties)
    ```

### Domain

* Add, read, remove, write-attribute affecting all hosts and server instances in domain (the system property will be added in domain.xml)
    
    ```
    /system-property=foo:add(value=bar)
    /system-property=foo:read-resource
    /system-property=foo:remove
    /system-property=foo:write-attribute(name="value", value="newValue")
    ```
    
* Add, read, remove write-attribute affecting only host and its server instances (the system property will be added in host.xml)
    
    ```
    /host=master/system-property=foo:add(value=bar)
    /host=master/system-property=foo:read-resource
    /host=master/system-property=foo:remove
    /host=master/system-property=foo:write-attribute(name="value", value="newValue")
    ```
    
* Manage system properties at server instance level
    
    ```
    /host=master/server-config=server-one/system-property=foo:add(value=bar)
    /host=master/server-config=server-one/system-property=foo:read-resource
    /host=master/server-config=server-one/system-property=foo:remove
    /host=master/server-config=server-one/system-property=foo:write-attribute(name="value", value="newValue")
    ```
    
* View system properties
    * At host level:
    
    ```
    /host=master/core-service=platform-mbean/type=runtime:read-attribute(name=system-properties)
    ```
    
    * At server instance level:
    
    ```
    /host=master/server=server-one/core-service=platform-mbean/type=runtime:read-attribute(name=system-properties)
    ```


## Domain specific tips and tricks

* View to which server group belongs a server:
    
    ```
    /host=master/server-config=server-one:read-attribute(name=group)
    ```
    
* To view all the servers and the groups they belong to:
    
    ```
    /host=master/server-config=*:read-attribute(name=group)
    ```
    
* Get list of servers available in the domain in a particular host:
    
    ```
    ls host=master/server-config
    ```
    
    * Or also:
    
    ```
    /host=master:read-children-names(child-type=server-config)
    ```
    
* To view info about a particular server
    
    ```
    /host=master/server-config=server-one:read-resource(include-runtime=true)
    ```
    
* Configure JVM for a particular Jboss instance
    
    ```
    /host=master/server-config=server-one/jvm=MYJVM/:add(max-heap-size=1028m,env-classpath-ignored=false,permgen-size=256m,max-permgen-size=256m,heap-size=1028m,jvm-options=["-server"])
    ```
    
* Add JVM option to the default definition of your JVM

    * Host level
    
    ```
    /host=master/jvm=default:add-jvm-option(jvm-option="-XX:NewSize=1024m")
    ```
    
    * Server-group level
    
    ```
    /server-group=main-server-group/jvm=default:add-jvm-option(jvm-option="-XX:NewSize=1024m")
    ```
    
    * Jboss instance levelç
    
    ```
    /host=master/server-config=server-one/jvm=MYJVM:add-jvm-option(jvm-option="-XX:NewSize=1024m")
    ```


## Network

### Standalone

* See public interface
    
    ```
    /interface=public:read-resource(include-runtime=true)
    ```

### Domain

* See public interface of a Jboss server instance
    
    ```
    /host=master/server=server-one/interface=public:read-resource(include-runtime=true)
    ```


## Naming and JNDI

### Standalone

* Add, remove, modify, read JNDI
    
    ```
    /subsystem=naming/binding=java\:jboss\/param\/demoParam:add(value="Demo configuration value",binding-type=simple)
    ```
    
    * Will result in:
    
    ```
    <subsystem xmlns="urn:jboss:domain:naming:1.4">
        <bindings>
            <simple name="java:jboss/param/demoParam" value="Demo configuration value"/>                            
        </bindings>
        <remote-naming/>
    </subsystem>
    ```
    
    * Remove, modify and read:
    
    ```
    /subsystem=naming/binding=java\:jboss\/param\/demoParam:remove()
    /subsystem=naming/binding=java\:jboss\/param\/demoParam:write-attribute(name=value,value="Modified")
    /subsystem=naming/binding=java\:jboss\/param\/demoParam:read-resource(include-defaults=true)
    ```
    
* Check JNDI Tree
    
    ```
    /subsystem=naming:jndi-view
    ```

### Domain

* Add,remove, modify, read JNDI
    
    ```
    /profile=full-ha/subsystem=naming/binding=java\:jboss\/param\/demoParam:add(value="Demo configuration value",binding-type=simple)
    /profile=full-ha/subsystem=naming/binding=java\:jboss\/param\/demoParam:remove()
    /profile=full-ha/subsystem=naming/binding=java\:jboss\/param\/demoParam:write-attribute(name=value,value="Modified")
    /profile=full-ha/subsystem=naming/binding=java\:jboss\/param\/demoParam:read-resource(include-defaults=true)
    ```
    
* Check JNDI Tree
    
    ```
    /host=master/server=server-one/subsystem=naming:jndi-view()
    ```


## Logging

### Standalone

* Change a logger log level, for example root-logger to DEBUG:

    ```
    /subsystem=logging/root-logger=ROOT/:write-attribute(name=level,value=DEBUG)
    ```

* Create log-category for a specific package:

    ```
    /subsystem=logging/logger=package.name/:add(category=package.name,level=INFO,use-parent-handlers=true)
    ```

* Change log level for that package to DEBUG:

    ```
    /subsystem=logging/logger=package.name:change-log-level(level=DEBUG)
    ```

* Take a look at the following interesting operations you can perform since Jboss EAP 7
    
    * List logs available for a Jboss instance 

    ```
    /subsystem=logging:list-log-files
    ```

    * Read last 5 lines of server.log 

    ```
    /subsystem=logging:read-log-file(name=server.log,lines=5,skip=0)
    ```

    * Make a tail showing the last 10 lines of server.log 
    ```
    /subsystem=logging:read-log-file(name=server.log,tail=true,lines=10,skip=0)
    ```

### Domain

* Change the log level of the root-logger to DEBUG in the profile my-app (this profile name will change in your case):
   
    ```
    /profile=my-app/subsystem=logging/root-logger=ROOT:change-root-log-level(level=DEBUG)
    ```

* Add log-category for a package:

    ```
    /profile=my-app/subsystem=logging/logger=package.name/:add(category=package.name,level=INFO,use-parent-handlers=true)
    ```

* Change log level of a log-category:

    ```
    /profile=my-app/subsystem=logging/logger=package.name:change-log-level(level=DEBUG)
    ```

* Also in domain mode, you can perform the following operations since Jboss EAP 7

    * List logs available for a Jboss instance server-one in master host

    ```
    /host=master/server=server-one/subsystem=logging:list-log-files
    ```

    * Read last 5 lines of server.log of server-one

    ```
    /host=master/server=server-one/subsystem=logging:read-log-file(name=server.log,lines=5,skip=0)
    ```

    * Make a tail showing the last 10 lines of server-one's server.log 

    ```
    /host=master/server=server-one/subsystem=logging:read-log-file(name=server.log,tail=true,lines=10,skip=0)
    ```
    
## License

This repo is licensed under [Creative Commons Attribution 4.0 International](https://creativecommons.org/licenses/by/4.0/)
