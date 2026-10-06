# Laboratorio 3 - Mi Despliegue CI/CD en Kubernetes

## 1. Descripción

El desafío final consiste en llevar una aplicación web desde su código fuente hasta un despliegue funcional en **Kubernetes**, utilizando:

- Docker
- Docker Hub o GitHub Container Registry
- Kubernetes
- Jenkins
- Jenkinsfile
- Agentes Kubernetes
- Credenciales
- Manifiestos Kubernetes

La tarea es **individual**.

No basta con copiar los ejemplos desarrollados durante las clases. Cada alumno debe personalizar nombres, imágenes, variables y configuraciones para demostrar que comprende cómo se conectan todos los componentes del flujo CI/CD.

---

## 2. Objetivo

Se debe construir un flujo completo de CI/CD que permita:

1. Crear una imagen Docker propia para la aplicación entregada.
2. Publicar la imagen en:
   - Docker Hub, o
   - GitHub Container Registry.
3. Desplegar la aplicación en un cluster Kubernetes local.
4. Automatizar el proceso mediante Jenkins utilizando agentes Kubernetes.
5. Demostrar que la aplicación funciona correctamente mediante `kubectl` y `curl`.

---

## 3. Personalización obligatoria

Cada alumno debe utilizar nombres propios en los recursos creados.

Por ejemplo, para un alumno llamado **Juan Perez**, se puede utilizar el formato:

`juan-perez`

### Ejemplo de nombres

| Recurso            | Ejemplo                              |
| ------------------ | ------------------------------------ |
| Namespace          | `ns-juan-perez`                      |
| Deployment         | `app-juan-perez`                     |
| Service            | `svc-juan-perez`                     |
| ConfigMap          | `config-juan-perez`                  |
| Secret             | `secret-juan-perez`                  |
| Imagen Docker      | `usuario/tarea-final:juan-perez`     |
| Imagen con versión | `usuario/tarea-final:${APP_VERSION}` |
| APP_VERSION        | `3.0.0`                              |
| Jenkinsfile        | `Jenkinsfile.Juan-Perez`             |

---

## 4. Requisitos

La entrega debe cumplir con los siguientes requisitos.

### 4.1 Cluster Kubernetes

Utilizar un cluster Kubernetes local.

Se puede utilizar cualquiera de las siguientes alternativas:

- Docker Desktop Kubernetes
- kubeadm
- Minikube
- Kind
- k3d
- Otro cluster Kubernetes local

---

### 4.2 Docker

Crear un `Dockerfile` funcional para la aplicación.

La imagen debe ser publicada en un registry de contenedores:

- Docker Hub
- GitHub Container Registry

Antes de automatizar el proceso mediante Jenkins, se recomienda comprobar manualmente:

```bash
docker build
docker push
```

---

### 4.3 Recursos Kubernetes

Se deben crear los siguientes recursos:

- Namespace
- Deployment
- Service
- ConfigMap
- Secret

Todos los recursos deben estar definidos en el archivo:

```text
entrega.yaml
```

---

### 4.4 Deployment

El Deployment debe:

- Utilizar la imagen publicada en Docker Hub o GitHub Container Registry.
- Utilizar un tag basado en el nombre solicitado para la entrega.
- Tener al menos **2 réplicas**.
- Tener labels compatibles con el selector utilizado por el Service.

Ejemplo:

```yaml
spec:
  replicas: 2
```

---

### 4.5 ConfigMap

La aplicación debe leer la siguiente variable desde un `ConfigMap`:

```text
AMBIENTE
```

Ejemplo:

```text
AMBIENTE=produccion
```

---

### 4.6 Secret

La aplicación debe leer la siguiente variable desde un `Secret`:

```text
API_KEY
```

La API Key no debe quedar escrita directamente dentro del código de la aplicación ni dentro del Jenkinsfile.

---

## 5. Pipeline Jenkins

El pipeline debe estar definido mediante un `Jenkinsfile`.

Debe contener como mínimo los siguientes stages:

```text
install
test
build
push
deploy
```

El flujo esperado será:

```text
Código fuente
    ↓
Install
    ↓
Test
    ↓
Build Docker
    ↓
Push Registry
    ↓
Deploy Kubernetes
```

---

## 6. Agente Kubernetes de Jenkins

El pipeline debe utilizar un agente de tipo **Kubernetes**.

La configuración del agente debe estar definida en:

```text
agent.yaml
```

El Jenkinsfile debe utilizar este agente para ejecutar las distintas etapas del pipeline.

---

## 7. Credenciales

