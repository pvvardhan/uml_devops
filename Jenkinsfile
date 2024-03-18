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
}
