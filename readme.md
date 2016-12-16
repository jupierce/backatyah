
# OpenShift Jenkins Pipeline (DSL) Plugin - Experimental

## Overview
The [OpenShift](https://www.openshift.com) [Pipeline](https://jenkins.io/solutions/pipeline/) 
DSL Plugin is presently an experimental Jenkins plugin which aims to provide a readable, concise, comprehensive, and fluent 
[Jenkins Pipeline](https://jenkins.io/doc/book/pipeline/) syntax for rich interactions with an OpenShift API Server.

Prerequisites:
* Familiarity with OpenShift [command line interface](https://docs.openshift.org/latest/cli_reference/basic_cli_operations.html)
is highly encouraged before exploring the plugin's features. The DSL leverages the [oc](https://docs.openshift.org/latest/cli_reference/index.html) 
binary and, in many cases, passes method arguments directly on to the command line. This document cannot, therefore,
provide a complete description of all possible OpenShift interactions -- the user may need to reference
the CLI documentation to find the pass-through arguments a given interaction requires.
* A fundamental level of understanding of the Jenkins [Pipeline](https://jenkins.io/solutions/pipeline/) architecture and 
[basic pipeline steps](https://jenkins.io/doc/pipeline/steps/workflow-basic-steps/) may be required to appreciate
the remainder of this document. Readers may also find it useful to understand basic [Groovy syntax](http://groovy-lang.org/syntax.html),
since pipeline scripts are written using Groovy (Note: Jenkins sandboxes and [interferes](https://issues.jenkins-ci.org/browse/JENKINS-26481) 
with the use of some Groovy facilities). 

As the DSL is designed to be intuitive for experienced OpenShift users, the following high level examples 
may serve to build that intuition before delving into the detailed documentation.

```groovy
/** Use of hostnames and OAuth token values in the DSL is heavily discouraged for maintenance and security reasons. **/
/** The global Jenkins configuration and credential store should be used instead. Subsequent examples **/
/** will demonstrate how to do this. **/
openshift.withCluster( 'https://https://10.13.137.207:8443', 'CO8wPaLV2M2yC_jrm00hCmaz5JgwTLzvAOHYNxxv6kE' ) {
    openshift.withProject( 'myproject' ) {
        // Create selector capable of selecting all service account objects
        def saSelector = openshift.selector( 'serviceaccount' )
        
        // Prints `oc describe serviceaccount` to Jenkins console
        saSelector.describe() 
        
        // Prints a list of current service accounts to the console
        echo "There are ${saSelector.count()} service accounts in project ${openshift.project()}: ${saSelector.names()}"
        
        // Create a selector that will always select the same named resource within a project; chain the delete operation
        openshift.selector( 'deploymentconfig/mydc' ).delete()
        
        // Create a selector which will select all deploymentconfigs with the metadata label value ('environment':'qa') 
        def qaDCs = openshift.selector( 'dc', [ environment:'qa' ] )
        
        // The objects selected by a selector can be iterated within the script
        qaDCs.withEach {
            // The conventional Groovy 'it' var is bound to a selector which selects a single object found by qaDCs
            echo "Found deployment config: ${it.name()}
            
            // The selector can model the selected object as a Groovy object. The model is a copy of the selected 
            // deployment config. Changes to it will not be reflected back to the server.
            def dc = it.object()
            
            // The model is a Groovy Map whose values consist of child Maps/Lists/Primitives
            // The script may directly interogate the values of the model according to the object's 
            // schema: https://docs.openshift.com/container-platform/3.3/dev_guide/deployments/how_deployments_work.html
            echo "The deployment config has labels: ${dc.metadata.labels}"
            
            // 
            dc.metadata.name = 
        }
    }
}
```

If you are interested in a non-experimental Jenkins plugin, find it
[here](https://github.com/openshift/jenkins-plugin).  

## Configuring an OpenShift Cluster

## Setting up Credentials
