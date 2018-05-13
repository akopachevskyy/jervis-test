

pipeline {
    agent {
        label 'stable'
    }
    stages {
        stage('register docker template') {
            steps {
                script {
                    def template = [
                        docker_image_name: "jervis-docker-jvm:latest",
                        pull_strategy: "pull_never",
                        remote_fs_root: "/home/jenkins",
                        labels: "kopachevsky2 stable docker ubuntu1604 sudo language:shell language:groovy language:java env jdk",

                        usage: "exclusive",
                        launch_method: "launch_jnlp",
                        launch_jnlp_linux_user: "jenkins",
                        launch_jnlp_slave_jar_options: "-workDir /home/jenkins",
                        launch_jnlp_lauch_timeout: 120,
                        launch_jnlp_different_jenkins_master_url: "http://172.22.28.12:8080",
                        launch_jnlp_ignore_certificate_check: false,
                    ]
                    registerDockerAgent(template)
                 }
            }
        }
    }
}