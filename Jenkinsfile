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
    stage('Build a gradle project') {
      git 'https://github.com/samuelomonedo247/Continuous-Delivery-with-Docker-and-Jenkins-Second-Edition.git'
      container('gradle') {
        stage('Build a gradle project') {
          sh '''
          cd Chapter08/sample1
          chmod +x gradlew
          ./gradlew build
          mv ./build/libs/calculator-0.0.1-SNAPSHOT.jar /mnt
          '''
        }
      }
    }

    stage('Build Java Image') {
      container('kaniko') {
        stage('Build a Java image') {
          sh '''
          echo 'FROM openjdk:8-jre' > Dockerfile
          echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
          echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
          mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
          /kaniko/executor --context `pwd` --destination somonedo/hello-kaniko:1.0
          '''
        }
      }
    }

    stage('Run pipeline against a gradle project - test MAIN') {
      container('gradle') {
        try {
          stage('Checkout code') {
            checkout scm
          }
          stage('Build a gradle project') {
            sh './gradlew build'
          }
          stage('Run tests') {
            if (env.BRANCH_NAME == "playground") {
              sh './gradlew test'
            } else if (env.BRANCH_NAME == "feature") {
              sh './gradlew test --exclude-task jacocoTestReport'
            } else if (env.BRANCH_NAME == "master") {
              sh './gradlew test'
            }
          }
          stage('Create container') {
            if (env.BRANCH_NAME != "playground") {
              def imageName = ""
              def version = ""
              if (env.BRANCH_NAME == "feature") {
                imageName = "calculator-feature"
                version = "0.1"
              } else if (env.BRANCH_NAME == "master") {
                imageName = "calculator"
                version = "1.0"
              }
              sh "docker build -t repository/${imageName}:${version} ."
            }
          }
        } catch (e) {
          currentBuild.result = '
