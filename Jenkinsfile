podTemplate(yaml: '''
kind: Pod
spec:
  containers:
  - name: golang
    image: golang:1.19.2
    command:
    - cat
    tty: true
  - name: kaniko
    image: gcr.io/kaniko-project/executor:v1.9.1-debug 
    imagePullPolicy: Always
    command:
    - sleep
    args:
    - 99d
    volumeMounts:
      - name: jenkins-docker-cfg
        mountPath: /kaniko/.docker
  volumes:
  - name: jenkins-docker-cfg
    secret:
      secretName: regcred
      items:
        - key: .dockerconfigjson
          path: config.json
'''
)
{
  node(POD_LABEL) {
    stage('Checkout') {
      checkout scm
    }
    stage('Build Golang') {
      container('golang'){
        sh 'go build'
      }
    }
    stage('Build with Kaniko') {
      container('kaniko') {
        if ( env.BRANCH_NAME == 'master' || env.BRANCH_NAME == 'main' ) {
            echo "this is $BRANCH_NAME"
            sh '/kaniko/executor -f `pwd`/Dockerfile -c `pwd` --insecure --skip-tls-verify --cache=true --destination=ludesdeveloper/jenkins-golang-container-demo:latest'
        } else if ( env.BRANCH_NAME.contains("v.") ) {
            sh '/kaniko/executor -f `pwd`/Dockerfile -c `pwd` --insecure --skip-tls-verify --cache=true --destination=ludesdeveloper/jenkins-golang-container-demo:$BRANCH_NAME'
            echo "this is $BRANCH_NAME"
        }
      }
    }
  }
}