Las credenciales utilizadas para acceder al registry no deben quedar escritas directamente dentro del `Jenkinsfile`.

Las credenciales deben ser administradas mediante el sistema de credenciales de Jenkins.

Por ejemplo:

```text
Jenkins
→ Manage Jenkins
→ Credentials
```

Desde el Jenkinsfile se deben consumir mediante su identificador correspondiente.

---

## 8. Archivos entregables

La entrega debe realizarse mediante una carpeta comprimida.

La estructura esperada puede ser similar a:

```text
laboratorio-3/
│
├── Dockerfile
├── .dockerignore
├── Jenkinsfile
├── entrega.yaml
├── agent.yaml
├── README.md
├── pipeline.log
│
└── evidencias/
    ├── cluster-info.png
    ├── nodes.png
    ├── pods.png
    ├── deployment.png
    ├── service.png
    ├── logs.png
    ├── printenv.png
    ├── configmap.png
    ├── secret.png
    ├── pipeline-jenkins.png
    └── curl-aplicacion.png
```

Los nombres exactos de las capturas pueden variar, pero deben permitir identificar fácilmente cada evidencia.

---

## 9. Evidencias obligatorias

Se deben incluir capturas o salidas de los siguientes comandos.

### 9.1 Información del cluster

```bash
kubectl cluster-info
```

```bash
kubectl get nodes
```

---

### 9.2 Pods

```bash
kubectl get pods -n ns-nombre-apellido
```

Ejemplo:

```bash
kubectl get pods -n ns-juan-perez
```

---

### 9.3 Deployment

```bash
kubectl get deployment -n ns-nombre-apellido
```

También se puede consultar específicamente:

```bash
kubectl get deployment app-nombre-apellido -n ns-nombre-apellido
```

---

### 9.4 Service

```bash
kubectl get svc -n ns-nombre-apellido
```

También se puede consultar específicamente:

```bash
kubectl get svc svc-nombre-apellido -n ns-nombre-apellido
```

---

### 9.5 Logs de la aplicación

```bash
kubectl logs deployment/app-nombre-apellido -n ns-nombre-apellido
```

---

### 9.6 Variables de entorno

Se debe demostrar que las variables provenientes del `ConfigMap` y del `Secret` se encuentran disponibles dentro de los Pods.

```bash
kubectl exec deployment/app-nombre-apellido -n ns-nombre-apellido -- printenv
```

En la salida deberían aparecer, entre otras:

```text
AMBIENTE
API_KEY
```

---

### 9.7 ConfigMap

```bash
kubectl get configmap config-nombre-apellido -n ns-nombre-apellido
```

Para revisar su contenido:

```bash
kubectl describe configmap config-nombre-apellido -n ns-nombre-apellido
```

---

### 9.8 Secret

```bash
kubectl get secret secret-nombre-apellido -n ns-nombre-apellido
```

Para revisar su configuración:

```bash
kubectl describe secret secret-nombre-apellido -n ns-nombre-apellido
```

> Importante: evitar mostrar el valor real de `API_KEY` en las capturas entregadas.

---

## 10. Evidencia del Pipeline Jenkins

Se debe incluir evidencia de una ejecución correcta del pipeline Jenkins.

El pipeline debe completar correctamente todos sus stages:

```text
install
  ↓
test
  ↓
build
  ↓
push
  ↓
deploy
```

También se debe incluir el log de ejecución del pipeline.

Ejemplo:

```text
pipeline.log
```

---

## 11. Prueba de funcionamiento de la aplicación

Para acceder a la aplicación desplegada se debe realizar un `port-forward` desde Kubernetes hacia la máquina local.

Ejecutar:

```bash
kubectl port-forward svc/svc-nombre-apellido 8080:80 -n ns-nombre-apellido
```

Ejemplo:

```bash
kubectl port-forward svc/svc-juan-perez 8080:80 -n ns-juan-perez
```

Luego, desde otra terminal:

```bash
curl http://localhost:8080/lab
```

Se debe incluir evidencia de la respuesta entregada por la aplicación.

---

## 12. Flujo completo esperado

El flujo general de la solución debe ser:

```text
Repositorio Git
      │
      ▼
   Jenkins
      │
      ├── Install
      │
      ├── Test
      │
      ├── Build
      │      │
      │      ▼
      │   Docker Image
      │
      ├── Push
      │      │
      │      ▼
      │   Docker Hub / GHCR
      │
      └── Deploy
             │
             ▼
         Kubernetes
             │
       ┌─────┴─────┐
       │           │
       ▼           ▼
   ConfigMap     Secret
       │           │
       └─────┬─────┘
             ▼
        Deployment
             │
             ▼
          Service
             │
             ▼
       Aplicación Web
```

