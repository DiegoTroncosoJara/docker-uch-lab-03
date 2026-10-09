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
        // Repositorio de destino en Docker Hub, sin etiqueta.
        DH_REPO = 'diegotroncoso/curso-03-uch-final-ghcr'
        // Repositorio de destino en GitHub Container Registry, sin etiqueta.
        GH_REPO = 'ghcr.io/diegotroncosojara/curso-03-uch-final-ghcr'
        // Namespace de la aplicacion que se actualizara durante el despliegue.
        K8S_NAMESPACE = 'ns-diego-troncoso'
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

        stage('CI - Creación de imagen') {
            steps {
                container('buildkit') {
                    sh '''
                        export DOCKER_CONFIG=/docker-config/dockerhub

                        # Construye y guarda la cache dentro del agente.
                        buildctl-daemonless.sh build \
                        --frontend dockerfile.v0 \
                        --local context=. \
                        --local dockerfile=. \
                        --export-cache type=local,dest=/tmp/buildkit-cache,mode=max
                    '''
                }
            }
        }

        stage('CD - upload de imagen') {
            steps {
                container('buildkit') {
                    sh '''
                        # Publicacion en Docker Hub.
                        export DOCKER_CONFIG=/docker-config/dockerhub
                        test -s "${DOCKER_CONFIG}/config.json"

                        buildctl-daemonless.sh build \
                        --frontend dockerfile.v0 \
                        --local context=. \
                        --local dockerfile=. \
                        --import-cache type=local,src=/tmp/buildkit-cache \
                        --output type=image,\\\"name=${DH_REPO}:diego-troncoso,${DH_REPO}:${BUILD_NUMBER}\\\",push=true

                        # Publicacion en GitHub Container Registry.
                        export DOCKER_CONFIG=/docker-config/github
                        test -s "${DOCKER_CONFIG}/config.json"

                        buildctl-daemonless.sh build \
                        --frontend dockerfile.v0 \
                        --local context=. \
                        --local dockerfile=. \
                        --import-cache type=local,src=/tmp/buildkit-cache \
                        --output type=image,\\\"name=${GH_REPO}:diego-troncoso,${GH_REPO}:${BUILD_NUMBER}\\\",push=true
                    '''
                }
            }
        }

        
        // ==============================================================================
        // CD: construir y publicar en dos registros
        // ==============================================================================
        // Esta etapa no tiene when: se ejecuta en todas las ramas que superan la CI.
        // stage("CD - Construccion imagen y upload"){
        //     steps{
        //         container("buildkit"){
        //             sh '''
        //                  # Selecciona la CARPETA que contiene config.json con autenticacion para Docker Hub.
        //                 export DOCKER_CONFIG=/docker-config/dockerhub
                        
        //                 # Falla si el archivo no existe o esta vacio, sin imprimir las credenciales.
        //                 test -s ${DOCKER_CONFIG}/config.json

        //                 # buildctl-daemonless.sh inicia BuildKit para esta construccion, sin usar el
        //                 # daemon Docker del host. Las barras finales continuan el comando en otra linea.
        //                 # --frontend dockerfile.v0 interpreta las instrucciones del Dockerfile.
        //                 # --local context=. envia el directorio actual como contexto de construccion.
        //                 # --local dockerfile=. indica donde encontrar el Dockerfile.
        //                 # --output type=image exporta una imagen; name contiene dos etiquetas del mismo
        //                 # repositorio. latest es mutable y BUILD_NUMBER identifica esta ejecucion Jenkins.
        //                 # push=true publica las etiquetas en el registro en lugar de solo construir.
        //                 # Las comillas escapadas mantienen unida la lista name con comas.
                        
        //                 buildctl-daemonless.sh build \
        //                 --frontend dockerfile.v0 \
        //                 --local context=. \
        //                 --local dockerfile=. \
        //                 --output type=image,\\\"name=${DH_REPO}:diego-troncoso,${DH_REPO}:${BUILD_NUMBER}\\\",push=true

        //                 # Cambia la carpeta de autenticacion para la segunda publicacion, esta vez en GHCR.
        //                 export DOCKER_CONFIG=/docker-config/github
        //                 # Comprueba tambien que la configuracion de GHCR exista y tenga contenido.
        //                 test -s ${DOCKER_CONFIG}/config.json

        //                 # Segunda llamada a BuildKit: construye y publica las dos etiquetas de GHCR.
        //                 # El archivo actual realiza dos construcciones; no es una copia entre registros.
                        
        //                 buildctl-daemonless.sh build \
        //                 --frontend dockerfile.v0 \
        //                 --local context=. \
        //                 --local dockerfile=. \
        //                 --output type=image,\\\"name=${GH_REPO}:diego-troncoso,${GH_REPO}:${BUILD_NUMBER}\\\",push=true
        //             '''
        //         }
        //     }
        // }

        // ==============================================================================
        // CD: actualizar el Deployment existente en Kubernetes
        // ==============================================================================
        stage('CD - Despliegue continuo'){
            // Condicion para ejecutar SOLO esta etapa; no restringe la publicacion anterior.
            // when {
            //     // Basta con que una de las condiciones de rama se cumpla.
            //     anyOf {
            //         // Permite desplegar desde main. branch se evalua en un job Multibranch Pipeline.
            //         branch 'main'
            //         // Tambien permite desplegar desde test.
            //         branch 'test'
            //     }
            // }
            steps{
                // Ejecuta kubectl en la imagen que contiene las herramientas de Kubernetes.
                container('kubectl-tool'){
                    // El plugin crea temporalmente un kubeconfig usando la credencial de Jenkins
                    // kubernetes-config. La identidad debe tener permisos RBAC en el namespace;
                    // este paso no crea por si solo el namespace, el Deployment ni los Secrets.
                    withKubeConfig([credentialsId: 'kubernetes-config']){
                        // El shell expande K8S_NAMESPACE, GH_REPO y BUILD_NUMBER definidos por Jenkins.
                        sh '''
                            kubectl -n "${K8S_NAMESPACE}" set image \
                            deployment/app-diego-troncoso \
                            app="${GH_REPO}:diego-troncoso"

                            kubectl -n "${K8S_NAMESPACE}" rollout restart \
                            deployment/app-diego-troncoso

                            kubectl -n "${K8S_NAMESPACE}" rollout status \
                            deployment/app-diego-troncoso --timeout=180s
                        '''
                    }
                }
            }
        }
    }
}