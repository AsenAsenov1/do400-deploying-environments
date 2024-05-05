pipeline {
    agent {
        node {
            label 'maven'
        }
    }
    stages {
        stage('Tests') {
            steps {
                sh './mvnw clean test'
            }
        }
        stage('Package') {
            steps {
                sh '''
                    ./mvnw package -DskipTests \
                    -Dquarkus.package.type=uber-jar
                '''
                archiveArtifacts 'target/*.jar'
            }
        }
        stage('Build Image') {
            environment {
                QUAY_USR = credentials('QUAY_USER')
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'QUAY_USER', usernameVariable: 'QUAY_USERNAME', passwordVariable: 'QUAY_PASSWORD')]) {
                    sh '''
                        ./mvnw quarkus:add-extension \
                        -Dextensions="kubernetes,container-image-jib"
                    '''
                    sh '''
                        ./mvnw package -DskipTests \
                        -Dquarkus.jib.base-jvm-image=quay.io/redhattraining/do400-java-alpine-openjdk11-jre:latest \
                        -Dquarkus.container-image.build=true \
                        -Dquarkus.container-image.registry=quay.io \
                        -Dquarkus.container-image.group=$QUAY_USERNAME \
                        -Dquarkus.container-image.name=do400-deploying-environments \
                        -Dquarkus.container-image.username=$QUAY_USERNAME \
                        -Dquarkus.container-image.password="$QUAY_PASSWORD" \
                        -Dquarkus.container-image.push=true
                    '''
                }
            }
        }
    }
}

