@Library('sharedlibrary')_

pipeline {
    agent any 
    
    environment {
        ecrRegistry = "971002455839.dkr.ecr.ap-south-1.amazonaws.com"
        frontendImage = "${ecrRegistry}/frontend"
        BRANCH_NAME = sh(script: 'echo $BRANCH_NAME | sed "s#/#-#"', returnStdout: true).trim()
        gitCommit = "${GIT_COMMIT[0..6]}"
        dockerTag = "${BRANCH_NAME}-${gitCommit}-${env.BUILD_NUMBER}"
        gitRepoURL = "https://github.com/prashanth0996/final_project-2026.git"
    }
    stages {
        stage('Git Checkout') {
            steps {
                gitCheckout("$gitRepoURL", "$BRANCH_NAME", "githubCred")
            }
        }

        stage('Docker Build') {
            stages {
                stage('Build Frontend') {
                    steps {
                        dir('frontend') {
                            dockerImageBuild('$frontendImage', '$dockerTag')
                        }
                    }
                }
            }
        }

        // stage('Security Scans') {
        //     parallel {
        //         stage('Snyk Scan Backend') {
        //             steps {
        //                 snykImageScan('$backendImage', '$dockerTag', 'snykCred', '$snykOrg')
        //             }
        //         }
        //         stage('Snyk Scan Frontend') {
        //             steps {
        //                 snykImageScan('$frontendImage', '$dockerTag', 'snykCred', '$snykOrg')
        //             }
        //         }
        //         stage('Snyk Scan Model') {
        //             steps {
        //                 snykImageScan('$modelImage', '$dockerTag', 'snykCred', '$snykOrg')
        //             }
        //         }
        //     }
        // }

        // stage('Trivy Scans') {
        //     parallel {
        //         stage('Trivy Scan Backend') {
        //             steps {
        //                 sh "trivy image -f json -o backend-results-${BUILD_NUMBER}.json ${backendImage}:${dockerTag}"
        //             }
        //         }
        //         stage('Trivy Scan Frontend') {
        //             steps {
        //                 sh "trivy image -f json -o frontend-results-${BUILD_NUMBER}.json ${frontendImage}:${dockerTag}"
        //             }
        //         }
        //         stage('Trivy Scan Model') {
        //             steps {
        //                 sh "trivy image -f json -o model-results-${BUILD_NUMBER}.json ${modelImage}:${dockerTag}"
        //             }
        //         }
        //     }
        // }

        stage('Docker Push') {
            stages {
                stage('Push Frontend') {
                    steps {
                        dockerECRImagePush('$frontendImage', '$dockerTag', 'ap-south-1')
                    }
                }
            }
        }
    }
}
