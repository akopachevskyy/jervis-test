

import com.github.kostyasha.yad.DockerCloud
import com.github.kostyasha.yad.DockerConnector
import com.github.kostyasha.yad.DockerContainerLifecycle
import com.github.kostyasha.yad.DockerGlobalConfiguration
import com.github.kostyasha.yad.DockerSlaveTemplate
import com.github.kostyasha.yad.commons.DockerCreateContainer
import com.github.kostyasha.yad.commons.DockerImagePullStrategy
import com.github.kostyasha.yad.commons.DockerPullImage
import com.github.kostyasha.yad.commons.DockerRemoveContainer
import com.github.kostyasha.yad.commons.DockerStopContainer
import com.github.kostyasha.yad.launcher.DockerComputerJNLPLauncher
import com.github.kostyasha.yad.launcher.DockerComputerLauncher
import com.github.kostyasha.yad.launcher.DockerComputerSSHLauncher
import com.github.kostyasha.yad.other.ConnectorType
import com.github.kostyasha.yad.other.cloudorder.RandomLeastLoadedDockerCloudOrder
import com.github.kostyasha.yad.strategy.DockerOnceRetentionStrategy

import hudson.model.Node
import hudson.plugins.sshslaves.SSHConnector
import hudson.slaves.EnvironmentVariablesNodeProperty
import hudson.slaves.NodeProperty
import hudson.slaves.RetentionStrategy
import hudson.tools.ToolLocationNodeProperty
import jenkins.model.Jenkins
import net.sf.json.JSONArray
import net.sf.json.JSONObject

//return a launcher
def selectLauncher(String launcherType, JSONObject obj) {
    switch(launcherType) {
        case 'launch_ssh':
            SSHConnector sshConnector = new SSHConnector(obj.optInt('launch_ssh_port', 22),
                    obj.optString('launch_ssh_credentials_id'),
                    obj.optString('launch_ssh_jvm_options'),
                    obj.optString('launch_ssh_java_path'),
                    obj.optString('launch_ssh_prefix_start_slave_command'),
                    obj.optString('launch_ssh_suffix_start_slave_command'),
                    obj.optInt('launch_ssh_connection_timeout'),
                    obj.optInt('launch_ssh_max_num_retries'),
                    obj.optInt('launch_ssh_time_wait_between_retries')
            )
            return new DockerComputerSSHLauncher(sshConnector)
        case 'launch_jnlp':
            DockerComputerJNLPLauncher dockerComputerJNLPLauncher = new DockerComputerJNLPLauncher()
            dockerComputerJNLPLauncher.setUser(obj.optString('launch_jnlp_linux_user','jenkins'))
            dockerComputerJNLPLauncher.setLaunchTimeout(obj.optLong('launch_jnlp_launch_timeout', 120L))
            dockerComputerJNLPLauncher.setSlaveOpts(obj.optString('launch_jnlp_slave_jar_options'))
            dockerComputerJNLPLauncher.setJvmOpts(obj.optString('launch_jnlp_slave_jvm_options'))
            dockerComputerJNLPLauncher.setJenkinsUrl(obj.optString('launch_jnlp_different_jenkins_master_url'))
            dockerComputerJNLPLauncher.setNoCertificateCheck(obj.optBoolean('launch_jnlp_ignore_certificate_check', false))
            return dockerComputerJNLPLauncher
        default:
            return null
    }
}

//create a new instance of the DockerCloud class from a JSONObject
def newDockerCloud(JSONObject obj) {
    DockerConnector connector = new DockerConnector(obj.optString('docker_url', 'tcp://localhost:2375'))
    if(obj.optInt('connect_timeout')) {
        connector.setConnectTimeout(obj.optInt('connect_timeout'))
    }
    connector.setApiVersion(obj.optString('docker_api_version'))
    connector.setCredentialsId(obj.optString('host_credentials_id'))
    //select connection_type
    List<String> connection_types = ['NETTY', 'JERSEY']
    String connection_type_default = 'NETTY'
    String user_selected_connection_type = obj.optString('connection_type', connection_type_default).toUpperCase()
    String connection_type = (user_selected_connection_type in connection_types)? user_selected_connection_type : connection_type_default
    connector.setConnectorType(ConnectorType."${connection_type}")

    DockerCloud cloud = new DockerCloud(obj.optString('cloud_name'),
            bindJSONToList(DockerSlaveTemplate.class, obj.opt('docker_templates')),
            obj.optInt('max_containers', 50),
            connector)

    return cloud
}

