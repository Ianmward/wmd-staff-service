def label = "mypod-${UUID.randomUUID().toString()}"
podTemplate(label: label, 
  envVars: [
    secretEnvVar(key: 'DOCKER_USR', secretName: 'docker-store-cred', secretKey: 'username'),
    secretEnvVar(key: 'DOCKER_PSW', secretName: 'docker-store-cred', secretKey: 'password'),
    secretEnvVar(key: 'NEXUS_USR', secretName: 'docker-nexus-cred', secretKey: 'username'),
    secretEnvVar(key: 'NEXUS_PSW', secretName: 'docker-nexus-cred', secretKey: 'password'),
    secretEnvVar(key: 'HARBOR_USR', secretName: 'docker-harbor-cred', secretKey: 'username'),
    secretEnvVar(key: 'HARBOR_PSW', secretName: 'docker-harbor-cred', secretKey: 'password'),
		envVar(key: 'REGISTRY', value: 'harbor.eks-iw.au-poc.com/library')
  ],
  yaml: """
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: test
spec:
  securityContext:
    runAsUser: 1724
    fsGroup: 1724
  containers:
  - name: ci-is
    image: harbor.eks-iw.au-poc.com/library/ci-is:10.5-betaa
    securityContext:
      runAsUser: 1724
      fsGroup: 1724
      allowPrivilegeEscalation: false
    envVars:
      - containerEnvVar:
          key: INSTANCE_NAME
          value: default
    command:
    - /opt/softwareag/IntegrationServer/bin/startContainer.sh
    tty: true
  - name: docker
    image: docker:18.02
    securityContext:
      runAsUser: 0
      fsGroup: 0
    command:
    - cat
    tty: true
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-sock-volume
  volumes:
  - name: docker-sock-volume
    hostPath:
      # location on host
      path: /var/run/docker.sock
      # this field is optional
      type: File
  imagePullSecrets:
  - name: regcred
"""
) {
    node(label) {
        def commitId
		stage ('test-docker') {
            container('docker') {
                sh "echo $commitId"
                sh "id"
                sh "ls -l /var/run/docker.sock"
			}
		}
        stage ('Extract') {
            checkout scm
            commitId = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        }
        def repository
        stage('Build'){
            container('ci-is') {
                sh "sh ./wait_for_is.sh"
                sh "/opt/softwareag/common/lib/ant/bin/ant -DSAGHome=/opt/softwareag -DSAG_CI_HOME=/opt/softwareag/sagdevops-ci-assets -DprojectName=${env.JOB_NAME} build"
            }
        }
        stage('Deploy') {
            container('ci-is') {
                sh "sh ./wait_for_is.sh"
                sh "/opt/softwareag/common/lib/ant/bin/ant -DSAGHome=/opt/softwareag -DSAG_CI_HOME=/opt/softwareag/sagdevops-ci-assets -DprojectName=${env.JOB_NAME} deploy"
            }
        }
        stage('Test') {
            container('ci-is') {
                sh "/opt/softwareag/common/lib/ant/bin/ant -DSAGHome=/opt/softwareag -DSAG_CI_HOME=/opt/softwareag/sagdevops-ci-assets -DprojectName=${env.JOB_NAME} test"
                junit 'report/'
            }
        }
        stage('Image') {
            container('docker') {
                sh "echo $commitId"
                sh "docker login -u ${env.HARBOR_USR} -p ${env.HARBOR_PSW} ${REGISTRY}"
                sh "cp target/${env.JOB_BASE_NAME}/build/IS/*.zip image/"
                sh "cd image; for pkg in *.zip; do basefilename=`echo \${pkg} | sed 's/.zip\$//'`; md5sum \${basefilename}.zip > \${basefilename}.md5; done"
                sh "cd image; docker build -t ${REGISTRY}/bookstore:${commitId} ."
                sh "docker push ${REGISTRY}/bookstore:${commitId}"
            }
        }
    }
}
