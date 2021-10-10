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

     }
}