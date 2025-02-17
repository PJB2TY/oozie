

[::Go back to Oozie Documentation Index::](index.html)

-----

# Oozie Shell Action Extension

<!-- MACRO{toc|fromDepth=1|toDepth=4} -->

<a name="ShellAction"></a>
## Shell Action

The `shell` action runs a Shell command.

The workflow job will wait until the Shell command completes before
continuing to the next action.

To run the Shell job, you have to configure the `shell` action with the
`job-tracker`, `name-node` and Shell `exec` elements as
well as the necessary arguments and configuration.

A `shell` action can be configured to create or delete HDFS directories
before starting the Shell job.

Shell _launcher_ configuration can be specified with a file, using the `job-xml`
element, and inline, using the `configuration` elements.

Oozie EL expressions can be used in the inline configuration. Property
values specified in the `configuration` element override values specified
in the `job-xml` file.

Note that YARN `yarn.resourcemanager.address` (`resource-manager`) and HDFS `fs.default.name` (`name-node`) properties
must not be present in the inline configuration.

As with Hadoop `map-reduce` jobs, it is possible to add files and
archives in order to make them available to the Shell job. Refer to the
[WorkflowFunctionalSpec#FilesArchives][Adding Files and Archives for the Job]
section for more information about this feature.

The output (STDOUT) of the Shell job can be made available to the workflow job after the Shell job ends. This information
could be used from within decision nodes. If the output of the Shell job is made available to the workflow job the shell
command must follow the following requirements:

   * The format of the output must be a valid Java Properties file.
   * The size of the output must not exceed 2KB.

**Syntax:**


```
<workflow-app name="[WF-DEF-NAME]" xmlns="uri:oozie:workflow:1.0">
    ...
    <action name="[NODE-NAME]">
        <shell xmlns="uri:oozie:shell-action:1.0">
            <resource-manager>[RESOURCE-MANAGER]</resource-manager>
            <name-node>[NAME-NODE]</name-node>
            <prepare>
               <delete path="[PATH]"/>
               ...
               <mkdir path="[PATH]"/>
               ...
            </prepare>
            <job-xml>[SHELL SETTINGS FILE]</job-xml>
            <configuration>
                <property>
                    <name>[PROPERTY-NAME]</name>
                    <value>[PROPERTY-VALUE]</value>
                </property>
                ...
            </configuration>
            <exec>[SHELL-COMMAND]</exec>
            <argument>[ARG-VALUE]</argument>
                ...
            <argument>[ARG-VALUE]</argument>
            <env-var>[VAR1=VALUE1]</env-var>
               ...
            <env-var>[VARN=VALUEN]</env-var>
            <file>[FILE-PATH]</file>
            ...
            <archive>[FILE-PATH]</archive>
            ...
            <capture-output/>
        </shell>
        <ok to="[NODE-NAME]"/>
        <error to="[NODE-NAME]"/>
    </action>
    ...
</workflow-app>
```

The `prepare` element, if present, indicates a list of paths to delete
or create before starting the job. Specified paths must start with `hdfs://HOST:PORT`.

The `job-xml` element, if present, specifies a file containing configuration
for the Shell job. As of schema 0.2, multiple `job-xml` elements are allowed in order to
specify multiple `job.xml` files.

The `configuration` element, if present, contains configuration
properties that are passed to the Shell job.

The `exec` element must contain the path of the Shell command to
execute. The arguments of Shell command can then be specified
using one or more `argument` element.

The `argument` element, if present, contains argument to be passed to
the Shell command.

The `env-var` element, if present, contains the environment to be passed
to the Shell command. `env-var` should contain only one pair of environment variable
and value. If the pair contains the variable such as $PATH, it should follow the
Unix convention such as PATH=$PATH:mypath. Don't use ${PATH} which will be
substituted by Oozie's EL evaluator.

A `shell` action creates a Hadoop configuration. The Hadoop configuration is made available as a local file to the
Shell application in its running directory. The exact file path is exposed to the spawned shell using the environment
variable called `OOZIE_ACTION_CONF_XML`.The Shell application can access the environment variable to read the action
configuration XML file path.

If the `capture-output` element is present, it indicates Oozie to capture output of the STDOUT of the shell command
execution. The Shell command output must be in Java Properties file format and it must not exceed 2KB. From within the
workflow definition, the output of an Shell action node is accessible via the `String action:output(String node,
String key)` function (Refer to section '4.2.6 Action EL Functions').

All the above elements can be parameterized (templatized) using EL
expressions.

**Example:**

How to run any shell script or perl script or CPP executable


```
<workflow-app xmlns='uri:oozie:workflow:1.0' name='shell-wf'>
    <start to='shell1' />
    <action name='shell1'>
        <shell xmlns="uri:oozie:shell-action:1.0">
            <resource-manager>${resourceManager}</resource-manager>
            <name-node>${nameNode}</name-node>
            <configuration>
                <property>
                  <name>mapred.job.queue.name</name>
                  <value>${queueName}</value>
                </property>
            </configuration>
            <exec>${EXEC}</exec>
            <argument>A</argument>
            <argument>B</argument>
            <file>${EXEC}#${EXEC}</file> <!--Copy the executable to compute node's current working directory -->
        </shell>
        <ok to="end" />
        <error to="fail" />
    </action>
    <kill name="fail">
        <message>Script failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
    </kill>
    <end name='end' />
</workflow-app>
```

The corresponding job properties file used to submit Oozie job could be as follows:


```
oozie.wf.application.path=hdfs://localhost:8020/user/kamrul/workflows/script

#Execute is expected to be in the Workflow directory.
#Shell Script to run
EXEC=script.sh
#CPP executable. Executable should be binary compatible to the compute node OS.
#EXEC=hello
#Perl script
#EXEC=script.pl

resourceManager=localhost:8032
nameNode=hdfs://localhost:8020
queueName=default

```

How to run any java program bundles in a jar.


```
<workflow-app xmlns='uri:oozie:workflow:1.0' name='shell-wf'>
    <start to='shell1' />
    <action name='shell1'>
        <shell xmlns="uri:oozie:shell-action:1.0">
            <resource-manager>${resourceManager}</resource-manager>
            <name-node>${nameNode}</name-node>
            <configuration>
                <property>
                  <name>mapred.job.queue.name</name>
                  <value>${queueName}</value>
                </property>
            </configuration>
            <exec>java</exec>
            <argument>-classpath</argument>
            <argument>./${EXEC}:$CLASSPATH</argument>
            <argument>Hello</argument>
            <file>${EXEC}#${EXEC}</file> <!--Copy the jar to compute node current working directory -->
        </shell>
        <ok to="end" />
        <error to="fail" />
    </action>
    <kill name="fail">
        <message>Script failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
    </kill>
    <end name='end' />
</workflow-app>
```

The corresponding job properties file used to submit Oozie job could be as follows:


```
oozie.wf.application.path=hdfs://localhost:8020/user/kamrul/workflows/script

#Hello.jar file is expected to be in the Workflow directory.
EXEC=Hello.jar

resourceManager=localhost:8032
nameNode=hdfs://localhost:8020
queueName=default
```

### Shell Action Configuration

 * `oozie.action.shell.setup.hadoop.conf.dir` - Generates a config directory with various core/hdfs/yarn/mapred-site.xml files and points `HADOOP_CONF_DIR` and `YARN_CONF_DIR` env-vars to it, before the Script is invoked. XML is sourced from the action configuration. Useful when the Shell script passed uses various `hadoop` commands. Default is false.
 * `oozie.action.shell.setup.hadoop.conf.dir.write.log4j.properties` - When `oozie.action.shell.setup.hadoop.conf.dir` is enabled, toggle if a log4j.properties file should also be written under the configuration files directory. Default is true.
 * `oozie.action.shell.setup.hadoop.conf.dir.log4j.content` - When `oozie.action.shell.setup.hadoop.conf.dir.write.log4j.properties` is enabled, the content to write into the log4j.properties file under the configuration files directory. Default is a simple console based stderr logger, as presented below:

```
log4j.rootLogger=${hadoop.root.logger}
hadoop.root.logger=INFO,console
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.target=System.err
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=%d{yy/MM/dd HH:mm:ss} %p %c{2}: %m%n
```

### Shell Action Logging

Shell action's stdout and stderr output are redirected to the Oozie Launcher map-reduce job task STDOUT that runs the shell command.

### Shell Action Limitations
Although Shell action can execute any shell command, there are some limitations.

   * No interactive command is supported.
   * Command can't be executed as different user using sudo.
   * User has to explicitly upload the required 3rd party packages (such as jar, so lib, executable etc). Oozie provides a way using \<file\> and \<archive\> tag through Hadoop's Distributed Cache to upload.
   * Since Oozie will execute the shell command into a Hadoop compute node, the default installation of utility in the compute node might not be fixed. However, the most common unix utilities are usually installed on all compute nodes. It is important to note that Oozie could only support the commands that are installed into the compute nodes or that are uploaded through Distributed Cache.

## Appendix, Shell XML-Schema

### AE.A Appendix A, Shell XML-Schema

#### Shell Action Schema Version 1.0

```
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
           xmlns:shell="uri:oozie:shell-action:1.0"
           elementFormDefault="qualified"
           targetNamespace="uri:oozie:shell-action:1.0">
.
    <xs:include schemaLocation="oozie-common-1.0.xsd"/>
.
    <xs:element name="shell" type="shell:ACTION"/>
.
    <xs:complexType name="ACTION">
      <xs:sequence>
            <xs:choice>
                <xs:element name="job-tracker" type="xs:string" minOccurs="0" maxOccurs="1"/>
                <xs:element name="resource-manager" type="xs:string" minOccurs="0" maxOccurs="1"/>
            </xs:choice>
            <xs:element name="name-node" type="xs:string" minOccurs="0" maxOccurs="1"/>
            <xs:element name="prepare" type="shell:PREPARE" minOccurs="0" maxOccurs="1"/>
            <xs:element name="launcher" type="shell:LAUNCHER" minOccurs="0" maxOccurs="1"/>
            <xs:element name="job-xml" type="xs:string" minOccurs="0" maxOccurs="unbounded"/>
            <xs:element name="configuration" type="shell:CONFIGURATION" minOccurs="0" maxOccurs="1"/>
            <xs:element name="exec" type="xs:string" minOccurs="1" maxOccurs="1"/>
            <xs:element name="argument" type="xs:string" minOccurs="0" maxOccurs="unbounded"/>
            <xs:element name="env-var" type="xs:string" minOccurs="0" maxOccurs="unbounded"/>
            <xs:element name="file" type="xs:string" minOccurs="0" maxOccurs="unbounded"/>
            <xs:element name="archive" type="xs:string" minOccurs="0" maxOccurs="unbounded"/>
            <xs:element name="capture-output" type="shell:FLAG" minOccurs="0" maxOccurs="1"/>
        </xs:sequence>
    </xs:complexType>
.
</xs:schema>
```

#### Shell Action Schema Version 0.3

```
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
           xmlns:shell="uri:oozie:shell-action:0.3" elementFormDefault="qualified"
           targetNamespace="uri:oozie:shell-action:0.3">

    <xs:element name="shell" type="shell:ACTION"/>

    <xs:complexType name="ACTION">
      <xs:sequence>
            <xs:element name="job-tracker" type="xs:string" minOccurs="0" maxOccurs="1"/>
            <xs:element name="name-node" type="xs:string" minOccurs="0" maxOccurs="1"/>
            <xs:element name="prepare" type="shell:PREPARE" minOccurs="0" maxOccurs="1"/>
            <xs:element name="job-xml" type="xs:string" minOccurs="0" maxOccurs="unbounded"/>
            <xs:element name="configuration" type="shell:CONFIGURATION" minOccurs="0" maxOccurs="1"/>
            <xs:element name="exec" type="xs:string" minOccurs="1" maxOccurs="1"/>
            <xs:element name="argument" type="xs:string" minOccurs="0" maxOccurs="unbounded"/>
            <xs:element name="env-var" type="xs:string" minOccurs="0" maxOccurs="unbounded"/>
            <xs:element name="file" type="xs:string" minOccurs="0" maxOccurs="unbounded"/>
            <xs:element name="archive" type="xs:string" minOccurs="0" maxOccurs="unbounded"/>
            <xs:element name="capture-output" type="shell:FLAG" minOccurs="0" maxOccurs="1"/>
        </xs:sequence>
    </xs:complexType>

    <xs:complexType name="FLAG"/>

    <xs:complexType name="CONFIGURATION">
        <xs:sequence>
            <xs:element name="property" minOccurs="1" maxOccurs="unbounded">
                <xs:complexType>
                    <xs:sequence>
                        <xs:element name="name" minOccurs="1" maxOccurs="1" type="xs:string"/>
                        <xs:element name="value" minOccurs="1" maxOccurs="1" type="xs:string"/>
                        <xs:element name="description" minOccurs="0" maxOccurs="1" type="xs:string"/>
                    </xs:sequence>
                </xs:complexType>
            </xs:element>
        </xs:sequence>
    </xs:complexType>

    <xs:complexType name="PREPARE">
        <xs:sequence>
            <xs:element name="delete" type="shell:DELETE" minOccurs="0" maxOccurs="unbounded"/>
            <xs:element name="mkdir" type="shell:MKDIR" minOccurs="0" maxOccurs="unbounded"/>
        </xs:sequence>
    </xs:complexType>

    <xs:complexType name="DELETE">
        <xs:attribute name="path" type="xs:string" use="required"/>
    </xs:complexType>

    <xs:complexType name="MKDIR">
        <xs:attribute name="path" type="xs:string" use="required"/>
    </xs:complexType>

</xs:schema>
```

#### Shell Action Schema Version 0.2

```
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
           xmlns:shell="uri:oozie:shell-action:0.2" elementFormDefault="qualified"
           targetNamespace="uri:oozie:shell-action:0.2">

    <xs:element name="shell" type="shell:ACTION"/>

    <xs:complexType name="ACTION">
      <xs:sequence>
            <xs:element name="job-tracker" type="xs:string" minOccurs="1" maxOccurs="1"/>
            <xs:element name="name-node" type="xs:string" minOccurs="1" maxOccurs="1"/>
            <xs:element name="prepare" type="shell:PREPARE" minOccurs="0" maxOccurs="1"/>
            <xs:element name="job-xml" type="xs:string" minOccurs="0" maxOccurs="unbounded"/>
            <xs:element name="configuration" type="shell:CONFIGURATION" minOccurs="0" maxOccurs="1"/>
            <xs:element name="exec" type="xs:string" minOccurs="1" maxOccurs="1"/>
            <xs:element name="argument" type="xs:string" minOccurs="0" maxOccurs="unbounded"/>
            <xs:element name="env-var" type="xs:string" minOccurs="0" maxOccurs="unbounded"/>
            <xs:element name="file" type="xs:string" minOccurs="0" maxOccurs="unbounded"/>
            <xs:element name="archive" type="xs:string" minOccurs="0" maxOccurs="unbounded"/>
            <xs:element name="capture-output" type="shell:FLAG" minOccurs="0" maxOccurs="1"/>
        </xs:sequence>
    </xs:complexType>

    <xs:complexType name="FLAG"/>

    <xs:complexType name="CONFIGURATION">
        <xs:sequence>
            <xs:element name="property" minOccurs="1" maxOccurs="unbounded">
                <xs:complexType>
                    <xs:sequence>
                        <xs:element name="name" minOccurs="1" maxOccurs="1" type="xs:string"/>
                        <xs:element name="value" minOccurs="1" maxOccurs="1" type="xs:string"/>
                        <xs:element name="description" minOccurs="0" maxOccurs="1" type="xs:string"/>
                    </xs:sequence>
                </xs:complexType>
            </xs:element>
        </xs:sequence>
    </xs:complexType>

    <xs:complexType name="PREPARE">
        <xs:sequence>
            <xs:element name="delete" type="shell:DELETE" minOccurs="0" maxOccurs="unbounded"/>
            <xs:element name="mkdir" type="shell:MKDIR" minOccurs="0" maxOccurs="unbounded"/>
        </xs:sequence>
    </xs:complexType>

    <xs:complexType name="DELETE">
        <xs:attribute name="path" type="xs:string" use="required"/>
    </xs:complexType>

    <xs:complexType name="MKDIR">
        <xs:attribute name="path" type="xs:string" use="required"/>
    </xs:complexType>

</xs:schema>
```

#### Shell Action Schema Version 0.1

```
<?xml version="1.0" encoding="UTF-8"?>
<!--
  Licensed to the Apache Software Foundation (ASF) under one
  or more contributor license agreements.  See the NOTICE file
  distributed with this work for additional information
  regarding copyright ownership.  The ASF licenses this file
  to you under the Apache License, Version 2.0 (the
  "License"); you may not use this file except in compliance
  with the License.  You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->

<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
           xmlns:shell="uri:oozie:shell-action:0.1" elementFormDefault="qualified"
           targetNamespace="uri:oozie:shell-action:0.1">

    <xs:element name="shell" type="shell:ACTION"/>

    <xs:complexType name="ACTION">
      <xs:sequence>
            <xs:element name="job-tracker" type="xs:string" minOccurs="1" maxOccurs="1"/>
            <xs:element name="name-node" type="xs:string" minOccurs="1" maxOccurs="1"/>
            <xs:element name="prepare" type="shell:PREPARE" minOccurs="0" maxOccurs="1"/>
            <xs:element name="job-xml" type="xs:string" minOccurs="0" maxOccurs="1"/>
            <xs:element name="configuration" type="shell:CONFIGURATION" minOccurs="0" maxOccurs="1"/>
             <xs:element name="exec" type="xs:string" minOccurs="1" maxOccurs="1"/>
            <xs:element name="argument" type="xs:string" minOccurs="0" maxOccurs="unbounded"/>
            <xs:element name="env-var" type="xs:string" minOccurs="0" maxOccurs="unbounded"/>
            <xs:element name="file" type="xs:string" minOccurs="0" maxOccurs="unbounded"/>
            <xs:element name="archive" type="xs:string" minOccurs="0" maxOccurs="unbounded"/>
            <xs:element name="capture-output" type="shell:FLAG" minOccurs="0" maxOccurs="1"/>
        </xs:sequence>
    </xs:complexType>

    <xs:complexType name="FLAG"/>

    <xs:complexType name="CONFIGURATION">
        <xs:sequence>
            <xs:element name="property" minOccurs="1" maxOccurs="unbounded">
                <xs:complexType>
                    <xs:sequence>
                        <xs:element name="name" minOccurs="1" maxOccurs="1" type="xs:string"/>
                        <xs:element name="value" minOccurs="1" maxOccurs="1" type="xs:string"/>
                        <xs:element name="description" minOccurs="0" maxOccurs="1" type="xs:string"/>
                    </xs:sequence>
                </xs:complexType>
            </xs:element>
        </xs:sequence>
    </xs:complexType>

    <xs:complexType name="PREPARE">
        <xs:sequence>
            <xs:element name="delete" type="shell:DELETE" minOccurs="0" maxOccurs="unbounded"/>
            <xs:element name="mkdir" type="shell:MKDIR" minOccurs="0" maxOccurs="unbounded"/>
        </xs:sequence>
    </xs:complexType>

    <xs:complexType name="DELETE">
        <xs:attribute name="path" type="xs:string" use="required"/>
    </xs:complexType>

    <xs:complexType name="MKDIR">
        <xs:attribute name="path" type="xs:string" use="required"/>
    </xs:complexType>

</xs:schema>
```

[::Go back to Oozie Documentation Index::](index.html)


