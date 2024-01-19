pipeline {
    agent any

    options {
        timeout(time: 1, unit: 'HOURS') // Set a timeout for the entire pipeline
	buildDiscarder(logRotator(numToKeepStr: '2', daysToKeepStr: '0'))
	disableConcurrentBuilds()
        timestamps() // Add timestamps to the console output
    }

    parameters {
        booleanParam(name: 'EXPRESS_MODE', defaultValue: false, description: 'Enable express mode')
    }

    stages {
        stage('Build Maven') {
            steps {
                git branch: 'main', url: 'https://github.com/kumarcc1/devops-automation.git'
                script {
                    try {
                        if (params.EXPRESS_MODE) {
                            sh 'mvn clean install -DskipTests' // Skip tests in express mode
                        } else {
                            sh 'mvn clean install'
                        }
                    } catch (Exception e) {
                        echo "Build failed. Exception: ${e}"
                        currentBuild.result = 'FAILURE'
                        error("Build failed.")
                    }
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    def dockerImageTag = params.EXPRESS_MODE ? 'express' : env.BUILD_NUMBER
                    echo "Docker Image Tag: ${dockerImageTag}" // Add this line for debugging
                    sh "docker build -t kumarcc1/k8s-jenkin-integration:${dockerImageTag} ."
                }
            }
        }

        stage('Push Image to Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerhub', usernameVariable: 'kumarcc1')]) {
                        sh 'docker login -u ${kumarcc1} -p ${dockerhub}'
                        sh 'docker push kumarcc1/k8s-jenkin-integration:${dockerImageTag}'
                    }
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                script {
                    kubernetesDeploy(configs: 'deploymentservice.yaml', kubeconfigId: 'k8scluster', env: [DEPLOY_ENV: params.DEPLOY_ENV])
                }
            }
        }
    }

    post {
        always {
            script {
                // Attach the complete log to the email
                emailext attachLog: true,
                        body: "Build ${currentBuild.displayName} has finished. View console output at ${env.BUILD_URL}",
                        subject: "Build ${currentBuild.displayName} - ${currentBuild.currentResult}",
                        to: 'kumarcc1@gmail.com'
            }
        }
    }
}
