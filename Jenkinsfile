#!/usr/bin/groovy
@Library('github.com/vpavlin/fabric8-pipeline-library@resource/200')

def versionPrefix = ""
try {
  versionPrefix = VERSION_PREFIX
} catch (Throwable e) {
  versionPrefix = "1.0"
}

def canaryVersion = "${versionPrefix}.${env.BUILD_NUMBER}"
def fabric8Console = "${env.FABRIC8_CONSOLE ?: ''}"
def utils = new io.fabric8.Utils()
def label = "buildpod.${env.JOB_NAME}.${env.BUILD_NUMBER}".replace('-', '_').replace('/', '_')

clientsTemplate{
  mavenNode{
    def currentNamespace = utils.getNamespace()
    def envStage = utils.environmentNamespace('staging')
    def envProd = utils.environmentNamespace('production')

    git 'https://github.com/vpavlin/vertx-web.git'
    def m = readMavenPom file: 'pom.xml'
    def artifactId = m.artifactId

    echo 'NOTE: running pipelines for the first time will take longer as build and base docker images are pulled onto the node'
    container(name: 'maven') {
      stage 'Build Release'
      mavenCanaryRelease {
        version = canaryVersion
      }
    }

    container(name: 'clients') {
      stage 'Rollout Staging'
      kubernetesApply(environment: envStage)
      sh "oc tag ${currentNamespace}/${artifactId}:${canaryVersion} ${envStage}/${artifactId}:${canaryVersion}"

      stage 'Rollout Production'
      kubernetesApply(environment: envProd)
      sh "oc tag ${currentNamespace}/${artifactId}:${canaryVersion} ${envProd}/${artifactId}:${canaryVersion}"
    }
  }
}