def newDockerSlaveTemplate(JSONObject obj) {
    //DockerPullImage
    DockerPullImage pullImage = new DockerPullImage()
    String pullStrategy = obj.optString('pull_strategy', 'PULL_LATEST').toUpperCase()
    if(pullStrategy in ['PULL_LATEST', 'PULL_ALWAYS', 'PULL_ONCE', 'PULL_NEVER']) {
        pullImage.setPullStrategy(DockerImagePullStrategy."${pullStrategy}")
    }
    else {
        pullImage.setPullStrategy(DockerImagePullStrategy.PULL_LATEST)
    }
    pullImage.setCredentialsId(obj.optString('pull_registry_credentials_id'))

    //DockerCreateContainer
    DockerCreateContainer createContainer = new DockerCreateContainer()
    createContainer.setBindPorts(obj.optString('port_bindings'))
    createContainer.setBindAllPorts(obj.optBoolean('bind_all_declared_ports', false))
    createContainer.setDnsString(obj.optString('dns'))
    createContainer.setHostname(obj.optString('hostname'))
    if(obj.optLong('memory_limit_in_mb')) {
        createContainer.setMemoryLimit(obj.optLong('memory_limit_in_mb'))
    }
    createContainer.setPrivileged(obj.optBoolean('run_container_privileged', false))
    createContainer.setTty(obj.optBoolean('allocate_pseudo_tty', false))
    if(obj.optJSONArray('volumes')) {
        createContainer.setVolumes(obj.optJSONArray('volumes') as List<String>)
    }
    else {
        createContainer.setVolumesString(obj.optString('volumes'))
    }
    if(obj.optJSONArray('volumes_from')) {
        createContainer.setVolumesFrom(obj.optJSONArray('volumes_from') as List<String>)
    }
    else {
        createContainer.setVolumesFromString(obj.optString('volumes_from'))
    }
    createContainer.setMacAddress(obj.optString('mac_address'))
    if(obj.optInt('cpu_shares')) {
        createContainer.setCpuShares(obj.optInt('cpu_shares'))
    }
    createContainer.setCommand(obj.optString('docker_command'))
    if(obj.optJSONArray('environment')) {
        createContainer.setEnvironment(obj.optJSONArray('environment') as List<String>)
    }
    else {
        createContainer.setEnvironmentString(obj.optString('environment'))
    }
    if(obj.optJSONArray('extra_hosts')) {
        createContainer.setExtraHosts(obj.optJSONArray('extra_hosts') as List<String>)
    }
    else {
        createContainer.setExtraHostsString(obj.optString('extra_hosts'))
    }
    createContainer.setNetworkMode(obj.optString('network_mode'))
    if(obj.optJSONArray('devices')) {
        createContainer.setDevices(obj.optJSONArray('devices'))
    }
    else {
        createContainer.setDevicesString(obj.optString('devices'))
    }
    createContainer.setCpusetCpus(obj.optString('cpuset_constraint_cpus'))
    createContainer.setCpusetMems(obj.optString('cpuset_constraint_mems'))
    if(obj.optJSONArray('links')) {
        createContainer.setLinks(obj.optJSONArray('links'))
    }
    else {
        createContainer.setLinksString(obj.optString('links'))
    }

    //DockerStopContainer
    DockerStopContainer stopContainer = new DockerStopContainer()
    stopContainer.setTimeout(obj.optInt('stop_container_timeout', 10))

    //DockerRemoveContainer
    DockerRemoveContainer removeContainer = new DockerRemoveContainer()
    removeContainer.setRemoveVolumes(obj.optBoolean('remove_volumes', false))
    removeContainer.setForce(obj.optBoolean('force_remove_containers', false))

    //DockerContainerLifecycle
    DockerContainerLifecycle dockerContainerLifecycle = new DockerContainerLifecycle()
    dockerContainerLifecycle.setImage(obj.optString('docker_image_name'))
    dockerContainerLifecycle.setPullImage(pullImage)
    dockerContainerLifecycle.setCreateContainer(createContainer)
    dockerContainerLifecycle.setStopContainer(stopContainer)
    dockerContainerLifecycle.setRemoveContainer(removeContainer)

    //DockerOnceRetentionStrategy
    //this availability_strategy is for "run_once".  We can customize it later
    RetentionStrategy retentionStrategy = new DockerOnceRetentionStrategy(obj.optInt('availability_idle_timeout', 10))

    //DockerComputerLauncher
    //select a launch method from the list of available launch methods
    List<String> launch_methods = ['launch_ssh', 'launch_jnlp']
    String default_launch_method = 'launch_jnlp'
    String user_selected_launch_method = obj.optString('launch_method', default_launch_method).toLowerCase()
    String launch_method = (user_selected_launch_method in launch_methods)? user_selected_launch_method : default_launch_method
    DockerComputerLauncher launcher = selectLauncher(launch_method, obj)

    //DockerSlaveTemplate
    DockerSlaveTemplate dockerSlaveTemplate = new DockerSlaveTemplate()
    dockerSlaveTemplate.setDockerContainerLifecycle(dockerContainerLifecycle)
    dockerSlaveTemplate.setLabelString(obj.optString('labels', 'docker'))
    String node_usage = (obj.optString('usage', 'EXCLUSIVE').toUpperCase().equals('NORMAL'))? 'NORMAL' : 'EXCLUSIVE'
    dockerSlaveTemplate.setMode(Node.Mode."${node_usage}")
    dockerSlaveTemplate.setNumExecutors(obj.optInt('executors', 1))
    dockerSlaveTemplate.setRetentionStrategy(retentionStrategy)
    dockerSlaveTemplate.setLauncher(launcher)
    dockerSlaveTemplate.setRemoteFs(obj.optString('remote_fs_root', '/home/jenkins'))
    dockerSlaveTemplate.setMaxCapacity(obj.optInt('max_instances', 10))
    //define NODE PROPERTIES
    List<NodeProperty> nodeProperties = [] as List<NodeProperty>
    if(obj.optJSONObject('environment_variables')) {
        HashMap<String,String> env = obj.optJSONObject('environment_variables') as HashMap<String,String>
        List<EnvironmentVariablesNodeProperty.Entry> envEntries = [] as List<EnvironmentVariablesNodeProperty.Entry>
        (env.keySet() as String[]).each { var ->
            envEntries << (new EnvironmentVariablesNodeProperty.Entry(var, env[var]))
        }
        //add environment_variables to nodeProperties
        nodeProperties << (new EnvironmentVariablesNodeProperty(envEntries))
    }
    if(obj.optJSONObject('tool_locations')) {
        HashMap<String,String> tool = obj.optJSONObject('tool_locations') as HashMap<String,String>
        List<ToolLocationNodeProperty.ToolLocation> toolLocations = [] as List<ToolLocationNodeProperty.ToolLocation>
        (tool.keySet() as String[]).each { location ->
            //tool location is only valid if it contains a type i.e. @ symbol
            if(location.contains('@') && detectGlobalToolExists(location)) {
                toolLocations << (new ToolLocationNodeProperty.ToolLocation(location, tool[location]))
            }
            else {
                //alert the user they configured an invalid tool type or name in tool_locations
                println "WARNING: Invalid tool: '${location}'.  Format should be 'type@name' where both type and name already exist in global tool configurations."
            }
        }
        nodeProperties << (new ToolLocationNodeProperty(toolLocations))
    }
    //set NODE PROPERTIES
    if(nodeProperties) {
        dockerSlaveTemplate.setNodeProperties(nodeProperties)
    }
    return dockerSlaveTemplate
}


