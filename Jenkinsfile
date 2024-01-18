pipeline {
    agent any
    tools{
        maven 'maven_3_5_0'
    }
    options{

        buildDiscarder(logRotator(numToKeepStr: '2', daysToKeepStr: '1'))
        disableConcurrentBuilds()
		  timeout(time:1, unit:'hours')
        timestamps()
        
    }
        
    stages{
        stage('Build Maven'){
            steps{
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'git branch: 'main', url: 'https://github.com/kumarcc1/devops-automation.git']])
                sh 'mvn clean install'
            }
        }
        stage('Build docker image'){
            steps{
                script{
                    sh "docker build -t kumarcc1/devops-integration:${BUILD_NUMBER} ."

                }
            }
        }
        stage('Push image to Hub'){
            steps{
                script{
                   withCredentials([usernamePassword(credentialsId: 'dockerhublogin', passwordVariable: 'dockerhub', usernameVariable: 'kumarcc1')]) {
                       sh 'docker login -u ${kumarcc1} -p ${dockerhub}'
                       sh 'docker push kumarcc1/devops-integration'
                   }
                }
            }
        }
        stage('Deploy to k8s'){
            steps{
                script{
                    kubernetesDeploy (configs: 'deploymentservice.yaml',kubeconfigId: 'k8scluster')
                }
            }
        }
    }
}
