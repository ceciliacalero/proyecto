// <c.dominguez.calero@accenture.com>

    pipeline {
        agent {/*Crea un pod de kubernetes con contenedor maven, docker y helm, se usa docker in docker(De manera predeterminada, 
               el complemento Docker Pipeline se comunicará con un demonio Docker local, 
               al que normalmente se accede a través de /var/run/docker.sock.)*/
          kubernetes {
            label 'pipeline-as-code'
            yaml """
              apiVersion: v1 
              kind: Pod 
              metadata:
                labels:
                  app: jenkins
              spec:
                containers:
                   - name: maven 
                     image: maven:3.6.2-jdk-14
                     command:
                     - cat
                     tty: true
                   - name: docker
                     image: docker:stable-dind
                     command:
                     - cat
                     tty: true
                     volumeMounts:
                     - name: dockersock
                       mountPath: "/var/run/docker.sock"
                   - name: helm
                     image: alpine/helm:3.0.0
                     command:
                     - cat
                     tty: true
                volumes:
                - name: dockersock
                  hostPath: 
                    path: "/var/run/docker.sock"
            """
          }
        }

        environment {
        

          gitBranch      = "${BRANCH_NAME}" //nombre de la rama
          gitCommit      = "${GIT_COMMIT}" //devulve commit realizados
          shortGitCommit = "${gitCommit[0..10]}"
          // this variable get the last tag on the branch
          //release      = sh(returnStdout: true, script: 'git tag | head -1').trim()
          release        = "0.0.1" 
          /*
          * these variables must be configured
          */

          // docker
          dockerfile    = ".deploy/Dockerfile"  //ruta dockerfile
          imageName     = "ceciliadominguez/app:${gitBranch}" //nombre de la imagen
          registry      = "ceciliadominguez/app:${gitBranch}" //resgistro de dockerHub
          credentialsId = 'dockerhub' //credenciales de dockerhub en jenkins

          // k8s deploy
          appName        = "app-api-${gitBranch}" //nombre de la app
          appChart       = ".deploy/helm"  //ruta de helm
          helmAppVersion = "1"
        }
      stages {
          stage('Maven Build') {
            steps {
              container('maven') { //descargar las dependencias y construir la app 
                  sh """
                    mvn compile
                    mvn -B -DskipTests clean package
                  """
              }
            }
          }
          stage('Code Analysis') {  
            parallel {
              stage ('Maven Test') {//test unitarios
                steps {
                  container('maven') {
                   sh '''
                    mvn test
                    ls `pwd`/target
                   '''
                  }
                }
              }
            }
          }
          stage('Create Docker Image') {
            steps {
              container('docker') {
                  script{
                    def app = docker.build("${imageName}", "-f ${dockerfile} $WORKSPACE") //construye la imagen
                    sh """
                      sed -i.bak -e "s/mydemo/${gitBranch}/" "${dockerfile}" 
                      pwd
                    """
                    docker.withRegistry("", "${credentialsId}") { //login dockerHub
                      app.push "${gitBranch}-${shortGitCommit}" //subimos la imagen
                      app.push "${gitBranch}"
                }
               }    
              }
            }
          }
          stage('Deploy to Kubernetes') { // deliegue de la imagen   CD
          //1.cambio automatico de los valores de la app (expresiones regulares)
          //2.cat leemos el archivo y descargamos
          //3. creamos el pod
            steps {
              container('helm') {
                  sh """
                    sed -i -e "s/\\(tag:\\).*/\\1 ${gitBranch}/" ${appChart}/values.yaml
                    sed -i -e "s/\\(^appVersion:\\).*/\\1 ${helmAppVersion}/" ${appChart}/Chart.yaml
                    sed -i -e "s/\\(^name:\\).*/\\1 ${appName}/" ${appChart}/Chart.yaml
                    sed -i -e "s/\\(^version:\\).*/\\1 ${release}/" ${appChart}/Chart.yaml
                    cat ${appChart}/values.yaml 
                    cat ${appChart}/Chart.yaml
                    helm upgrade -i --recreate-pods ${appName} ${appChart}
                    helm list
                    ls
                    pwd
                  """
              }
            }
          }
        }
    }