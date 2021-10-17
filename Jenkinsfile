pipeline {
     agent {
          kubernetes {
               cloud 'kubernetes'
               label 'temp'
               yaml """
  apiVersion: v1
  kind: Pod
  spec:
    containers:
    - name: gradle
      image: gradle:6.3-jdk14
      command: ["sleep"]
      args: ["99d"]
      volumeMounts:
      - name: shared-storage
        mountPath: /mnt
    - name: kaniko
      image: gcr.io/kaniko-project/executor:debug
      command: ["sleep"]
      args: ["9999999"]
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
     triggers {
          pollSCM('* * * * *')
     }
     stages {
          stage("Compile") {
               when { expression { BRANCH_NAME ==~ /(main|future)/ } }
               steps {
                    container("gradle"){
                    sh "./gradlew compileJava"
                    }
               }
          }
          stage("Unit test") {
               when { expression { BRANCH_NAME ==~ /(main|future)/ } }
               steps {
                    container("gradle"){
                    sh "./gradlew test"
               }
               }
          }
          stage("Code coverage") {
               when { branch 'main' }
               steps {
                    container("gradle"){
                    sh "./gradlew jacocoTestReport"
                    sh "./gradlew jacocoTestCoverageVerification"
               }
               }
          }
          stage("Static code analysis") {
               when { expression { BRANCH_NAME ==~ /(main|future)/ } }
               steps {
                    container("gradle"){
                    sh "./gradlew checkstyleMain"
               }
               }
          }
          stage("Package") {
               when { expression { BRANCH_NAME ==~ /(main|future)/ } }
               steps {
                    container("gradle"){
                    sh "./gradlew build"
                    sh "mv ./build/libs/calculator-0.0.1-SNAPSHOT.jar /mnt"
               }
               }
          }
          stage("create Dockerfile") {
               when { expression { BRANCH_NAME ==~ /(main|future)/ } }
               steps {
                    container("kaniko"){
                    sh "echo 'FROM openjdk:8-jre' > Dockerfile"
                    sh "echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile"
                    sh "echo 'ENTRYPOINT [\"java\", \"-jar\", \"app.jar\"]' >> Dockerfile"
                    sh "mv /mnt/calculator-0.0.1-SNAPSHOT.jar ./calculator-0.0.1-SNAPSHOT.jar"
               }
               }
          }
          stage("kaniko build main") {
               when { branch 'main' }
               steps {
                    container("kaniko"){
                    sh "/kaniko/executor --context `pwd` --destination sudheermanubolu/calculator:1.0"
               }
               }
          }
          stage("Docker build future") {
               when { branch 'future' }
               steps {
                    container("kaniko"){
                    sh "/kaniko/executor --context `pwd` --destination sudheermanubolu/calculator-feature:0.1 ."
               }
               }
          }
     }
}