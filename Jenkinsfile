// <a.galvez.malpartida@accenture.com>

    pipeline {
        agent {
          kubernetes {
            label 'pipeline-as-code'
            defaultContainer 'jnlp'
            yaml """
              apiVersion: v1
              kind: Pod
              metadata:
                labels:
                  app: jenkins
              spec:
                containers:
                   - name: jnlp
                     image: jenkins/jnlp-slave:3.35-5
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
          AUTHOR = 'Alberto'
          // global
          build    = "true"
          unit     = "true"
          docker   = "true"
          deploy   = "true"
          email    = "false"

          gitBranch      = "${env.BRANCH_NAME}"
          gitCommit      = "${env.GIT_COMMIT}"
          shortGitCommit = "${gitCommit[0..10]}"
          // this variable get the last tag on the branch
          //release      = sh(returnStdout: true, script: 'git tag | head -1').trim()
          release        = "0.0.1"
          /*
          * these variables must be configured
          */

          // docker
          dockerfile    = ".deploy/Dockerfile"
          imageName     = "ceciliadominguez/private:${gitBranch}"
          registry      = 'https://registry.hub.docker.com'
          credentialsId = 'dockerhub'

          // k8s deploy
          appName        = "app-api-${gitBranch}"
          appChart       = ".deploy/helm"
          helmAppVersion = "1"
        }
      stages {
          stage('Debug') {
            steps {
                echo env.BRANCH_NAME
                echo gitBranch
            }
          }
          stage('Maven Build') {
            when {
              environment name: 'build', value: 'true'
            }
            steps {
              container('maven') {
                  sh """
                    mvn compile
                    mvn -B -DskipTests clean package
                  """
              }
            }
          }
          stage('Code Analysis') {
            parallel {
              stage ('Maven Test') {
                when {
                  environment name: 'unit', value: 'true'
                }
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
            when {
              environment name: 'docker', value: 'true'
            }
            steps {
              container('docker') {
                /*dockerImageCreate(dockerfile: "${dockerfile}", imageName: "${imageName}", \
                  registry: "${registry}", credentialsId: "${credentialsId}", gitBranch: "${gitBranch}", \
                  shortGitCommit: "${shortGitCommit}")*/
                  script{
                    docker.build("${imageName}", "-f ${dockerfile} $WORKSPACE")
                    sh """
                      sed -i.bak -e "s/master/${gitBranch}/" "${dockerfile}"
                      pwd
                    """
                    docker.withRegistry("${registry}", "${credentialsId}") {
                      app.push "${gitBranch}-${shortGitCommit}"
                      app.push "${gitBranch}"
                }
               }    
              }
            }
          }
          stage('Deploy to Kubernetes') {
            when {
              environment name: 'deploy', value: 'true'
            }
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
       /* post {
            success {
                mail to: "${email}",
                from: "jenkins@devsecopsascode.com",
                subject: "Successful Pipeline: ${currentBuild.fullDisplayName}",
                body: "Successful build completed ${env.BUILD_URL}"
            }
            failure {
                mail to: "${email}",
                from: "jenkins@devsecopsascode.com",
                subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
                body: "Something is wrong with ${env.BUILD_URL}"
            }
        }*/
    }