



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