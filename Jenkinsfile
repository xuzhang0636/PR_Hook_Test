pipeline {
  agent any
  
  stages {
    stage('Checkout & Prepare') {
      steps {
        script {
          // Ensure we have a valid branch name
          def branchName = env.CHANGE_BRANCH ?: env.BRANCH_NAME ?: 'main'
          def targetBranch = env.CHANGE_TARGET ?: env.BRANCH_NAME ?: 'main'
          def gitUrl = env.GIT_URL ?: 'git@github.com:xuzhang0636/PR_Hook_Test.git'
          
          checkout([
            $class: 'GitSCM',
            branches: [[name: branchName]],
            extensions: [
              [$class: 'PreBuildMerge',
               options: [
                 fastForwardMode: 'FF',
                 mergeTarget: targetBranch,
                 mergeStrategy: 'default',
                 remote: env.CHANGE_FORK ?: gitUrl,
                 refspec: "+refs/heads/${branchName}:refs/remotes/origin/${branchName}"
               ]
              ]
            ],
            userRemoteConfigs: [[url: gitUrl]]
          ])
        }
      }
    }
    stage('Unit Test') {
      steps {
        sh 'mvn clean test'  // 根据项目替换为实际测试命令（如 mvn test、pytest 等）
      }
      post {
        success {
          // Update GitHub PR status
          updateGitHubCommitStatus(
            state: 'SUCCESS',
            context: 'jenkins/unit-test',
            description: 'Unit tests passed successfully'
          )
          
          // If this is a PR, add a comment
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
          // Update GitHub PR status
          updateGitHubCommitStatus(
            state: 'FAILURE',
            context: 'jenkins/unit-test',
            description: 'Unit tests failed'
          )
          
          // If this is a PR, add a comment
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