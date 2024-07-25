pipeline {
  agent any
  tools {
    nodejs 'node'
  }
  environment {
    GITHUB_CREDENTIALS = credentials('GITHUB_CREDENTIALS') 
  }
  stages {
    stage('Clone repository') {
      steps {
        cleanWs()
        checkout scm
      }
    }
    stage('Validate Conventional Commits') {
      steps {
        withEnv(["GITHUB_TOKEN=${GITHUB_CREDENTIALS_PSW}"]) {
          sh '''
          echo "Validate commit messages"

          npm i -D @semantic-release/commit-analyzer @semantic-release/exec @semantic-release/git @semantic-release/github
          npx semantic-release --dry-run
          '''
        }
      }
    }
    stage('Checks for helm') {
      steps {
        withEnv(["GITHUB_TOKEN=${GITHUB_CREDENTIALS_PSW}"]) {
          sh '''
          echo "Checks for helm"
          helm lint .
          '''
        }
      }
    }
    stage('Change chart version and release') {
      when {
        branch 'main'
      }
      steps {
        withEnv(["GITHUB_TOKEN=${GITHUB_CREDENTIALS_PSW}"]) {
          sh '''
          npx semantic-release --dry-run > semantic-release-output.txt
          newVersion=$(grep 'next release version' semantic-release-output.txt | awk '{print $NF}')
          echo "Change chart version"
          echo "New version: ${newVersion}"
          sed -i "s/^version: .*/version: ${newVersion}/" Chart.yaml
          git add Chart.yaml
          npx semantic-release
          '''
        }
      }
    }
  }
  post {
    always {
      cleanWs()
    }
    success {
      echo 'Pipeline completed successfully!'
    }
    failure {
      echo 'Pipeline failed!'
    }
  }
}
