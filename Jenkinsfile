pipeline {
 agent {
 kubernetes {
 // Define the pod template with container template
 containerTemplate {
 name 'gradle'
 image 'gradle'
 command 'sleep'
 args '30d'
 }
 }
 }
 stages {
 stage('Checkout code and prepare environment') {
 steps {
 git url: 'https://github.com/pvvardhan/Continuous-Delivery-with-Docker-and-Jenkins-Second-Edition.git', branch: 'master'
 sh """
 cd Chapter08/sample1
 chmod +x gradlew
 """
 }
 }
 stage('Run pipeline against a gradle project- all branches') {
 steps {
 echo 'Unit test execution'
 sh """
 cd Chapter08/sample1;
 ./gradlew test
 """
 }
 }
 stage('Checkstyle Analysis') {
 steps {
 echo 'Running checkstyle rules'
 sh """
 cd Chapter08/sample1;
 ./gradlew checkstyleMain
 ./gradlew jacocoTestReport
 """
 publishHTML (
          target: [
            reportDir: 'Chapter08/sample1/build/reports/checkstyle',
            reportFiles: 'main.html',
            reportName: "JaCoCo Checkstyle Report"
          ]
        )
 }
 }
 stage('Run code coverage in main branch') {
 when { branch 'main' }
 steps {
 echo 'Running the code coverage in main branch'
 sh """
 cd Chapter08/sample1;
 ./gradlew jacocoTestCoverageVerification
 ./gradlew jacocoTestReport
 """
 publishHTML (
          target: [
            reportDir: 'Chapter08/sample1/build/reports/tests/test',
            reportFiles: 'index.html',
            reportName: "JaCoCo Code Coverage Report"
          ]
        )
 }
 }
 }
 post {
 always {
 echo 'pipeline completed'
 }
 }
 }