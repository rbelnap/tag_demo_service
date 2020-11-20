#!groovy

node('centos8') {
  def gitUrl = 'https://github.com/rbelnap/tag_demo_service.git'
  def imageName = 'tag_demo_service'

  stage('checkout') {
    checkout([
      $class: 'GitSCM',
      doGenerateSubmoduleConfigurations: false,
      extensions: [[
                     $class: 'SubmoduleOption',
                     disableSubModules: true,
                     noTags: false
                   ]],
      userRemoteConfigs:[[url: gitUrl]]
    ])

  }

  stage('build') {
    withCredentials([
      usernameColonPassword(
        credentialsId: '0f11d4d1-6557-423c-b5ae-693cc87f7b4b',
        variable: 'HUB_LOGIN'
      )
    ]) {


      env.VERSION_GIT_TAG = sh(returnStdout: true, script: 'git tag -l v* --points-at HEAD')

      sh "env"
      // build the image
      sh "podman build --format=docker -t ${imageName} ."

        sh "echo podman tag ${imageName} ${imageName}:${env.VERSION_GIT_TAG}"
        // push to dockerhub (for now)
        sh "echo podman tag ${imageName} ${imageName}:latest"

        //sh "podman push --creds \"$HUB_LOGIN\" ${imageName} docker://docker.io/veupathdb/${imageName}"
    }
  }
}