---

## 13. Validaciones recomendadas

Antes de ejecutar todo mediante Jenkins, validar manualmente cada parte.

### Construir la imagen

```bash
docker build -t usuario/tarea-final:nombre-apellido .
```

### Revisar la imagen

```bash
docker images
```

### Ejecutar la imagen localmente

```bash
docker run --rm -p 8080:80 usuario/tarea-final:nombre-apellido
```

### Publicar la imagen

```bash
docker push usuario/tarea-final:nombre-apellido
```

### Crear los recursos Kubernetes

```bash
kubectl apply -f entrega.yaml
```

### Revisar recursos

```bash
kubectl get all -n ns-nombre-apellido
```

---

## 14. Comandos útiles para solucionar problemas

### Revisar Pods

```bash
kubectl get pods -n ns-nombre-apellido
```

### Revisar detalles de un Pod

```bash
kubectl describe pod <nombre-pod> -n ns-nombre-apellido
```

### Revisar logs

```bash
kubectl logs <nombre-pod> -n ns-nombre-apellido
```

O mediante el Deployment:

```bash
kubectl logs deployment/app-nombre-apellido -n ns-nombre-apellido
```

### Revisar Deployment

```bash
kubectl describe deployment app-nombre-apellido -n ns-nombre-apellido
```

### Revisar Service

```bash
kubectl describe service svc-nombre-apellido -n ns-nombre-apellido
```

### Revisar todos los recursos

```bash
kubectl get all -n ns-nombre-apellido
```

---

## 15. Tips

- Validar manualmente `docker build`, `docker push` y `kubectl apply` antes de automatizar el proceso mediante Jenkins.
- Revisar que los labels del Deployment coincidan exactamente con el selector utilizado por el Service.
- Si un Pod no inicia, utilizar primero:

```bash
kubectl describe pod <pod>
```

y:

```bash
kubectl logs <pod>
```

- No modificar configuraciones al azar sin revisar antes el error entregado por Kubernetes.
- Verificar que Jenkins tenga acceso al cluster Kubernetes.
- Verificar que las credenciales del registry estén correctamente configuradas en Jenkins.
- Verificar que la imagen utilizada por el Deployment exista en el registry.
- Comprobar que las variables `AMBIENTE` y `API_KEY` estén disponibles dentro del contenedor mediante `printenv`.

---

## 16. Checklist final

Antes de entregar, verificar:

- [ ] Cluster Kubernetes funcionando.
- [ ] Dockerfile funcionando.
- [ ] `.dockerignore` creado.
- [ ] Imagen construida correctamente.
- [ ] Imagen publicada en Docker Hub o GHCR.
- [ ] Namespace creado.
- [ ] Deployment creado.
- [ ] Deployment con mínimo 2 réplicas.
- [ ] Service creado.
- [ ] ConfigMap creado.
- [ ] Variable `AMBIENTE` disponible en los Pods.
- [ ] Secret creado.
- [ ] Variable `API_KEY` disponible en los Pods.
- [ ] Jenkinsfile creado.
- [ ] `agent.yaml` creado.
- [ ] Pipeline utiliza agente Kubernetes.
- [ ] Stage `install` funcionando.
- [ ] Stage `test` funcionando.
- [ ] Stage `build` funcionando.
- [ ] Stage `push` funcionando.
- [ ] Stage `deploy` funcionando.
- [ ] Credenciales administradas desde Jenkins.
- [ ] Pipeline ejecutado exitosamente.
- [ ] Log del pipeline guardado.
- [ ] Evidencias de comandos `kubectl` guardadas.
- [ ] `port-forward` funcionando.
- [ ] `curl http://localhost:8080/lab` funcionando.
- [ ] Carpeta `evidencias/` completa.
- [ ] `README.md` incluido.

<p align="center">
  <a href="http://nestjs.com/" target="blank"><img src="https://nestjs.com/img/logo-small.svg" width="120" alt="Nest Logo" /></a>
</p>

[circleci-image]: https://img.shields.io/circleci/build/github/nestjs/nest/master?token=abc123def456
[circleci-url]: https://circleci.com/gh/nestjs/nest

  <p align="center">A progressive <a href="http://nodejs.org" target="_blank">Node.js</a> framework for building efficient and scalable server-side applications.</p>
    <p align="center">
