Laboratorio 3 - Mi despliegue CI/CD
en Kubernetes
El desafío final consiste en llevar una aplicación web desde el código fuente hasta un
despliegue funcional en Kubernetes, usando Docker, registros de imágenes, Jenkins,
Jenkinsfile, credenciales y manifiestos Kubernetes.
La tarea es individual. No basta con copiar los ejemplos de clases: debes cambiar nombres,
imágenes, variables y configuraciones para demostrar que entiendes cómo se conectan las
piezas.
Objetivo del desafío
Debes construir un flujo completo que permita:
● Crear una imagen Docker propia para la aplicación entregada.
● Publicar la imagen en Docker Hub o GitHub Container Registry.
● Desplegar la aplicación en un cluster Kubernetes local.
● Automatizar el proceso con Jenkins usando agentes kubernetes.
● Demostrar que la aplicación funciona usando kubectl y curl.
Personalización obligatoria
Cada alumno debe usar nombres propios. Si te llamas Juan Perez, puedes usar el formato
juan-perez.
Ejemplo:
● Namespace: ns-juan-perez
● Deployment: app-juan-perez
● Service: svc-juan-perez
● ConfigMap: config-juan-perez
● Secret: secret-juan-perez
● Imagen Docker: usuario/tarea-final:juan-perez
● Imagen Docker: usuario/tarea-final:${APP_VERSION}
● APP_VERSION: 3.0.0
● Pipeline Jenkins: Jenkinsfile.Juan-Perez
Requisitos
La entrega debe cumplir con lo siguiente:

1. Usar un cluster local: Docker Desktop Kubernetes,kubeadm, Minikube, Kind, k3d u
   otro.
2. Crear un Dockerfile funcional y publicar la imagen en el registry de dockerhub y
   github
3. Crear Namespace, Deployment, Service, ConfigMap y Secret.
4. El Deployment debe usar la imagen publicada según tag de nombre (no versión) y
   tener al menos 2 réplicas. Puedes usar la de dockerhub o github, tu eliges.
5. La aplicación debe leer una variable desde un ConfigMap y un valor desde un
   Secret. Para el caso del configmap, debes guardar y leer la variable AMBIENTE, y
   en el caso del secreto la variable API_KEY
6. El Jenkinsfile debe tener stages: install,test, build, push y deploy.
7. El pipeline debe escribirse usando un agente de tipo kubernetes y escribiendo un
   agent.yaml
8. Las credenciales no deben quedar escritas directamente en el Jenkinsfile.
   Entregables
   Debes una carpeta comprimida con:
   ● Dockerfile
   ● .dockerignore
   ● Jenkinsfile
   ● Archivo entrega.yaml con los manifiestos Kubernetes
   ● Archivo agent.yaml con configuración de agente kuberentes.
   ● Log de pipeline de jenkins
   ● README.md con instrucciones de ejecución o instrucciones generales.
   ● Carpeta evidencias/ con capturas o salidas de comandos
   Evidencias obligatorias
   Incluye evidencia de estos comandos en tu terminal:
   ● kubectl cluster-info
   ● kubectl get nodes
   ● kubectl get pods -n ns-nombre-apellido
   ● kubectl get deployment -n app-nombre-apellido
   ● kubectl get svc -n svc-nombre-apellido
   ● kubectl logs deployment/app-nombre-apellido -n ns-nombre-apellido
   ● kubectl exec deployment/app-nombre-apellido -n ns-nombre-apellido --
   printenv
   ● kubectl get configmap config-nombre-apellido
   ● kubectl get secret secret-nombre-apellido
   También debes incluir evidencia del pipeline Jenkins ejecutado correctamente y una prueba
   de consulta a la aplicación con:
   ● kubectl port-forward svc/svc-nombre-apellido 8080:80 -n
   ns-nombre-apellido
   ● curl http://localhost:8080/lab
   Tips
   ● Válida manualmente docker build, docker push y kubectl apply antes de automatizar
   en Jenkins.
   ● Revisa que los labels del Deployment coincidan con el selector del Service.
   ● Si tu pod no inicia, usa kubectl describe pod y kubectl logs antes de modificar al
   azar.

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
