@Library('sharedlibrary')_

pipeline {
    agent any

    environment {
        ecrRegistry   = "971002455839.dkr.ecr.ap-south-1.amazonaws.com"
        frontendImage = "${ecrRegistry}/frontend"
        gitCommit     = "${GIT_COMMIT[0..6]}"
        BRANCH_NAME = sh(script: 'echo $BRANCH_NAME | sed "s#/#-#"', returnStdout: true
        dockerTag     = "${BRANCH_NAME}-${gitCommit}-${env.BUILD_NUMBER}"
        gitRepoURL    = "https://github.com/prashanth0996/final_project-2026.git"
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
