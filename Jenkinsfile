pipeline {
  agent any
  
  stages {
    stage('Checkout & Prepare') {
      steps {
        script {
          // Print GitHub webhook payload information
          println "\nGitHub Webhook Payload Information:"
          // 方法1：使用 GitHub 插件提供的变量
          println "CHANGE_ID = ${env.CHANGE_ID ?: 'Not set'}"  // PR number
          println "CHANGE_TITLE = ${env.CHANGE_TITLE ?: 'Not set'}"  // PR title
          println "CHANGE_AUTHOR = ${env.CHANGE_AUTHOR ?: 'Not set'}"  // PR author
          println "CHANGE_BRANCH = ${env.CHANGE_BRANCH ?: 'Not set'}"  // PR branch
          println "CHANGE_TARGET = ${env.CHANGE_TARGET ?: 'Not set'}"  // PR target branch
          println "CHANGE_URL = ${env.CHANGE_URL ?: 'Not set'}"  // PR URL
          
          // 方法2：如果是 push 事件
          println "GIT_BRANCH = ${env.GIT_BRANCH ?: 'Not set'}"
          println "GIT_COMMIT = ${env.GIT_COMMIT ?: 'Not set'}"
          println "GIT_URL = ${env.GIT_URL ?: 'Not set'}"
          
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