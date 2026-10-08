pipeline {
    agent {
        kubernetes {
            cloud 'kubernetes'
            yamlFile 'agent.yaml'
        }
    }

    stages {
        stage('CI - Comprobar Agente') {
            steps {
                container('node') {
                    sh 'node --version'
                    sh 'npm --version'
                }
            }
        }

         // ==============================================================================
        // CI: activar el gestor de paquetes
        // ==============================================================================
        stage("CI - Activacion de pnpm"){
            // Pasos de esta etapa; sh ejecuta comandos en el contenedor seleccionado.
            steps{
                // Corepack habilita los ejecutables del gestor declarado en packageManager
                // de package.json; el proyecto fija pnpm@11.1.2.
                sh 'corepack enable'
                // Muestra la version de Node del agente, no la version de la imagen final de la API.
                sh 'node --version'
                // Permite comprobar en el log que se esta usando el pnpm esperado.
                sh 'pnpm --version'
            }
        }

        // ==============================================================================
        // CI: instalar exactamente las dependencias del lockfile
        // ==============================================================================
        stage("CI - Instalacion de dependencias"){
            steps{
                // No actualiza pnpm-lock.yaml; falla si no coincide con package.json.
                // Instala tambien dependencias de desarrollo necesarias para lint, Jest y build.
                sh 'pnpm install --frozen-lockfile'
            }
        }

        // ==============================================================================
        // CI: comprobar estilo y reglas de codigo
        // ==============================================================================
        stage("CI - Revision de Linter"){
            steps{
                // Ejecuta scripts.lint de package.json. No usa --fix: informa y falla,
                // pero no modifica el codigo para ocultar problemas durante la validacion.
                sh 'pnpm lint'
            }
        }
        // ==============================================================================
        // CI: ejecutar las pruebas automatizadas
        // ==============================================================================
        stage("CI - Ejecucion de Test"){
            steps{
                 // Ejecuta Jest mediante scripts.test, que habilita los modulos VM para NestJS 12.
                 // --runInBand usa un solo proceso para reducir memoria en el agente Node.
                 sh 'pnpm test --runInBand'
            }
        }
        // ==============================================================================
        // CI: compilar la aplicacion
        // ==============================================================================
        stage("CI - Construccion de aplicacion"){
            steps{
                 // Ejecuta nest build y genera dist/. Comprueba la compilacion antes de publicar.
                 sh 'pnpm build'
            }
        }
    }
}