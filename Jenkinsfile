pipeline {
    agent {
        kubernetes {
            cloud 'kubernetes'
            yamlFile 'agent.yaml'
        }
    }

    stages {
        stage('comprobar agente') {
            steps {
                container('node') {
                    sh 'node --version'
                    sh 'npm --version'
                }
            }
        }
    }
}