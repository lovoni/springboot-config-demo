#!/usr/bin/groovy
def failIfNoTests = ""
try {
  failIfNoTests = ITEST_FAIL_IF_NO_TEST
} catch (Throwable e) {
  failIfNoTests = "false"
}

def itestPattern = ""
try {
  itestPattern = ITEST_PATTERN
} catch (Throwable e) {
  itestPattern = "*KT"
}

def versionPrefix = ""
try {
  versionPrefix = VERSION_PREFIX
} catch (Throwable e) {
  versionPrefix = "1.0"
}

def canaryVersion = "${versionPrefix}.${env.BUILD_NUMBER}"
def utils = new io.fabric8.Utils()

node {
  def envStage = utils.environmentNamespace('staging')

  git 'http://gogs-default.vagrant.k8s/gogsadmin/awesome2.git'

  echo 'NOTE: running pipelines for the first time will take longer as build and base docker images are pulled onto the node'
  kubernetes.pod('buildpod').withImage('fabric8/maven-builder')
      .withPrivileged(true)
      .withHostPathMount('/var/run/docker.sock','/var/run/docker.sock')
      .withEnvVar('DOCKER_CONFIG','/home/jenkins/.docker/')
      .withSecret('jenkins-docker-cfg','/home/jenkins/.docker')
      .withSecret('jenkins-maven-settings','/root/.m2')
      .withServiceAccount('jenkins')
      .inside {

    stage 'Canary Release'
    mavenCanaryRelease{
      version = canaryVersion
    }

    stage 'Integration Test'
    mavenIntegrationTest{
      environment = 'Testing'
      failIfNoTests = localFailIfNoTests
      itestPattern = localItestPattern
    }

    stage 'Rolling Upgrade Staging'
    def rc = readFile 'target/classes/kubernetes.json'
    kubernetesApply(file: rc, environment: envStage)

  }
}
