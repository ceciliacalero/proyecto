# Proyecto DevOps

En este proyecto se puede crea un ciclo completo CI/CD de una aplicacion simple de Java, con un pipeline en Jenkins.

## Comenzando 🚀

_Estas instrucciones te permitirán obtener una copia del proyecto en funcionamiento en tu máquina local para propósitos de desarrollo y pruebas._


### Pre-requisitos 📋

_Que cosas necesitas para instalar el software 

```
Docker, Kubernetes y Jenkins.
```
### Instalación 🔧

_Arrancar manquina virtual_

```
Minikube Start
```

_Descargar Jenkins_

```
helm install --name jenkins --set rbac.create=false --set master.adminUser=username --set master.adminPassword=password stable/jenkins
```

_Finaliza con un ejemplo de cómo obtener datos del sistema o como usarlos para una pequeña demo_

## Despliegue 📦

_El despliegue se realiza automatico mediante la pipeline con Jenkins_

## Construido con 🛠️

* [Spring Boot](https://spring.io/projects/spring-boot) - El framework web usado
* [Maven](https://maven.apache.org/) - Manejador de dependencias y construcción de proyecto Java.
* [Helm](https://helm.sh/) - Usado para despliegue y versionado


## Wiki 📖

Puedes encontrar mucho más de cómo utilizar este proyecto en  [Wiki](https://github.com/ceciliacalero/docu-proyecto)

## Versionado 📌

Usamos [Helm](https://helm.sh/) para el versionado.

## Autor ✒️

**Cecilia Domínguez Calero** - [CeciliaCalero](https://github.com/ceciliacalero)


## Expresiones de Gratitud 🎁

* Comenta a otros sobre este proyecto 📢
* Invita una cerveza 🍺 o un café ☕ a alguien del equipo. 
* Da las gracias públicamente 🤓.



---
⌨️ con ❤️ por [CeciliaCalero](https://github.com/ceciliacalero) 😊
