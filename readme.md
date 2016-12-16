# OpenShift Pipeline (DSL) Plugin



<style>
    #openshift a:not([href]) /* Styles for anchors without href */ {
    display: block;
    position: relative;
    top: -50px;
    visibility: hidden;
    }
</style>

<div id="openshift">
    <a name="openshift"/>
    <h2>Global Variable: openshift</h2>
    <div style="margin-left: 1em;">

        <p>
            The <code>openshift</code> variable offers convenient access to Openshift-related functions from a Pipeline
            script. Each method will be documented using the following conventions:
            <p style="margin-left: 1em;">
                <b>Method with closure:</b>
                <code>receiver.methodName(requiredParameter:type[, optionalParameter:type]):returnType {…closure body…}</code>
                <br></br>
                <b>Method with return value:</b>
                <code>receiver.methodName(requiredParameter:type[, optionalParameter:type]):returnType</code>
                <br></br>
                <br></br>
                Return types will may be standard types (String, List<![CDATA[&lt;]]>String<![CDATA[&gt;]]>, bool, int, etc.) or complex types,
                specific to this plugin, which have behaviors of their own (e.g. <a href="#Result"><code>Result</code></a>,
                <a href="#Selector"><code>Selector</code></a>, <a href="#RolloutManager"><code>RolloutManager</code></a>). These return types are
                detailed after the methods have been described.
            </p>
        </p>
        <p style="border:1px;">
            <b>Note:</b> Unless otherwise noted, operations within this DSL must be performed within a <code>withCluster</code>
            closure. Exceptions include <code>openshift.logLevel</code>, <code>openshift.verbose</code>, and
            <code>openshift.selector</code>.
        </p>
        <p>
            Methods needing a Jenkins agent will implicitly run a <code>node {…}</code> block if you have not wrapped them in one.
            It is a good idea to enclose a block of steps which should all run on the same node in such a block yourself.
        </p>

        <dl>
            <a name="openshift_withCluster"/>
            <dt><code>openshift.withCluster([cluster:String[, credential:String]]) {…}</code></dt>
            <dd>
                <p>
                    Declares that OpenShift operations within the closure body should be executed against the
                    specified cluster. The statement also implicitly acts as both a
                    <a href="#openshift_withProject"><code>openshift.withProject</code></a> (establishing the default project within
                    the closure) and <a href="#openshift_doAs"><code>openshift.doAs</code></a> (establishing the default authorization
                    token within the closure).
                    <br></br>
                    <br></br>
                    <code>withCluster</code> may not be contained within a
                    <a href="#openshift_withProject"><code>openshift.withProject</code></a> or
                    <a href="#openshift_doAs"><code>openshift.doAs</code></a> closure.
                    <br></br>
                    <br></br>
                    When <code>withCluster</code> closures are nested within each other, OpenShift operations will use the
                    cluster information associated with the most tightly scoped occurrence.
                    <ul>
                        <li>
                            <b>cluster</b> - The name of an OpenShift cluster defined in the global configuration OR a String
                            literal URL (e.g. "https://10.13.137.186:8443"). The special protocol "insecure://" may also be used if https is
                            desired but TLS certificate verification should be disabled.
                            <br></br>
                            <br></br>
                            Use of the Jenkins global configuration method is highly encouraged as it will insulate your pipeline
                            scripts from changes to your server (e.g. a change in its IP address). It will also allow
                            <code>withCluster</code> to implicitly act as both
                            <a href="#openshift_withProject"><code>openshift.withProject</code></a> (changing the target project within
                            the closure) and <a href="#openshift_doAs"><code>openshift.doAs</code></a> (changing the authorization
                            token within the closure) if project and authorization information is associated with the cluster
                            configuration.
                            <br></br>
                            <br></br>
                            If the cluster argument is omitted, the plugin will try to derive cluster information in the following ways:
                            <ol>
                                <li>A lookup for a cluster named "default" in the Jenkins global configuration.</li>
                                <li>Any host information stored in the environment variables KUBERNETES_SERVICE_HOST:KUBERNETES_SERVICE_PORT_HTTPS .</li>
                            </ol>
                        </li>
                        <li>
                            <b>credential</b> - The credentialId of an OpenShift Auth token stored in the Jenkins credential
                            manager OR a String literal token value with which to authenticate. The use of the Jenkins credential
                            manager is highly encouraged because the token will be encrypted and, if the token value changes,
                            your scripts will not need to be updated. This parameter overrides any default credentials associated
                            with the cluster in the Jenkins global configuration.
                        </li>
                    </ul>
                </p>
            </dd>

            <a name="openshift_doAs"/>
            <dt><code>openshift.doAs(credential:String):Object {…}</code></dt>
            <dd>
                <p>
                    Specifies that OpenShift operations within the closure body should use the identified credential.
                    The return value is the value returned by (or the value of the last statement within) the closure.
                    <br></br>
                    <br></br>
                    When <code>doAs</code> closures are nested, OpenShift operations will use the
                    credential information associated with the most tightly scoped occurrence.
                    <br></br>
                    <br></br>
                    If no credential information is found within a given scope, it is assumed that Jenkins is running
                    within an OpenShift Pod. An attempt will be made to read a token from the Jenkins master
                    filesystem at <b>/run/secrets/kubernetes.io/serviceaccount/token</b>.
                    <ul>
                        <li>
                            <b>credential</b> - The credentialId of an OpenShift Auth token stored in the Jenkins credential
                            manager OR a String literal token value with which to authenticate. The use of the Jenkins credential
                            manager is highly encouraged because the token will be encrypted and, if the token value changes,
                            your scripts will not need to be updated.
                        </li>
                    </ul>
                </p>
            </dd>

            <a name="openshift_withProject"/>
            <dt><code>openshift.withProject(projectName:String):Object {…}</code></dt>
            <dd>

                <p>
                    <p  style="margin-left: 1em; color:#657383;">
                        <code>
                            def template = openshift.withProject( 'openshift' ) {
                            <![CDATA[&nbsp;&nbsp;&nbsp;&nbsp;]]>openshift.selector('template','mongodb-ephemeral').object()
                            }
                        </code>
                    </p>

                    Specifies that OpenShift operations within the closure body should target the identified project.
                    The return value is the value returned by (or the value of the last statement within) the closure.
                    <br></br>
                    <br></br>
                    When <code>withProject</code> closures are nested, OpenShift operations will use the
                    project information associated with the most tightly scoped occurrence.
                    <br></br>
                    <br></br>
                    If no project information is found within a given scope, it is assumed that Jenkins is running
                    within an OpenShift Pod. An attempt will be made to read the current project from the Jenkins master
                    filesystem at <b>/run/secrets/kubernetes.io/serviceaccount/project</b>.
                    <ul>
                        <li>
                            <b>projectName</b> - The name of the project to target for operations within the closure.
                        </li>
                    </ul>
                </p>
            </dd>


            <a name="openshift_selector"/>
            <dt>
                <code>openshift.selector(…):Selector</code>
            </dt>
            <dd>
                <p>
                    <code>selector</code> has multiple variations:
                    <ul>
                        <li>
                            <code>openshift.selector(kind:String):DynamicSelector</code><br></br>
                            <i style="margin-left: 1em; color:#657383;">Example: <code>openshift.selector("nodes")</code></i>
                        </li>
                        <li>
                            <code>openshift.selector(kind:String,labels:Map<![CDATA[&lt;]]>String,String<![CDATA[&gt;]]>):DynamicSelector</code><br></br>
                            <i style="margin-left: 1em; color:#657383;">Example: <code>openshift.selector("pod", [ alabel : "avalue", l2: "v2" ])</code></i>
                        </li>
                        <li>
                            <code>openshift.selector(kind:String,name:String):StaticSelector</code><br></br>
                            <i style="margin-left: 1em; color:#657383;">Example: <code>openshift.selector("dc", "frontend")</code></i>
                        </li>
                        <li>
                            <code>openshift.selector(qualifiedName:String):StaticSelector</code><br></br>
                            <i style="margin-left: 1em; color:#657383;">Example: <code>openshift.selector("dc/mysql")</code></i>
                        </li>
                    </ul>
                </p>
                <p>
                    <b>Context:</b> Does not need to be contained within <a href="#openshift_withCluster"><code>openshift.withCluster</code></a>.
                    <br></br>
                    <br></br>
                    Creates a <a href="#Selector">Selector</a> object (either a <a href="#DynamicSelector">DynamicSelector</a> or a
                    <a href="#StaticSelector">StaticSelector</a> depending on the invocation) which can be stored
                    away in a variable or used inline within the DSL. The creation of a Selector does not perform any
                    immediate operation on an OpenShift cluster -- it merely describes a grouping of objects which
                    can be subsequently be acted on by methods exposed by the Selector object.
                    <br></br>
                    <br></br>
                    Operations performed using a given Selector will be relative to the context in which those operations
                    are encountered. That is, the context of surrounding
                    <a href="#openshift_withCluster"><code>openshift.withCluster</code></a>,
                    <a href="#openshift_withProject"><code>openshift.withProject</code></a>,
                    <a href="#openshift_doAs"><code>openshift.doAs</code></a> closures. For example, a Selector
                    can be established once and subsequently used within a variety of
                    <code>openshift.withCluster</code> closures. Each time a method
                    is invoked on the Selector, the cluster affected will differ depending on the
                    <code>openshift.withCluster</code> which contains the invocation.
                    <ul>
                        <li>
                            <b>kind</b> - An OpenShift kind to select. Established
                            abbreviations are supported (e.g. "bc"->"buildconfig", "dc"->"deploymentconfig", etc.).
                        </li>
                        <li>
                            <b>name</b> - The name of an object which, when paired with a kind, uniquely identifies an object.
                        </li>
                        <li>
                            <b>labels</b> - A Groovy map of labels with which to refine a selector. Only resources containing
                            all label pairs will be selected.
                        </li>
                        <li>
                            <b>qualifiedName</b> - A qualified object name of the form "kind/name".
                        </li>
                    </ul>
                </p>
            </dd>


            <dt>
                <a name="openshift_create"/>
                <code>openshift.create(…):StaticSelector</code><br></br>
                <a name="openshift_replace"/>
                <code>openshift.replace(…):StaticSelector</code><br></br>
                <a name="openshift_apply"/>
                <code>openshift.apply(…):StaticSelector</code><br></br>
            </dt>
            <dd>
                <p>
                    <code>create</code>, <code>replace</code>, and <code>apply</code> have identical argument signatures. <code>create</code> is used in the variations below,
                    but the same pattern applies to each of these methods:
                    <ul>
                        <li>
                            <code>openshift.create(obj:Map[, args...:String]):StaticSelector</code><br></br>
                            <i style="margin-left: 1em; color:#657383;">Example: <code>openshift.create( [ kind : "DeploymentConfig", metadata: [ ... ], ... ] )</code></i>
                        </li>
                        <li>
                            <code>openshift.create(list:List<![CDATA[&lt;]]>Map<![CDATA[&gt;]]>[, args...:String]):StaticSelector</code><br></br>
                            <i style="margin-left: 1em; color:#657383;">Example: <code>openshift.create( [ objModel1, objModel1, ... ] )</code></i>
                        </li>
                        <li>
                            <code>openshift.create(markup:String[, args...:String]):StaticSelector</code><br></br>
                            <i style="margin-left: 1em; color:#657383;">Example: <code>openshift.create("{ \"metadata\" : { ... } ... ")</code></i>
                        </li>
                        <li>
                            <code>openshift.create(subVerb:String[, args...:String]):StaticSelector</code><br></br>
                            <i style="margin-left: 1em; color:#657383;">Example: <code>openshift.create( "serviceaccount", "jenkins" )</code></i>
                        </li>
                    </ul>
                </p>
                <p>
                    Requests OpenShift to create/replace/apply a new object. The object may be defined using a Goovy map or as a String
                    containing JSON or YAML. The method returns a StaticSelector containing the names of the objects created/modified.
                    <ul>
                        <li>
                            <b>obj</b> - A Groovy Map which models the content of the object. The model will be converted to
                            JSON prior to being sent to the API Server.
                        </li>
                        <li>
                            <b>list</b> - A List of Groovy Map objects which model the content of multiple OpenShift object. The models
                            will be merged into an OpenShift List and then converted to JSON prior to being sent to the API Server.
                        </li>
                        <li>
                            <b>markup</b> - JSON or YAML to send directly to the create API.
                        </li>
                        <li>
                            <b>subVerb</b> - An argument which is neither JSON or YAML. It will be passed directly to the main verb as an argument.
                        </li>
                        <li>
                            <b>args...</b> - A list of arguments which will be sent verbatim to the OpenShift command line tool.
                        </li>
                    </ul>
                </p>
            </dd>

            <a name="openshift_process"/>
            <dt>
                <code>openshift.process(…):Map</code>
            </dt>
            <dd>
                <p>
                    <code>process</code> has multiple variations:
                    <ul>
                        <li>
                            <code>openshift.process(json:String[,args...:String]):Map</code><br></br>
                            <i style="margin-left: 1em; color:#657383;">Example: <code>openshift.process( "{ \"metadata\": ... }", "-p", "PARAM=VALUE")</code></i><br></br>
                            <i style="margin-left: 1em; color:#657383;">Example: <code>openshift.process( readFile(file:'template.json'), "-p", "PARAM=VALUE")</code></i>
                        </li>
                        <li>
                            <code>openshift.process(obj:Map[,args...:String]):Map</code><br></br>
                            <i style="margin-left: 1em; color:#657383;">Example: <code>openshift.process( [ metadata: [... ] ],"-p", "PARAM=VALUE")</code></i>
                        </li>
                        <li>
                            <code>openshift.process(templateName:String[,args...:String]):Map</code><br></br>
                            <i style="margin-left: 1em; color:#657383;">Example: <code>openshift.process("openshift//foo","-p=PARAM=VALUE")</code></i>
                        </li>
                    </ul>
                </p>
                <p>
                    Processes and OpenShift template, using any specified parameter values, and returns a Groovy
                    Map modeling the resulting JSON. The Map can subsequently be passed to <code>openshift.create</code>,
                    <code>openshift.replace</code>, or <code>openshift.apply</code>.
                    <ul>
                        <li>
                            <b>json</b> - A literal string containing JSON to process.
                        </li>
                        <li>
                            <b>obj</b> - A Groovy map which models an OpenShift template.
                        </li>
                        <li>
                            <b>templateName</b> - The name of a template object stored in OpenShift.
                        </li>
                        <li>
                            <b>args...</b> - Arguments that will be passed verbatim to the OpenShift process facility.
                        </li>
                    </ul>
                </p>
            </dd>


            <dt>
                <a name="openshift_newProject"/>
                <code>openshift.newProject(name:String[,args...:String]):Result</code><br></br>
            </dt>
            <dd>
                <p>
                    <i style="margin-left: 1em; color:#657383;">Example: <code>openshift.newProject("ruby-temp","--display-name", "Ruby temporary")</code></i>
                    <br></br>
                    Creates a new OpenShift project.
                    <ul>
                        <li>
                            <b>name</b> - The name of the project to create.
                        </li>
                        <li>
                            <b>args...</b> - A list of arguments which will be sent verbatim to the OpenShift new-project tool.
                        </li>
                    </ul>
                </p>
            </dd>


            <dt>
                <a name="openshift_newApp"/>
                <code>openshift.newApp(args...:String):StaticSelector</code><br></br>
            </dt>
            <dd>
                <p>
                    <i style="margin-left: 1em; color:#657383;">Example: <code>openshift.newApp("https://github.com/openshift/ruby-hello-world")</code></i>
                    <br></br>
                    Invokes the OpenShift new-app facility. The Selector returned identifies the objects created by the request.
                    <ul>
                        <li>
                            <b>args...</b> - A list of arguments which will be sent verbatim to the OpenShift new-app tool.
                        </li>
                    </ul>
                </p>
            </dd>


            <dt>
                <a name="openshift_newBuild"/>
                <code>openshift.newBuild(args...:String):StaticSelector</code><br></br>
            </dt>
            <dd>
                <p>
                    <i style="margin-left: 1em; color:#657383;">Example: <code>openshift.newBuild(".","--docker-image=repo/langimage")</code></i>
                    <br></br>
                    Invokes the OpenShift new-build facility. The Selector returned identifies the objects created by the request.
                    <ul>
                        <li>
                            <b>args...</b> - A list of arguments which will be sent verbatim to the OpenShift new-build tool.
                        </li>
                    </ul>
                </p>
            </dd>

            <dt>
                <a name="openshift_startBuild"/>
                <code>openshift.startBuild(args...:String):StaticSelector</code><br></br>
            </dt>
            <dd>
                <p>
                    <i style="margin-left: 1em; color:#657383;">Example: <code>openshift.startBuild("--from-build=hello-world-1")</code></i>
                    <br></br>
                    Invokes the OpenShift start-build facility allowing the caller to specify command line arguments. This invocation
                    differs from the <a href="#Selector_startBuild"><code>Selector.startBuild</code></a> method as
                    the caller to this method must supply the BuildConfig name, if any.
                    <br></br>
                    The Selector returned identifies the objects created by the request.
                    <ul>
                        <li>
                            <b>args...</b> - A list of arguments which will be sent verbatim to the OpenShift start-build tool.
                        </li>
                    </ul>
                </p>
            </dd>


            <dt>
                <a name="openshift_exec"/>
                <code>openshift.exec(args...:String):Result</code><br></br>
                <a name="openshift_idle"/>
                <code>openshift.idle(args...:String):Result</code><br></br>
                <a name="openshift_import"/>
                <code>openshift._import(args...:String):Result</code><br></br>
                <a name="openshift_policy"/>
                <code>openshift.policy(args...:String):Result</code><br></br>
                <a name="openshift_run"/>
                <code>openshift.run(args...:String):Result</code><br></br>
                <a name="openshift_secrets"/>
                <code>openshift.secrets(args...:String):Result</code><br></br>
                <a name="openshift_tag"/>
                <code>openshift.tag(args...:String):Result</code><br></br>
                <a name="openshift_delete"/>
                <code>openshift.delete(args...:String):Result</code><br></br>
            </dt>
            <dd>
                <p>
                    <i style="margin-left: 1em; color:#657383;">Example: <code>openshift.tag("openshift/ruby:2.0", "yourproject/ruby:tip")</code></i>
                    <br></br>
                    Several operations: <code>exec</code>, <code>idle</code>, etc. have identical argument signatures and are
                    direct passthroughs to corresponding OpenShift command-line-interface. Each parameter will be passed
                    verbatim to the command line utility.<br></br>
                    <b>Note:</b> The "import" verb is preceded by an underscore (i.e. openshift._import(...)) because "import" is a reserved word.
                    <ul>
                        <li>
                            <b>args...</b> - A list of arguments which will be sent verbatim to the OpenShift tool.
                        </li>
                    </ul>
                </p>
            </dd>

            <a name="openshift_logLevel"/>
            <dt><code>openshift.logLevel(level:Integer):void</code></dt>
            <dd>
                <p>
                    <b>Context:</b> Does not need to be contained within <a href="#openshift_withCluster"><code>openshift.withCluster</code></a>.
                    <br></br>
                    <br></br>
                    Sets the logging level for all OpenShift operations subsequently executed. The
                    logging level is a global singleton and maintains its value until changed (i.e. it
                    is not affected by scopes like <a href="#openshift_withCluster"><code>openshift.withCluster</code></a>.
                    <br></br>
                    <br></br>
                    Setting a non-zero logging level increases the output sent to the Jenkins console by this plugin
                    and is also passed as the --loglevel argument to the underlying OpenShift tool.
                    <ul>
                        <li>
                            <b>level</b> - A value between 0 (disable logging) to 10 (maximum logging).
                        </li>
                    </ul>
                </p>
            </dd>


            <a name="openshift_verbose"/>
            <dt><code>openshift.verbose([enable:Boolean=true]):void</code></dt>
            <dd>
                <p>
                    <b>Context:</b> Does not need to be contained within <a href="#openshift_withCluster"><code>openshift.withCluster</code></a>.
                    <br></br>
                    <br></br>
                    Shorthand notation for <a href="#openshift_logLevel"><code>openshift.logLevel</code></a>.
                    <ul>
                        <li>
                            <b>enable</b> - Defaults to true if not specified. If true, equivalent to
                            <code>openshift.logLevel(8)</code>. If false, equivalent to <code>openshift.logLevel(0)</code>.
                        </li>
                    </ul>
                </p>
            </dd>
        </dl>
    </div>


    <a name="Result"/>
    <h2>Return Type: Result</h2>
    <div style="margin-left: 1em;">
        <p>
            Many operations within the OpenShift DSL return a <code>Result</code> object (a Selector is a
            subclass of Result). A Result object details whether a given operation was successful
            and any sub-actions that went into fulfilling the request.
            <br></br>
            <p  style="padding-left: 1em; color:#657383;">
                <code>
                    def result = openshift.selector( "dc" ).delete()  // Delete all deployment configs<br></br>
                    echo "Overall status: $${result.operation}" // The name of the operation performed (i.e. "delete")<br></br>
                    echo "Overall status: $${result.status}" // The number of sub-actions run<br></br>
                    echo "Actions performed: $${result.actions[0].cmd}" // First OpenShift command which was executed<br></br>
                    echo "Operation output: $${result.out}" // Aggregate output from all sub-actions<br></br>
                </code>
            </p>
        </p>
        <p>
            <b>Context:</b> Accesses to Result fields do not need to be contained within <a href="#openshift_withCluster"><code>openshift.withCluster</code></a>.
            <br></br>
            <br></br>
            The Result is effectively a data structure with the following members:
            <br></br>
            <ul>
                <li>
                    <code>Result.operation:String</code> - The high level operation that was performed. A high level
                    operation may be fulfilled by multiple underlying sub-actions.
                </li>
                <li>
                    <code>Result.out:String</code> - contains the aggregate stdout of the sub-actions
                    corresponding to an operation.
                </li>
                <li>
                    <code>Result.err:String</code> - contains the aggregate stderr of the sub-actions
                    corresponding to an operation.
                </li>
                <li>
                    <code>Result.status:int</code> - contains the aggregate (bitwise OR) status of the sub-actions
                    corresponding to an operation.
                </li>
                <li>
                    <code>Result.actions:List<![CDATA[&lt;]]>Action<![CDATA[&gt;]]></code> - A list of zero or more actions which took place to
                    satisfy the operation. Each Action object contains the following fields.
                    <ul>
                        <li>
                            <code>Action.verb:String</code> - the OpenShift tool verb executed.
                        </li>
                        <li>
                            <code>Action.cmd:String</code> - the approximate OpenShift command line executed (secrets redacted).
                        </li>
                        <li>
                            <code>Action.out:String</code> - the stdout of the OpenShift tool execution.
                        </li>
                        <li>
                            <code>Action.err:String</code> - the stderr of the OpenShift tool execution.
                        </li>
                        <li>
                            <code>Action.status:int</code> - the exit code of the OpenShift tool execution.
                        </li>
                    </ul>
                </li>
            </ul>
        </p>

    </div>


    <a name="Selector"/>
    <h2>Return Type: Selector (extends <a href="#Result">Result</a>)</h2>
    <div style="margin-left: 1em;">
        <p>
            Selectors identify a set of OpenShift resources on which operations can be performed. Consider the
            following example where a selector is created and used:
            <br></br>
            <p  style="margin-left: 1em; color:#657383;">
                <code>
                    def dcSelector = openshift.selector( "dc", [ app : "ruby" ] )  // Selector created<br></br>
                    def result = dcSelector.describe()   // 'oc describe' run against the selector<br></br>
                    dcSelector.delete()  // 'oc delete' run against the same selector<br></br>
                </code>
            </p>
            <a name="StaticSelector"/>
            <a name="DynamicSelector"/>
            <b>Advanced:</b> <b><code>StaticSelector</code></b> and <b><code>DynamicSelector</code></b> <br></br>
            For many pipelines, it is unnecessary to understand the difference between
            <code>StaticSelector</code> and <code>DynamicSelector</code>. In practice, the two subclasses
            of Selector perform an identical set of operations on the resources they select; however, complex scenarios may
            depend upon the differences in their behavior.
            <br></br>
            <ul>
                <li>
                    A <code>StaticSelector</code> contains a fixed set of qualified object names and will only ever
                    operate on those named objects.
                </li>
                <li>
                    A <code>DynamicSelector</code> contains object criteria
                    (e.g. a kind and a set of labels) which will be evaluated whenever
                    an operation is performed using the selector. As such, the selector may select different
                    objects depending on the state of the sever.
                </li>
            </ul>
            <br></br>
            The <a href="#Selector_freeze"><code>Selector.freeze</code></a> API can be used to convert a
            <code>DynamicSelector</code> into a <code>StaticSelector</code>. Consider the following example:
            <br></br>
            <p  style="margin-left: 1em; color:#657383;">
                <code>
                    def secretSelector = openshift.selector( 'secrets' ) // Creates a dynamic selector<br></br>
                    def staticSelector = secretSelector.freeze()  // Creates a point-in-time, static list of Secret objects<br></br>
                    openshift.create( ...new secret named XYZ... ) // Create a new Secret<br></br>
                    echo "The DynamicSelector found objects: $${secretSelector.names()}"   // List will contain XYZ<br></br>
                    echo "The StaticSelector found objects: $${staticSelector.names()}"  // List will not contain XYZ<br></br>
                </code>
            </p>
        </p>

        <dl>

            <dt>
                <a name="Selector_withEach"/>
                <code>Selector.withEach {…}</code><br></br>
            </dt>
            <dd>
                <p>
                    <p style="margin-left: 1em; color:#657383;">
                        Example:<br></br>
                        <code>
                            def list = openshift.newBuild("...").withEach {<br></br>
                            <![CDATA[&nbsp;&nbsp;&nbsp;&nbsp;]]>// The variable 'it' is bound to a StaticSelector containing a single name for each iteration <br></br>
                            <![CDATA[&nbsp;&nbsp;&nbsp;&nbsp;]]>echo "new-build created resource: $${it.name()}"<br></br>
                            }
                        </code>
                    </p>
                    <br></br>
                    Invokes the closure body once for each resource selected by the receiver. Before each iteration,
                    a new <code>StaticSelector</code> will be created which names on a single resource from the
                    selection.
                </p>
            </dd>


            <dt>
                <a name="Selector_narrow"/>
                <code>Selector.narrow(kind:String):StaticSelector</code><br></br>
            </dt>
            <dd>
                <p>
                    <p style="margin-left: 1em; color:#657383;">
                        Example: <code>def bc = openshift.newBuild("...").narrow("bc") // Creates selector which operates only on the BuildConfig returned by newBuild</code>
                    </p>
                    <br></br>
                    Creates a new Selector by filtering the objects selected by the receiver Selector based on Kind.
                    <ul>
                        <li>
                            <b>kind</b> - The kind (e.g. "deploymentconfig", "bc", etc.).
                        </li>
                    </ul>
                </p>
            </dd>


            <dt>
                <a name="Selector_related"/>
                <code>Selector.related(kind:String):DynamicSelector</code><br></br>
            </dt>
            <dd>
                <p>
                    <p style="margin-left: 1em; color:#657383;">
                        Example: <code>def builds = openshift.newBuild("...").narrow("bc").related("builds") // Selector will find builds created by BuildConfig</code>
                        Example: <code>def pods = openshift.newApp("...").narrow("dc").related("pods") // Selector will find pods created by DeploymentConfig</code>
                        Example: <code>def dcs = openshift.selector("template/xyz").related("dc") // Selector will find DeploymentConfigs created by a Template</code>
                    </p>
                    <br></br>
                    Creates a new Selector which finds objects of a specified kind which are related to the object
                    selected by the receiver. The receiver must only select a single object.<br></br>
                    This operation is knows how to find objects related to: DeploymentConfig, Template, and BuildConfig objects.
                    If any other kind is selected by the receiver, an error will be thrown.
                    <ul>
                        <li>
                            <b>kind</b> - The kind to find (e.g. "pod", "build", etc.).
                        </li>
                    </ul>
                </p>
            </dd>

            <dt>
                <a name="Selector_count"/>
                <code>Selector.count():Integer</code><br></br>
            </dt>
            <dd>
                <p>
                    <p style="margin-left: 1em; color:#657383;">Example:
                        <code>def n = openshift.newBuild("...").count() // Returns the number of objects created by newBuild</code>
                    </p>
                    <br></br>
                    Returns the number of objects the receiver selects. In the case of a DynamicSelector,
                    a query is made to the server to establish the current count. In this case of StaticSelectors,
                    this count is a constant (even if the objects it selects do not exist on the server).
                </p>
            </dd>

            <dt>
                <a name="Selector_names"/>
                <code>Selector.names():List<![CDATA[&lt;]]>String<![CDATA[&gt;]]></code><br></br>
            </dt>
            <dd>
                <p>
                    <p style="margin-left: 1em; color:#657383;">Example:
                        <code>def n = openshift.newBuild("...").names() // Returns a list of qualified object names created by newBuild</code>
                    </p>
                    <br></br>
                    Returns a list of names selected by the receiver (e.g. [ "pods/p1", "pods/p2" ] ).
                </p>
            </dd>


            <dt>
                <a name="Selector_name"/>
                <code>Selector.name():String</code><br></br>
            </dt>
            <dd>
                <p>
                    <p style="margin-left: 1em; color:#657383;">Example:
                        <code>def n = openshift.newBuild("...").narrow("bc").name() // Return the qualified name of the buildconfig created</code>
                    </p>
                    <br></br>
                    Returns the name of the single object selected by the receiver (e.g. "bc/ruby"). This method should
                    only be used if the caller is certain the receiver only selects a single object. If multiple
                    objects are selected, an error is thrown.
                </p>
            </dd>

            <dt>
                <a name="Selector_objects"/>
                <code>Selector.objects():List<![CDATA[&lt;]]>Map<![CDATA[&gt;]]></code><br></br>
            </dt>
            <dd>
                <p>
                    <p style="margin-left: 1em; color:#657383;">
                        Example:<br></br>
                        <code>
                            def list = openshift.newBuild("...").objects() // Returns a list of maps -- each modeling an object selected<br></br>
                            for ( obj in list ) {<br></br>
                            <![CDATA[&nbsp;&nbsp;&nbsp;&nbsp;]]>echo "newBuild created object of type: $${obj.kind}"<br></br>
                            }<br></br>
                        </code>
                    </p>
                    <br></br>
                    The <code>objects</code> method queries for the JSON definition of all objects selected by the
                    receiver and unmarshals that JSON into Groovy Maps. Those Maps are always returned in a list
                    (even if there are zero). The models are simple copies of the API server objects and modifications
                    are not reflected back to the server.
                </p>
            </dd>


            <dt>
                <a name="Selector_object"/>
                <code>Selector.objects():Map</code><br></br>
            </dt>
            <dd>
                <p>
                    <p style="margin-left: 1em; color:#657383;">
                        Example:<br></br>
                        <code>
                            def list = openshift.newBuild("...").narrow("bc").object() // Returns a model of the builcconfig created by newBuild<br></br>
                            echo "newBuild created buildconfig: $${obj.metadata.name}"<br></br>
                        </code>
                    </p>
                    <br></br>
                    The <code>object</code> method queries for the JSON definition of the singular object selected by the
                    receiver and unmarshals that JSON into a Groovy Map. The model is a simple copy of the API server
                    object and modifications are not reflected back to the server.<br></br>
                    This method should only be used if the caller is certain the receiver selects a single object. If multiple
                    objects are selected, an error is thrown.

                </p>
            </dd>


            <dt>
                <a name="Selector_freeze"/>
                <code>Selector.freeze():StaticSelector</code><br></br>
            </dt>
            <dd>
                <p>
                    Populates and returns a new StaticSelector by querying for objects selected by the receiver. If executed on
                    a StaticSelector receiver, <code>freeze</code> simply returns a copy of the receiver.
                </p>
            </dd>

            <dt>
                <a name="Selector_logs"/>
                <code>Selector.logs(args...:String):Result</code><br></br>
            </dt>
            <dd>
                <p>
                    Runs a logs operation against each object selected by the receiver. The logs will be captured in the
                    <code>Result</code> object and also streamed to the Jenkins console.
                    <ul>
                        <li>
                            <b>args...</b> - A optional list of arguments which will be appended directly to the logs invocation.
                        </li>
                    </ul>
                </p>
            </dd>


            <dt>
                <a name="Selector_describe"/>
                <code>Selector.describe(args...:String):Result</code><br></br>
            </dt>
            <dd>
                <p>
                    Runs a describe operation against each object selected by the receiver. The description will be captured in the
                    <code>Result</code> object and also streamed to the Jenkins console.
                    <ul>
                        <li>
                            <b>args...</b> - A optional list of arguments which will be appended directly to the describe invocation.
                        </li>
                    </ul>
                </p>
            </dd>


            <dt>
                <a name="Selector_delete"/>
                <code>Selector.delete([args...:String]):Result</code><br></br>
            </dt>
            <dd>
                <p>
                    Deletes the objects selected by the receiver.
                    <ul>
                        <li>
                            <b>args...</b> - A optional list of arguments which will be appended directly to the delete invocation.
                        </li>
                    </ul>
                </p>
            </dd>

            <dt>
                <a name="Selector_startBuild"/>
                <code>Selector.startBuild([args...:String]):StaticSelector</code><br></br>
            </dt>
            <dd>
                <p>
                    Executes start-build for each object selected by the receiver. Since this operation is only valid on BuildConfig
                    objects, <a href="#Selector_narrow"><code>Selector.narrow</code></a> should be used before this operation
                    if heterogeneous objects can be returned by a Selector. If non-BuildConfig objects are encountered,
                    the OpenShift tool will return an error and the overall operation will fail.
                    <br></br>
                    The Selector returned identifies the objects created by the request.
                    <ul>
                        <li>
                            <b>args...</b> - A optional list of arguments which will be appended directly to the
                            start-build invocation. Any arguments will supplement the BuildConfig name which will be
                            supplied automatically by the receiver.
                        </li>
                    </ul>
                </p>
            </dd>

            <dt>
                <a name="Selector_deploy"/>
                <code>Selector.deploy([args...:String]):Result</code><br></br>
            </dt>
            <dd>
                <p>
                    Executes deploy for each object selected by the receiver. Since this operation is only valid on DeploymentConfig
                    objects, <a href="#Selector_narrow"><code>Selector.narrow</code></a> should be used before this operation
                    if heterogeneous objects can be returned by a Selector. If non-DeploymentConfig objects are encountered,
                    the OpenShift tool will return an error and the overall operation will fail.
                    <ul>
                        <li>
                            <b>args...</b> - A optional list of arguments which will be appended directly to the
                            deploy invocation. Any arguments will supplement the DeploymentConfig name which will be
                            supplied automatically by the receiver.
                        </li>
                    </ul>
                </p>
            </dd>

            <dt>
                <a name="Selector_rollout"/>
                <code>Selector.rollout():RolloutManager</code><br></br>
            </dt>
            <dd>
                <p>
                    Returns a <a href="#RolloutManager"><code>RolloutManager</code></a> allowing the caller to perform
                    rollout-related operations on the objects selected by the receiver. If the receiver is a
                    <a href="#DynamicSelector"><code>DynamicSelector</code></a>, the RolloutManager will
                    perform a dynamic selection to satisfy each operation.
                    <br></br>
                </p>
            </dd>


            <dt>
                <a name="Selector_watch"/>
                <code>Selector.watch {…}</code><br></br>
            </dt>
            <dd>
                <p>
                    <p style="margin-left: 1em; color:#657383;">
                        Example:<br></br>
                        <code>
                            def nb = openshift.newBuild("https://github.com/openshift/ruby-hello-world", "--name=ruby") // Create a new buildconfig<br></br>
                            def buildSelector = nb.narrow("bc").related("builds")<br></br>
                            timeout(5) { // Throw exception after 5 minutes<br></br>
                            <![CDATA[&nbsp;&nbsp;&nbsp;&nbsp;]]>buildSelector.watch {<br></br>
                            <![CDATA[&nbsp;&nbsp;&nbsp;&nbsp;]]><![CDATA[&nbsp;&nbsp;&nbsp;&nbsp;]]>return (it.count() > 0)
                            <![CDATA[&nbsp;&nbsp;&nbsp;&nbsp;]]>}<br></br>
                            }<br></br>
                            echo "Builds have been created: $${buildSelector.names()}"<br></br>
                        </code>
                    </p>
                    <br></br>
                    The <code>watch</code> method listens for any changes from the objects selected by the receiver.
                    The closure body will be executed at least once before any changes have been detected. Any
                    subsequent watch events will re-invoke the body. Within the body, the <code>it</code> variable
                    will be bound to the receiver <code>Selector</code>.<br></br>
                    The watch body must return a boolean: <code>true</code> to exit the watch or <code>false</code>
                    to continue it.<br></br>
                    The author is encouraged to wrap this operation in a pipeline <code>timeout</code> step since there is no
                    guarantee the body will be executed again after the first invocation.<br></br>
                    If the API server gracefully disconnects the watch connection (e.g. due to watch inactivity),
                    the step will automatically and transparently reestablish the watch and re-invoke the body
                    in case changes took place during the disconnected interval.
                </p>
            </dd>

            <dt>
                <a name="Selector_untilEach"/>
                <code>Selector.untilEach([minimumCount:Integer=1) {…}</code><br></br>
            </dt>
            <dd>
                <p>
                    <p style="margin-left: 1em; color:#657383;">
                        Example:<br></br>
                        <code>
                            def nb = openshift.newBuild("https://github.com/openshift/ruby-hello-world", "--name=ruby") // Create a new buildconfig<br></br>
                            def buildSelector = nb.narrow("bc").related("builds")<br></br>
                            timeout(5) { // Throw exception after 5 minutes<br></br>
                            <![CDATA[&nbsp;&nbsp;&nbsp;&nbsp;]]>buildSelector.untilEach(1) {<br></br>
                            <![CDATA[&nbsp;&nbsp;&nbsp;&nbsp;]]><![CDATA[&nbsp;&nbsp;&nbsp;&nbsp;]]>return (it.object().status.phase == "Complete")<br></br>
                            <![CDATA[&nbsp;&nbsp;&nbsp;&nbsp;]]>}<br></br>
                            }<br></br>
                            echo "Builds have been completed: $${buildSelector.names()}"<br></br>
                        </code>
                    </p>
                    <br></br>
                    The <code>untilEach</code> provides a simple interface to ensure that all objects selected by
                    a receiver meet a user defined condition.<br></br>
                    Like <code>watch</code>, <code>untilEach</code> listens for any changes from the objects
                    selected by the receiver. When the number of objects is greater-than or equal to the
                    minimumCount parameter (defaults to 1), the body will be executed for each object.<br></br>
                    The <code>it</code> variable will be bound to a <code>Selector</code> for the single object to analyze
                    with the body invocation. If the selected object satisfies the user's condition, the body should
                    return true; if it does not, the body should return false.<br></br>
                    <code>untilEach</code> will not terminate until the number of objects selected by the receiver
                    is greater-than or equal to the minimumCount and the closure body returns true for each
                    selected object.
                </p>
            </dd>


        </dl>

    </div>

    <a name="RolloutManager"/>
    <h2>Return Type: RolloutManager</h2>
    <div style="margin-left: 1em;">
        <p>
            A <code>RolloutManager</code> object exposes methods which will run rollout related operations
            against a set of selected resources. A RolloutManager is created using
            <a href="#Selector_rollout"><code>Selector.rollout</code></a> . Before acquiring a <code>RolloutManager</code>,
            callers should ensure their <code>Selector</code> contains only appropriate resources (i.e. DeploymentConfigs).
            <br></br>
            <p  style="margin-left: 1em; color:#657383;">
                <code>
                    def dcSelector = openshift.selector( "dc", [ app : "ruby" ] )  // Selector created<br></br>
                    def rm = dcSelector.rollout()   // Create a RolloutManager which will act on all objects selected by the receiver<br></br>
                    def result = rm.history() // Gather history for the selected rollouts<br></br>
                    echo "DeploymentConfig history:\n$${result.out}"<br></br>
                    rm.pause() // Pause the selected rollouts<br></br>
                </code>
            </p>
        </p>

        <dl>

            <dt>
                <a name="RolloutManager_history"/>
                <code>RolloutManager.history([args...:String]):Result</code><br></br>
                <a name="RolloutManager_latest"/>
                <code>RolloutManager.latest([args...:String]):Result</code><br></br>
                <a name="RolloutManager_pause"/>
                <code>RolloutManager.pause([args...:String]):Result</code><br></br>
                <a name="RolloutManager_resume"/>
                <code>RolloutManager.resume([args...:String]):Result</code><br></br>
                <a name="RolloutManager_status"/>
                <code>RolloutManager.status([args...:String]):Result</code><br></br>
                <a name="RolloutManager_undo"/>
                <code>RolloutManager.undo([args...:String]):Result</code><br></br>
            </dt>
            <dd>
                <p>
                    <i style="margin-left: 1em; color:#657383;">Example: <code>openshift.selector("dc/nginx").rollout().undo("--to-revision=3")</code></i>
                    <br></br>
                    Several <code>RolloutManager</code> operations: <code>history</code>, <code>latest</code>, etc. have identical argument signatures and are
                    direct passthroughs to corresponding OpenShift command-line-interface. Each parameter will be passed
                    verbatim to the command line utility.<br></br>
                    <ul>
                        <li>
                            <b>args...</b> - A list of arguments which will be sent verbatim to the OpenShift tool.
                        </li>
                    </ul>
                </p>
            </dd>

        </dl>

    </div>


</div>


