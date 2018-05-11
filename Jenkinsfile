

pipeline {
    agent {
        label 'stable'
    }
    stages {
        stage('test load custom') {
            steps {
                script {
                    loadCustomResource("some-file")
                }
            }
        }
    }
}