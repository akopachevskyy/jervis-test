

pipeline {
    agent {
        label 'stable'
    }
    stages {
        stage('register docker template') {
            steps {
                template = [
                    //DOCKER CONTAINER LIFECYCLE
                    docker_image_name: "jervis-docker-jvm:12",
                    //PULL IMAGE SETTINGS
                    //valid values: pull_latest, pull_always, pull_once, pull_never
                    pull_strategy: "pull_never",
                    //volumes can be a string or list of strings
                    //volumes: [ '/opt/jenkins-cache:/home/jenkins/.jenkins:rw', '/opt/gradle-cache:/home/jenkins/.gradle:rw' ],
                    //volumes_from can be a string or list of strings
                    //volumes_from: '',
                    //environment can be a string or list of strings
                    //environment: '',
                    //JENKINS SLAVE CONFIG
                    remote_fs_root: "/home/jenkins",
                    labels: "stable docker centos5 sudo language:shell language:groovy language:java env jdk",
                    //valid values: exclusive or normal
                    usage: "exclusive",
                    //LAUNCH METHOD
                    //valid values: launch_ssh or launch_jnlp
                    launch_method: "launch_jnlp",
                    //settings specific to launch_jnlp
                    launch_jnlp_linux_user: "jenkins",
                    launch_jnlp_slave_jar_options: "-workDir /home/jenkins",
                    launch_jnlp_lauch_timeout: 120,
                    launch_jnlp_different_jenkins_master_url: "http://192.168.89.18:8080",
                    launch_jnlp_ignore_certificate_check: false,
                ]
                registerDockerAgent(template)
            }
        }
    }
}