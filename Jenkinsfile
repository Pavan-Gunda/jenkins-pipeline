pipeline {
	agent any
	stages {
	  stage('Lint python script') {
		  steps {
			  sh 'pylint *.py'
		  }
	  }
		stage('Build Docker Image') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker build -t pavangunda66/calc .
					'''
				}
			}
    }

		stage('Push Image To Dockerhub') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
						docker push pavangunda66/calc
					'''
				}
			}
		}

		stage('set label') {
			steps {
				withAWS(region:'us-east-1', credentials:'pagu18') {
					sh '''
						kubectl label node --all "disk=ssd" --overwrite=true
					'''
				}
			}
		}

		stage('deploy blue container') {
			steps {
				withAWS(region:'us-east-1', credentials:'pagu18') {
					sh '''
						kubectl apply -f bluecontroller.yml
					'''
				}
			}
		}

		stage('deploy green container') {
			steps {
				withAWS(region:'us-east-1', credentials:'pagu18') {
					sh '''
						kubectl apply -f greencontroller.yml
					'''
				}
			}
		}

		stage('install metrics server') {
			steps {
				withAWS(region:'us-east-1', credentials:'pagu18') {
					sh '''
						kubectl apply -f metricserver.yaml
					'''
				}
			}
		}

		stage('assign hpa blue') {
			steps {
				withAWS(region:'us-east-1', credentials:'pagu18') {
					sh '''
						kubectl apply -f hpablue.yml
					'''
				}
			}
		}

		stage('assign hpa green') {
			steps {
				withAWS(region:'us-east-1', credentials:'pagu18') {
					sh '''
						kubectl apply -f hpagreen.yml
					'''
				}
			}
		}

		stage('add pod disruption budget') {
			steps {
				withAWS(region:'us-east-1', credentials:'pagu18') {
					sh '''
						kubectl apply -f pdbblue.yml
					'''
				}
			}
		}

		stage('add pod disruption budget green') {
			steps {
				withAWS(region:'us-east-1', credentials:'pagu18') {
					sh '''
						kubectl apply -f pdbgreen.yml
					'''
				}
			}
		}

		stage('Create the service in the cluster, redirect to blue') {
			steps {
				withAWS(region:'us-east-1', credentials:'pagu18') {
					sh '''
						kubectl apply -f blueservice.yml
					'''
				}
			}
		}

		stage('Wait user approve') {
            steps {
                input "Ready to redirect traffic to green?"
            }
        }

		stage('Create the service in the cluster, redirect to green') {
			steps {
				withAWS(region:'us-east-1', credentials:'pagu18') {
					sh '''
						kubectl apply -f greenservice.yml
					'''
				}
			}
		}

	}
}
