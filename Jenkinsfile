pipeline {
  agent none
  stages {
    stage('Build and Publish Docker Image') {
      agent {
        kubernetes {
          yaml """
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
              image: gcr.io/kaniko-project/executor:debug
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
          """
        }
      }
      steps {
        script {
          def imageName = "${env.BRANCH_NAME == 'master' ? 'calculator' : 'calculator-feature'}"
          def imageVersion = "${env.BRANCH_NAME == 'master' ? '1.0' : '0.1'}"
          try {
            docker.build("${imageName}:${imageVersion}")
            docker.withRegistry('https://index.docker.io/v1/', [credentialsId: 'f7bda929-2990-4167-b177-c7c79fed71c1', usernameVariable: 'somonedo', passwordVariable: 'dckr_pat_vopZPIuCcfE52TeFUdJEGRMuC-E']) {
              docker.push()
            }
          } catch (Exception e) {
            echo "Failed to build and publish Docker image: ${e}"
          }
        }
      }
    }
    stage('Build Java Image') {
      agent any
      steps {
        container('kaniko') {
          sh '''
          echo 'FROM openjdk:8-jre' > Dockerfile
          echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
          echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
          mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
          /kaniko/executor --context `pwd` --destination samuelomonedo247/hello-kaniko:1.0
          '''
        }
      }
    }
  }
}
