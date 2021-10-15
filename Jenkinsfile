pipeline {
     agent any
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
          stage("Docker push") {
               when { branch 'main' }
               steps {
                    sh "docker push sudheermanubolu/calculator:1.0"
               }
          }
          stage("Docker push") {
               when { branch 'future' }
               steps {
                    sh "docker push sudheermanubolu/calculator-feature:0.1"
               }
          }
     }
}