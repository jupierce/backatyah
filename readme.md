
# OpenShift Jenkins Pipeline (DSL) Plugin - Experimental

## Overview
The [OpenShift](https://www.openshift.com) Pipeline DSL Plugin is presently an experimental Jenkins plugin 
which aims to provide a readable, concise, comprehensive, and fluent 
[Jenkins Pipeline](https://jenkins.io/doc/book/pipeline/) syntax for rich interactions with an OpenShift API Server.

Familiarity with OpenShift [command line interface](https://docs.openshift.org/latest/cli_reference/basic_cli_operations.html)
is highly encouraged before exploring the plugin's features. The DSL leverages the [oc](https://docs.openshift.org/latest/cli_reference/index.html) 
binary and, in many cases, passes method arguments directly on to the command line. This document cannot, therefore,
provide a complete description of all possible OpenShift interactions -- the user may need to reference
the CLI documentation to find the pass-through arguments a given interaction requires.



```groovy
def t="this is a string"
```

If you are interested in a non-experimental Jenkins plugin, find it
[here](https://github.com/openshift/jenkins-plugin).  

## Configuring an OpenShift Cluster

## Setting up Credentials
