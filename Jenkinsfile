/*pipeline {
    agent { label 'jappbuildserver1' }	

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "maven_3.6.3"
    }

	environment {	
		DOCKERHUB_CREDENTIALS=credentials('dockerloginid')
	} 
    
    stages {
        stage('SCM Checkout') {
            steps {
                // Get some code from a GitHub repository
                git 'https://github.com/LoksaiETA/BankingApp.git'
                //git 'https://github.com/LoksaiETA/Java-mvn-app2.git'
            }
		}
        stage('Maven Build') {
            steps {
                // Run Maven on a Unix agent.
                sh "mvn -Dmaven.test.failure.ignore=true clean package"
            }
		}
       stage("Docker build"){
            steps {
				sh 'docker version'
				sh "docker build -t loksaieta/bankapp-eta-app:${BUILD_NUMBER} ."
				sh 'docker image list'
				sh "docker tag loksaieta/bankapp-eta-app:${BUILD_NUMBER} loksaieta/bankapp-eta-app:latest"
            }
        }
		stage('Login2DockerHub') {

			steps {
				sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
			}
		}
		stage('Push2DockerHub') {

			steps {
				sh "docker push loksaieta/bankapp-eta-app:latest"
			}
		}
        stage('Deploy to Kubernetes Dev Environment') {
            steps {
		script {
		sshPublisher(publishers: [sshPublisherDesc(configName: 'Kubernetes', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'kubectl apply -f kubernetesdeploy.yaml', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '.', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '*.yaml')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
		       }
            }
    	}
    }
}
*/


pipeline {
    agent any
    
    tools{
        jdk'jkd8'
        jdk'jdk17'
        maven'maven3'
    }

    stages {
        stage('Git Cloning') {
            steps {
                git 'https://github.com/venkatesh9691/BankingApp.git'
            }
        }
        stage("Maven compiling"){
            steps{
                sh'mvn compile'
            }
        }
        stage("Maven Testing"){
            steps{
                sh'mvn test'
            }
        }
        stage("OWASP Dependency Checking"){
            steps{
                dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }
        }
        stage("file system scanning"){
            steps{
	            sh'trivy fs --format table -o trivy-fs-report.html .'
            }
        }
        stage("Maven Build"){
            steps{
                sh'mvn clean package'
            }
        }
        stage("Nexus Artifact"){
            steps{
                nexusArtifactUploader artifacts: [
                    [
                        artifactId: 'spring-boot-starter-parent',
                        classifier: '',
                        file: 'target/banking-0.0.1-SNAPSHOT.jar',
                        type: 'jar'
                    ]
                ],
                credentialsId: 'nexus',
                groupId: 'com.project.staragile',
                nexusUrl: '16.16.74.54:8081',
                nexusVersion: 'nexus3',
                protocol: 'http',
                repository: 'venkat',
                version: '0.0.1-SNAPSHOT'
            }
        }
        stage('Build Docker Image'){
            steps{
                sh'docker stop s4'
                sh 'docker rm s4'
                sh 'docker build -t venkatesh9691/venkatesh-projects-new .'
                sh 'docker build -t tomcat:${BUILD_NUMBER} .'
                sh 'docker run -itd --name s4 -p 3800:8080 tomcat:${BUILD_NUMBER}'
            }
        }
        stage("Docker image scaning"){
            steps{
                sh'trivy image --format table -o trivy-fs-report.html venkatesh9691/venkatesh-projects-new'
            }
        }
        stage("push to Docker Hub"){
            steps{
                withCredentials([usernameColonPassword(credentialsId: 'DOCKER_HUB_CREDENTIALS', variable: 'DOCKER_HUB_CREDENTIALS')]) {
                    sh'docker login -u ${DOCKER_HUB_CREDENTIALS} -p ${DOCKER_HUB_CREDENTIALS}'
                }
                sh 'docker push venkatesh9691/venkatesh-projects-new'
            }
        }
    }
}
