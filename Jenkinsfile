pipeline {
	agent any
	stages {

		stage('Lint HTML') {
			steps {
				sh 'tidy -q -e *.html'
			}
		}
		
		stage('Build Docker Image') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker build -t vivekgouda/capstone .
					'''
				}
			}
		}

		stage('Push Image To Dockerhub') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
						docker push vivekgouda/capstone
					'''
				}
			}
		}

		stage('Set current kubectl context') {
			steps {
				withAWS(region:'us-west-2', credentials:'ecr_credentials') {
					sh '''
						kubectl config use-context arn:aws:eks:us-west-2:078563032339:cluster/capstonecluster
					'''
				}
			}
		}

		stage('Deploy blue container') {
			steps {
				withAWS(region:'us-west-2', credentials:'ecr_credentials') {
					sh '''
						kubectl apply -f ./blue-controller.json
					'''
				}
			}
		}

		stage('Deploy green container') {
			steps {
				withAWS(region:'us-west-2', credentials:'ecr_credentials') {
					sh '''
						kubectl apply -f ./green-controller.json
					'''
				}
			}
		}

		stage('Create the service in the cluster, redirect to blue') {
			steps {
				withAWS(region:'us-west-2', credentials:'ecr_credentials') {
					sh '''
						kubectl apply -f ./blue-service.json
					'''
				}
			}
		}

		stage('Create the service in the cluster, redirect to green') {
			steps {
				withAWS(region:'us-west-2', credentials:'ecr_credentials') {
					sh '''
						kubectl apply -f ./green-service.json
					'''
				}
			}
		}

	}
}
