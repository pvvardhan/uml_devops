 pipeline {
 agent {
 kubernetes {
 yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
    - name: gradle
      image: gradle
      command:
        - sleep
      args:
        - 99d
      volumeMounts:
        - name: shared-storage
          mountPath: /mnt
    - name: kaniko
      image: 'gcr.io/kaniko-project/executor:debug'
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
 stages {
 stage('Build a gradle project') {
 steps {
 git url: 'https://github.com/pvvardhan/Continuous-Delivery-with-Docker-and-Jenkins-Second-Edition.git', branch: 'master'
 container('gradle') {
 sh '''
 cd Chapter08/sample1
 sed -i 's/minimum = 0.2/minimum = 0.1/' build.gradle
 sed -i '/checkstyle {/,/}/d' build.gradle
 sed -i '/checkstyle/d' build.gradle
 cat build.gradle
 chmod +x gradlew
 ./gradlew build
 mv ./build/libs/calculator-0.0.1-SNAPSHOT.jar /mnt
 '''
 }
 }
 }
 stage('Run test and checksyle tests in non-main branch') {
 when { not { branch 'main' } }
 steps {
 echo 'Unit test and checkstyle execution in ${env.BRANCH_NAME}'
 catchError {
 sh """
 cd Chapter08/sample1;
 ./gradlew test
 ./gradlew checkstyleMain
 """
 }
 }
 }
 stage('Run code coverage in main branch only') {
 when { branch 'main' }
 steps {
 echo 'Running the code coverage in main branch alone'
 catchError {
 sh """
 cd Chapter08/sample1;
 ./gradlew test
 ./gradlew checkstyleMain
 ./gradlew jacocoTestCoverageVerification
 """
 }
 }
 }
 stage('Build Java Image') {
 steps {
 container('kaniko') {
 sh '''
 echo 'FROM openjdk:8-jre' > Dockerfile
 echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
 echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
 mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
 /kaniko/executor --context `pwd` --destination pvvardhan/hello-kaniko:1.0
 '''
 }
 }
 }
 }
 post {
 always {
 echo 'pipeline completed'
 }
 success {
 echo 'pipeline succeeded and proceeding to create container'
when {branch 'main'}
container('kaniko') {
 sh '''
 echo 'FROM openjdk:8-jre' > Dockerfile
 echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
 echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
 mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
 /kaniko/executor --context `pwd` --destination pvvardhan/calculator:1.0
 '''
 }
 }
 failure {
 echo 'pipeline failed and error handled by catchError block'
 }
 }
 }