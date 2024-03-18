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
 when not { branch 'main' }
 steps {
 echo 'Unit test and checkstyle execution in non-main branch'
 catchError {
 sh """
 cd Chapter08/sample1;
 ./gradlew test
 ./gradlew checkstyleMain
 """
 }
 }
 }
 }
}
