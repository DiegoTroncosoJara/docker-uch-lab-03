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

        stage('install') {
            steps {
                container('node') {
                    sh '''
                        corepack enable
                        pnpm install --frozen-lockfile
                    '''
                }
            }
        }

        stage('test') {
            steps {
                container('node') {
                    sh 'pnpm test --runInBand'
                }
            }
        }
    }
}