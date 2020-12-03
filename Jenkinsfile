def pipelineVersion='1.1.3'
println "Pipeline version: ${pipelineVersion}"
def buildAgentName(String jobNameWithNamespace, String buildNumber, String namespace) {
    def jobName = removeNamespaceFromJobName(jobNameWithNamespace, namespace);
    if (jobName.length() > 52) {
        jobName = jobName.substring(0, 52);
    }
    return "a.${jobName}${buildNumber}".replace('_', '-').replace('/', '-').replace('-.', '.');
}
def removeNamespaceFromJobName(String jobName, String namespace) {
    return jobName.replaceAll(namespace + "-", "").replaceAll(jobName + "/", "");
}
def buildSecretName(String jobNameWithNamespace, String namespace) {
    return jobNameWithNamespace.replaceFirst(namespace + "/", "").replaceFirst(namespace + "-", "").replace(".", "-").toLowerCase();
}
def secretName = buildSecretName(env.JOB_NAME, env.NAMESPACE)
println "Job name: ${env.JOB_NAME}"
println "Secret name: ${secretName}"
def buildLabel = buildAgentName(env.JOB_NAME, env.BUILD_NUMBER, env.NAMESPACE);
def branch = env.BRANCH ?: "main"
def namespace = env.NAMESPACE ?: "dev-tm"
def cloudName = "openshift"
def workingDir = "/home/jenkins/agent"
podTemplate(
   label: buildLabel,
   cloud: cloudName,
   yaml: """
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins-example
  volumes:
    - emptyDir: {}
      name: varlibcontainers
  containers:
    - name: jdk11
      image: jenkins/slave:latest-jdk11
      tty: true
      command: ["/bin/bash"]
      workingDir: ${workingDir}
      env:
        - name: HOME
          value: ${workingDir}
    - name: buildah
      image: quay.io/buildah/stable:v1.9.0
      tty: true
      command: ["/bin/bash"]
      workingDir: ${workingDir}
      securityContext:
        privileged: true
      env:
        - name: HOME
          value: /home/devops
        - name: ENVIRONMENT_NAME
          value: ${env.NAMESPACE}
        - name: DOCKERFILE
          value: ./Dockerfile
        - name: CONTEXT
          value: .
        - name: TLSVERIFY
          value: "false"
      volumeMounts:
        - mountPath: /var/lib/containers
          name: varlibcontainers
"""
) {
    node(buildLabel) {
        container(name: 'jdk11', shell: '/bin/bash') {
            git branch: 'main',
                url: 'https://github.com/rumblejake/mod-resorts.git'
        }
        
        container(name: 'buildah', shell: '/bin/bash') {
            stage('Build image') {
                sh '''#!/bin/bash
                    set -e
                    . ./env-config
                    echo TLSVERIFY=${TLSVERIFY}
                    echo CONTEXT=${CONTEXT}
                    APP_IMAGE="test:latest"
                    buildah bud --tls-verify=false --format=docker -f ${DOCKERFILE} -t ${APP_IMAGE} ${CONTEXT}
                    
                '''
            }
        }
    }
}