pipeline {
  agent any
  
  stages {
    stage('Checkout & Prepare') {
      steps {
        script {
          // For push events, we only need the branch name
          def branchName = env.BRANCH_NAME ?: 'master'
          def gitUrl = env.GIT_URL ?: 'git@github.com:xuzhang0636/PR_Hook_Test.git'
          
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
        sh 'mvn clean test'  // 根据项目替换为实际测试命令（如 mvn test、pytest 等）
      }
      post {
        success {
          // Update GitHub commit status
          updateGitHubCommitStatus(
            state: 'SUCCESS',
            context: 'jenkins/unit-test',
            description: 'Unit tests passed successfully'
          )
          
          // Only add PR comment if this is a PR
          script {
            if (env.CHANGE_ID) {
              def comment = """
                ✅ Unit tests passed successfully!
                This PR is ready to be merged.
              """.stripIndent()
              
              githubNotify(
                issueNumber: env.CHANGE_ID,
                comment: comment
              )
            }
          }
        }
        failure {
          // Update GitHub commit status
          updateGitHubCommitStatus(
            state: 'FAILURE',
            context: 'jenkins/unit-test',
            description: 'Unit tests failed'
          )
          
          // Only add PR comment if this is a PR
          script {
            if (env.CHANGE_ID) {
              def comment = """
                ❌ Unit tests failed!
                Please fix the failing tests before merging.
              """.stripIndent()
              
              githubNotify(
                issueNumber: env.CHANGE_ID,
                comment: comment
              )
            }
          }
        }
      }
    }
  }
  
  post {
    always {
      // Clean workspace after build
      cleanWs()
    }
  }
}