def bindJSONToList(Class type, Object src) {
    if(!(type == DockerCloud) && !(type == DockerSlaveTemplate)) {
        throw new Exception("Must use DockerCloud or DockerSlaveTemplate class.")
    }
    //docker_array should be a DockerCloud or DockerSlaveTemplate
    ArrayList<?> docker_array
    if(type == DockerCloud){
        docker_array = new ArrayList<DockerCloud>()
    }
    else {
        docker_array = new ArrayList<DockerSlaveTemplate>()
    }
    //cast the configuration object to a Docker instance which Jenkins will use in configuration
    if (src instanceof JSONObject) {
        //uses string interpolation to call a method
        //e.g instead of newDockerCloud(src) we use instead...
        docker_array.add("new${type.getSimpleName()}"(src))
    }
    else if (src instanceof JSONArray) {
        for (Object o : src) {
            if (o instanceof JSONObject) {
                docker_array.add("new${type.getSimpleName()}"(o))
            }
        }
    }
    return docker_array
}

def getNewTemplate(Object clouds_yadocker) {
    cloud = bindJSONToList(DockerCloud.class, clouds_yadocker)
    assert cloud.templates.size() == 1, "Templates number should be 1, but got " + cloud.templates.size()
    return cloud.templates[0]
}

def overrideOldTemplate(String imageFullName,  Object newTemplate, DockerCloud cloud) {
    String imageName = imageFullName[0..imageFullName.indexOf(":") - 1]
    println "Templates size before removal ${cloud.templates.size()}"


    oldTemplates = []

    cloud.templates*.each {
        println "ImageName ${it.dockerContainerLifecycle.image}"
        if(it.dockerContainerLifecycle.image.startsWith(imageName + ":")) {
            println "Found existing template ${it}"
            oldTemplates << it
        }
    }

    assert oldTemplates.size() <= 1, "Found more than one existing templates"

    oldTemplates.each {
        println "Removing existing template ${it}"
        cloud.removeTemplate(it)
    }

    println "Templates size after removal ${cloud.templates.size()}"

    println "adding new template"

    cloud.addTemplate(newTemplate)

    println "Templates size after adding new ${cloud.templates.size()}"
}

def getDockerCloud() {
    def clouds = Jenkins.instance.clouds*.find {it instanceof DockerCloud }
    assert clouds.size() == 1, "Docker clouds size should be 1, but got" + clouds.size()
    println "Found Docker Cloud with name ${clouds[0].name}"
    return clouds[0]
}



pipeline {
    agent {
        label 'stable'
    }
    stages {
        stage('register docker template') {
            steps {
                registerDockerAgent("test")
            }
        }
    }
}