<a href="https://www.npmjs.com/~nestjscore" target="_blank"><img src="https://img.shields.io/npm/v/@nestjs/core.svg" alt="NPM Version" /></a>
<a href="https://www.npmjs.com/~nestjscore" target="_blank"><img src="https://img.shields.io/npm/l/@nestjs/core.svg" alt="Package License" /></a>
<a href="https://www.npmjs.com/~nestjscore" target="_blank"><img src="https://img.shields.io/npm/dm/@nestjs/common.svg" alt="NPM Downloads" /></a>
<a href="https://circleci.com/gh/nestjs/nest" target="_blank"><img src="https://img.shields.io/circleci/build/github/nestjs/nest/master" alt="CircleCI" /></a>
<a href="https://discord.gg/G7Qnnhy" target="_blank"><img src="https://img.shields.io/badge/discord-online-brightgreen.svg" alt="Discord"/></a>
<a href="https://opencollective.com/nest#backer" target="_blank"><img src="https://opencollective.com/nest/backers/badge.svg" alt="Backers on Open Collective" /></a>
<a href="https://opencollective.com/nest#sponsor" target="_blank"><img src="https://opencollective.com/nest/sponsors/badge.svg" alt="Sponsors on Open Collective" /></a>
  <a href="https://paypal.me/kamilmysliwiec" target="_blank"><img src="https://img.shields.io/badge/Donate-PayPal-ff3f59.svg" alt="Donate us"/></a>
    <a href="https://opencollective.com/nest#sponsor"  target="_blank"><img src="https://img.shields.io/badge/Support%20us-Open%20Collective-41B883.svg" alt="Support us"></a>
  <a href="https://twitter.com/nestframework" target="_blank"><img src="https://img.shields.io/twitter/follow/nestframework.svg?style=social&label=Follow" alt="Follow us on Twitter"></a>
</p>
  <!--[![Backers on Open Collective](https://opencollective.com/nest/backers/badge.svg)](https://opencollective.com/nest#backer)
  [![Sponsors on Open Collective](https://opencollective.com/nest/sponsors/badge.svg)](https://opencollective.com/nest#sponsor)-->

## Description

[Nest](https://github.com/nestjs/nest) framework TypeScript starter repository.

## Project setup

```bash
$ pnpm install
```

## Compile and run the project

```bash
# development
$ pnpm run start

# watch mode
$ pnpm run start:dev

# production mode
$ pnpm run start:prod
```

## Run tests

```bash
# unit tests
$ pnpm run test

# e2e tests
$ pnpm run test:e2e

# test coverage
$ pnpm run test:cov
```

## Deployment

When you're ready to deploy your NestJS application to production, there are some key steps you can take to ensure it runs as efficiently as possible. Check out the [deployment documentation](https://docs.nestjs.com/deployment) for more information.

If you are looking for a cloud-based platform to deploy your NestJS application, check out [Mau](https://mau.nestjs.com), our official platform for deploying NestJS applications on AWS. Mau makes deployment straightforward and fast, requiring just a few simple steps:

```bash
$ pnpm install -g @nestjs/mau
$ mau deploy
```

With Mau, you can deploy your application in just a few clicks, allowing you to focus on building features rather than managing infrastructure.

## Resources

Check out a few resources that may come in handy when working with NestJS:

- Visit the [NestJS Documentation](https://docs.nestjs.com) to learn more about the framework.
- For questions and support, please visit our [Discord channel](https://discord.gg/G7Qnnhy).
- To dive deeper and get more hands-on experience, check out our official video [courses](https://courses.nestjs.com/).
- Deploy your application to AWS with the help of [NestJS Mau](https://mau.nestjs.com) in just a few clicks.
- Visualize your application graph and interact with the NestJS application in real-time using [NestJS Devtools](https://devtools.nestjs.com).
- Need help with your project (part-time to full-time)? Check out our official [enterprise support](https://enterprise.nestjs.com).
- To stay in the loop and get updates, follow us on [X](https://x.com/nestframework) and [LinkedIn](https://linkedin.com/company/nestjs).
- Looking for a job, or have a job to offer? Check out our official [Jobs board](https://jobs.nestjs.com).

## Support

Nest is an MIT-licensed open source project. It can grow thanks to the sponsors and support by the amazing backers. If you'd like to join them, please [read more here](https://docs.nestjs.com/support).

## Stay in touch

- Author - [Kamil Myśliwiec](https://twitter.com/kammysliwiec)
- Website - [https://nestjs.com](https://nestjs.com/)
- Twitter - [@nestframework](https://twitter.com/nestframework)

## License

Nest is [MIT licensed](https://github.com/nestjs/nest/blob/master/LICENSE).
