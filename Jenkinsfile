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
						docker build -t pavangunda66/hellovadi .
					'''
				}
			}
		}

		stage('Push Image To Dockerhub') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
						docker push pavangunda66/hellovadi
					'''
				}
			}
		}

		stage('set label') {
			steps {
				withAWS(region:'us-east-1', credentials:'pagu18') {
					sh '''
						kubectl label node --all "disk=hdd"
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

		stage('set label') {
			steps {
				withAWS(region:'us-east-1', credentials:'pagu18') {
					sh '''
						kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.6/components.yaml
					'''
				}
			}
		}

		stage('autoscaler') {
			steps {
				withAWS(region:'us-east-1', credentials:'pagu18') {
					sh '''
						kubectl apply -f hpablue.yaml
					'''
				}
			}
		}

		stage('autoscaler') {
			steps {
				withAWS(region:'us-east-1', credentials:'pagu18') {
					sh '''
						kubectl apply -f hpagreen.yaml
					'''
				}
			}
		}

		stage('add pod disruption budget') {
			steps {
				withAWS(region:'us-east-1', credentials:'pagu18') {
					sh ''
						kubectl apply -f pdbblue.yaml
					'''
				}
			}
		}

		stage('add pod disruption budget') {
			steps {
				withAWS(region:'us-east-1', credentials:'pagu18') {
					sh ''
						kubectl apply -f pdbgreen.yaml
					'''
				}
			}
		}

		stage('Create the service in the cluster, redirect to blue') {
			steps {
				withAWS(region:'us-east-1', credentials:'pagu18') {
					sh '''
						kubectl apply -f blueservice.yaml
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
						kubectl apply -f greenservice.yaml
					'''
				}
			}
		}

	}
}
