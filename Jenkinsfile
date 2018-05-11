

pipeline {
    agent {
        label 'stable'
    }
    stages {
        stage('test load custom') {
            steps {
                registerDockerAgent("some-file")
            }
        }
    }
}