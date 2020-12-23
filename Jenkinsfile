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


      // grab the list of tags starting with 'v' that reference this commit
      env.VERSION_GIT_TAG = sh(returnStdout: true, script: 'git tag -l v* --points-at HEAD')
      // strip leading 'v' off version tag

      sh "env"
      // build the image
      sh "podman build --format=docker -t ${imageName} ."

      // if we have any git version tags, add them as image tags
      if (env.VERSION_GIT_TAG) {
        // there may be multiple versions tagged with the same commit, use them all
        for( tag in env.VERSION_GIT_TAG.split('\n'))
          {
           def imageTag = tag.drop(1)
           sh "podman tag ${imageName} ${imageName}:${imageTag}"
          }
      }

      // always tag with latest
      sh "podman tag ${imageName} ${imageName}:latest"

      // trigger tagger job after build to pick up any extra tags
      build job: "tag_demo/qa", wait: true
      // push to dockerhub (for now)
      //sh "podman push --creds \"$HUB_LOGIN\" ${imageName} docker://docker.io/veupathdb/${imageName}"
    }
  }
}
