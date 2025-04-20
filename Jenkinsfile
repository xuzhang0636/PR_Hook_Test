pipeline {
  agent any
  
  stages {
    stage('Checkout & Prepare') {
      steps {
        script {
          // Print all environment variables using Groovy
          println "Environment variables:"
          env.each { key, value ->
            println "${key} = ${value}"
          }
          
          // Print GitHub related information
          println "\nGitHub related information:"
          println "GITHUB_EVENT_NAME = ${env.GITHUB_EVENT_NAME ?: 'Not set'}"
          println "GITHUB_REF = ${env.GITHUB_REF ?: 'Not set'}"
          println "GITHUB_SHA = ${env.GITHUB_SHA ?: 'Not set'}"
          println "GITHUB_REPOSITORY = ${env.GITHUB_REPOSITORY ?: 'Not set'}"
          
          // Print build parameters if any
          println "\nBuild parameters:"
          if (params) {
            params.each { key, value ->
              println "${key} = ${value}"
            }
          } else {
            println "No build parameters set"
          }
          
          // For push events, we only need the branch name
          def branchName = env.BRANCH_NAME
          def gitUrl = env.GIT_URL
          
          // Different checkout configuration based on whether it's a PR or push
          if (env.CHANGE_ID) {
            // PR event
            checkout([
              $class: 'GitSCM',
              branches: [[name: env.CHANGE_BRANCH]],
              extensions: [
                [$class: 'PreBuildMerge',
                 options: [
                   fastForwardMode: 'FF',
                   mergeTarget: env.CHANGE_TARGET,
                   mergeStrategy: 'default',
                   remote: env.CHANGE_FORK ?: gitUrl,
                   refspec: "+refs/heads/${env.CHANGE_BRANCH}:refs/remotes/origin/${env.CHANGE_BRANCH}"
                 ]
                ]
              ],
              userRemoteConfigs: [[url: gitUrl]]
            ])
          } else {
            // Push event
            checkout([
              $class: 'GitSCM',
              branches: [[name: branchName]],
              userRemoteConfigs: [[url: gitUrl]]
            ])
          }
        }
      }
    }
    stage('Unit Test') {
      steps {
        sh 'mvn clean test'
      }
    }
  }
}