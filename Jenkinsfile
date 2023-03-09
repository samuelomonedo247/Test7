pipeline {
  agent any
  stages {
    stage('Build and Test') {
      steps {
        script {
          podTemplate(yaml: '''
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: gradle
                image: gradle:6.3-jdk14
                command:
                - sleep
                args:
                - 99d
                volumeMounts:
                - name: shared-storage
                  mountPath: /mnt        
              - name: kaniko
                image: gcr.io/kaniko-project/executor:debug-v0.16.0
                command:
                - sleep
                args:
                - 9999999
                volumeMounts:
                - name: shared-storage
                  mountPath: /mnt
                - name: kaniko-secret
                  mountPath: /kaniko/.docker
              restartPolicy: Never
              volumes:
              - name: shared-storage
                persistentVolumeClaim:
                  claimName: jenkins-pv-claim
              - name: kaniko-secret
                secret:
                    secretName: dockercred
                    items:
                    - key: .dockerconfigjson
                      path: config.json
          ''') {
            node(POD_LABEL) {
              def branchName = env.BRANCH_NAME
              def imageName = branchName == 'master' ? 'calculator' : 'calculator-feature'
              def imageTag = branchName == 'master' ? '1.0' : '0.1'
              def buildSucceeded = false

              try {
                stage('Build and Test') {
                  git 'https://github.com/samuelomonedo247/Continuous-Delivery-with-Docker-and-Jenkins-Second-Edition.git'
                  container('gradle') {
                    sh '''
                      cd Chapter08/sample1
                      ./gradlew clean build
                      '''
                  }
                  container('gradle') {
                    sh '''
                      cd Chapter08/sample1
                      ./gradlew test
                      '''
                  }
                  if (branchName != 'playground') {
                    container('gradle') {
                      sh '''
                        cd Chapter08/sample1
                        ./gradlew codeCoverageReport
                        '''
                    }
                  }
                  buildSucceeded = true
                }
              } catch (e) {
                echo "Build and Test failed: ${e.message}"
              }

              if (buildSucceeded && branchName != 'playground') {
                stage('Build Docker Image') {
                  container('kaniko') {
                    sh '''
                      cd Chapter08/sample1
                      echo 'FROM openjdk:8-jre' > Dockerfile
                      echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
                      echo "ENTRYPOINT ['java', '-jar', 'app.jar']" >> Dockerfile
					  mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
                      /kaniko/executor --context `pwd` --destination somonedo/hello-kaniko:1.0
                      '''
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
