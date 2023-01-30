pipeline {

    agent any
/*
	tools {
        maven "maven3"
    }
*/
    environment {
	registry = "earlyspring/test"
        registryCredential = 'dockerhub'
        DOCKERHUB_CREDENTIALS=credentials('dockerhub')
    }

    stages{

        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }

        stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }

        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }


        stage('Building image') {
            steps{
              sh 'docker build -t earlyspring/test:V$BUILD_NUMBER .'
	      sh 'docker build -t earlyspring/test:latest .'
            }
        }
	    
	stage('Docker Login'){
		steps{
		  sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
		}
	    }
        
        stage('Deploy Image') {
          steps{
            sh 'docker push earlyspring/test:V$BUILD_NUMBER'
            sh 'docker push earlyspring/test:latest'
          }
        }


        stage('Remove Unused docker image') {
          steps{
            sh "docker rmi $registry:V$BUILD_NUMBER" 
	    sh "docker rmi $registry:latest" 
          }
	}

	 stage(kubernetes kops ) {
	  steps{
            sh 'kubectl apply -f . --namespace staging'
        }
	   }

        stage('Kubernetes helm') {
            steps {
                    sh "helm upgrade --install --force vproifle-stack helm/vprofilecharts --set appimage=$registry:V$BUILD_NUMBER --namespace prod"
            }
        }

    }


}
