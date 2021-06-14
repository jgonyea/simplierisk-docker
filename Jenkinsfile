pipeline {
	agent {
		label 'buildtestmed'
	}
	stages {
		stage ('Initializing Variables') {
			steps {
				script {
					current_version = getOfficialVersion("updates")
					med_id = getEC2Metadata("instance-id")
				}
			}
		}
		stage ('Build') {
			parallel {
				stage ('Build SimpleRisk Ubuntu 18.04') {
					steps {
						sh """
							sudo docker build -t simplerisk/simplerisk -f simplerisk/bionic/Dockerfile simplerisk/
							sudo docker build -t simplerisk/simplerisk:$current_version -f simplerisk/bionic/Dockerfile simplerisk/
							sudo docker build -t simplerisk/simplerisk:$current_version-bionic -f simplerisk/bionic/Dockerfile simplerisk/
						"""
					}
				}
				stage ('Build SimpleRisk Ubuntu 20.04') {
					steps {
						sh """
							sudo docker build -t simplerisk/simplerisk:$current_version-focal -f simplerisk/focal/Dockerfile simplerisk/
						"""
					}
				}
				stage ('Build SimpleRisk Minimal PHP 7.2') {
					steps {
						sh """
							sudo docker build -t simplerisk/simplerisk-minimal -f simplerisk-minimal/php7.2/Dockerfile simplerisk-minimal/
							sudo docker build -t simplerisk/simplerisk-minimal:$current_version -f simplerisk-minimal/php7.2/Dockerfile simplerisk-minimal/
							sudo docker build -t simplerisk/simplerisk-minimal:$current_version-php72 -f simplerisk-minimal/php7.2/Dockerfile simplerisk-minimal/
						"""
					}
				}
				stage ('Build SimpleRisk Minimal PHP 7.4') {
					steps {
						sh """
							sudo docker build -t simplerisk/simplerisk-minimal:$current_version-php74 -f simplerisk-minimal/php7.4/Dockerfile simplerisk-minimal/
						"""
					}
				}
			}
			post {
				failure {
					node("jenkins") {
						terminateInstance("${med_id}")
					}
					error("Stopping full build")
				}
			}
		}
		stage ('Log into Docker Hub') {
			steps {
				withCredentials([usernamePassword(credentialsId: 'cb153fa6-2299-4bdb-9ef0-9c3e6382c87a', passwordVariable: 'docker_pass', usernameVariable: 'docker_user')]) {
					sh '''
						set +x
						echo $docker_pass >> /tmp/password.txt
						cat /tmp/password.txt | sudo docker login --username $docker_user --password-stdin
						rm /tmp/password.txt
					'''
				}
			}
		}
	}
}

def getEC2Metadata(String attribute){
	return sh(script: "echo \$(TOKEN=`curl -s -X PUT \"http://169.254.169.254/latest/api/token\" -H \"X-aws-ec2-metadata-token-ttl-seconds: 21600\"` && curl -s -H \"X-aws-ec2-metadata-token: \$TOKEN\" http://169.254.169.254/latest/meta-data/$attribute)", returnStdout: true).trim()
}

def getOfficialVersion(String updatesDomain="updates-test") {
	return sh(script: "echo \$(curl -sL https://$updatesDomain\\.simplerisk.com/Current_Version.xml | grep -oP '<appversion>(.*)</appversion>' | cut -d '>' -f 2 | cut -d '<' -f 1)", returnStdout: true).trim()
}

void terminateInstance(String instanceId, String region="us-east-1") {
	sh "aws ec2 terminate-instances --instance-ids $instanceId --region $region"
}
