pipeline {
     agent {
          kubernetes {
               cloud 'kubernetes'
               label kubeagent
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
                    sh "./gradlew compileJava"
               }
          }
          stage("Unit test") {
               when { expression { BRANCH_NAME ==~ /(main|future)/ } }
               steps {
                    sh "./gradlew test"
               }
          }
          stage("Code coverage") {
               when { branch 'main' }
               steps {
                    sh "./gradlew jacocoTestReport"
                    sh "./gradlew jacocoTestCoverageVerification"
               }
          }
          stage("Static code analysis") {
               when { expression { BRANCH_NAME ==~ /(main|future)/ } }
               steps {
                    sh "./gradlew checkstyleMain"
               }
          }
          stage("Package") {
               when { expression { BRANCH_NAME ==~ /(main|future)/ } }
               steps {
                    sh "./gradlew build"
               }
          }
          stage("Docker build main") {
               when { branch 'main' }
               steps {
                    sh "docker build -t sudheermanubolu/calculator:1.0 ."
               }
          }
          stage("Docker build future") {
               when { branch 'future' }
               steps {
                    sh "docker build -t sudheermanubolu/calculator-feature:0.1 ."
               }
          }
          stage("Docker login") {
               when { expression { BRANCH_NAME ==~ /(main|future)/ } }
               steps {
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'docker-hub-credentials',
                               usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                         sh "docker login --username $USERNAME --password $PASSWORD"
                    }
               }
          }
          stage("Docker push main") {
               when { branch 'main' }
               steps {
                    sh "docker push sudheermanubolu/calculator:1.0"
               }
          }
          stage("Docker push future") {
               when { branch 'future' }
               steps {
                    sh "docker push sudheermanubolu/calculator-feature:0.1"
               }
          }
     }
}