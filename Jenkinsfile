pipeline {
    environment {
        dbMigrationImageName = "marlapativ/db-migration"
        registryCredential = 'dockerhub'
    }
    agent any
    stages {
        stage('Setup Docker') {
            steps {
                sh '''
                    if [ -n "$(docker buildx ls | grep multiarch)" ]; then
                        docker buildx use multiarch
                    else
                        docker buildx create --name=multiarch --driver=docker-container --use --bootstrap 
                    fi
                '''
                
                script {
                    withCredentials([usernamePassword(credentialsId: registryCredential, passwordVariable: 'password', usernameVariable: 'username')]) {
                        sh('docker login -u $username -p $password')
                    }
                }
            }
        }
        stage('Generate Version') {
            tools {
                nodejs "nodejs"
            }
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-app', passwordVariable: 'GITHUB_TOKEN', usernameVariable: 'GITHUB_USERNAME')]) {
                        sh '''
                            export GITHUB_ACTION=true
                            npx semantic-release \
                                -p @semantic-release/commit-analyzer \
                                -p @semantic-release/release-notes-generator \
                                -p @semantic-release/github
                        '''
                    }
                }
            }
        }
        stage('Build and Push Flyway database migration image') {
            steps {
                sh '''
                    export IMAGE_TAG=$(git describe --tags --abbrev=0)

                    docker buildx build \
                    --platform linux/amd64,linux/arm64 \
                    --builder multiarch \
                    -t $dbMigrationImageName:latest \
                    -t $dbMigrationImageName:$IMAGE_TAG \
                    --push \
                    .
                '''
            }
        }
    }
}
