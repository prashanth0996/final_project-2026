@Library('sharedlibrary')_

pipeline {
    agent any

    environment {
        ecrRegistry   = "971002455839.dkr.ecr.ap-south-1.amazonaws.com"
        frontendImage = "${ecrRegistry}/frontend"
        gitRepoURL    = "https://github.com/prashanth0996/final_project-2026.git"
    }

    stages {
        stage('Git Checkout') {
            steps {
                script {
                    // Normalize branch name or default to "main"
                    env.BRANCH_NAME = env.BRANCH_NAME?.trim() ?: "main"
                    env.BRANCH_NAME = env.BRANCH_NAME.replaceAll("/", "-")

                    // Short commit hash
                    env.gitCommit   = env.GIT_COMMIT.take(7)

                    // Docker tag
                    env.dockerTag   = "${env.BRANCH_NAME}-${env.gitCommit}-${env.BUILD_NUMBER}"

                    echo "DEBUG: BRANCH_NAME=${env.BRANCH_NAME}"
                    echo "DEBUG: gitCommit=${env.gitCommit}"
                    echo "DEBUG: dockerTag=${env.dockerTag}"

                    // Checkout
                    gitCheckout("${env.gitRepoURL}", "${env.BRANCH_NAME}", "githubCred")
                }
            }
        }

           stage("Sonarqube Analysis ") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    dir('frontend') { 
                        sh '''
                        /opt/sonar/bin/sonar-scanner \
                        -Dsonar.projectName="movie-analyzer" \
                        -Dsonar.projectKey="movie-analyzer"
                        '''
                    }
                }
            }
        }

        stage('Docker Build Frontend') {
            steps {
                dir('frontend') {
                    dockerImageBuild("${env.frontendImage}", "${env.dockerTag}")
                }
            }
        }

        stage('Docker Push Frontend') {
            steps {
                dockerECRImagePush("${env.frontendImage}", "${env.dockerTag}", "ap-south-1")
            }
        }
    }
}
