@Library('sharedlibrary')_

pipeline {
    agent any

    environment {
        ecrRegistry   = "971002455839.dkr.ecr.ap-south-1.amazonaws.com"
        frontendImage = "${ecrRegistry}/frontend"
        gitRepoURL    = "https://github.com/prashanth0996/final_project-2026.git"

        // Normalize branch name or default to "main"
        BRANCH_NAME   = "${env.BRANCH_NAME?.trim() ?: 'main'}".replaceAll("/", "-")
        gitCommit     = "${env.GIT_COMMIT.take(7)}"
        dockerTag     = "${BRANCH_NAME}-${gitCommit}-${env.BUILD_NUMBER}"
    }

    stages {
        stage('Git Checkout') {
            steps {
                gitCheckout("${gitRepoURL}", "${BRANCH_NAME}", "githubCred")
            }
        }

        stage('Docker Build Frontend') {
            steps {
                dir('frontend') {
                    dockerImageBuild("${frontendImage}", "${dockerTag}")
                }
            }
        }

        stage('Docker Push Frontend') {
            steps {
                dockerECRImagePush("${frontendImage}", "${dockerTag}", "ap-south-1")
            }
        }
    }
}
