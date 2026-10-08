// ==============================================================================
// Pipeline principal: integracion continua (CI) y entrega/despliegue (CD)
// ==============================================================================
// CI: instala dependencias, ejecuta lint y pruebas, y compila NestJS.
// CD: construye y publica imagenes en Docker Hub y GHCR; main y test despliegan.
// Las etapas se ejecutan en orden. Un sh que termina con codigo distinto de cero
// falla el paso y normalmente impide continuar con las etapas siguientes.
//
// Requisitos de Jenkins: Declarative Pipeline, Kubernetes plugin para el agente
// y Kubernetes CLI plugin para withKubeConfig. El checkout debe incluir
// agent-node.yaml, Dockerfile y los archivos de dependencias de la aplicacion.
// regcred-dh y regcred-gh son Secrets del namespace del agente, montados en
// BuildKit; kubernetes-config es una credencial de Jenkins para el despliegue.
// Son mecanismos distintos: publicar una imagen no concede permisos en Kubernetes.
//
// Documentacion: https://www.jenkins.io/doc/book/pipeline/syntax/
// Plugins: https://plugins.jenkins.io/kubernetes/
//          https://plugins.jenkins.io/kubernetes-cli/
pipeline {
    agent {
        // El plugin Kubernetes crea un Pod temporal para ejecutar el pipeline.
        kubernetes {
            cloud 'kubernetes'
            // Los sh sin container(...) se ejecutan en node-tool.
            defaultContainer 'node-tool'
            yamlFile 'agent.yaml'
        }
    }

    environment {
        // Repositorio de destino en GitHub Container Registry, sin etiqueta.
        GH_REPO = 'ghcr.io/diegotroncosojara/curso-03-uch-final-ghcr'
        // Namespace de la aplicacion que se actualizara durante el despliegue.
        K8S_NAMESPACE = 'ns-diego-troncoso'
        GHCR = credentials('ghcr-credentials')
    }

    stages {
        stage('CI - Comprobar Agente') {
            steps {
                sh 'node --version'
                sh 'npm --version'
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

         // ==============================================================================
        // CI: TEST DE LOS SECRETOS
        // ==============================================================================
        stage('CI - Comprobar secrets de registry') {
            steps {
                container('buildkit') {
                    sh '''
                        test -s /docker-config/github/config.json
                        test -s /docker-config/dockerhub/config.json
                        echo "Ambos archivos de autenticacion estan disponibles"
                    '''
                }
            }
        }
        // ==============================================================================
        // CD: construir y publicar en dos registros
        // ==============================================================================
        // Esta etapa no tiene when: se ejecuta en todas las ramas que superan la CI.
        stage("CD - Construccion imagen y upload"){
            steps{
                container("buildkit"){
                    sh '''
                        set +x

                        export DOCKER_CONFIG="$(mktemp -d)"
                        trap 'rm -rf "$DOCKER_CONFIG"' EXIT

                        AUTH="$(printf '%s:%s' "$GHCR_USR" "$GHCR_PSW" \
                            | base64 | tr -d '\\n')"

                        printf '{"auths":{"ghcr.io":{"auth":"%s"}}}' "$AUTH" \
                            > "$DOCKER_CONFIG/config.json"

                        buildctl-daemonless.sh build \
                        --frontend dockerfile.v0 \
                        --local context=. \
                        --local dockerfile=. \
                        --output "type=image,name=${GH_REPO}:diego-troncoso-${BUILD_NUMBER},push=true"
                    '''
                }
            }
        }
    }
}