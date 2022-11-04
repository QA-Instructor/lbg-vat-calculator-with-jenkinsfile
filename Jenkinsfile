pipeline{
    environment {
        registry = "victorialloyd/vatcalc"
        registryCredentials = "dockerhub_id"
        dockerImage = ""
    }
    agent any
        stages {
            stage ('Build Docker Image'){
                steps{
                    script {
                        dockerImage = docker.build(registry)
                    }
                }
            }

            stage ("Push to Docker Hub"){
                steps {
                    script {
                        docker.withRegistry('', registryCredentials) {
                            dockerImage.push("${env.BUILD_NUMBER}")
                            dockerImage.push("latest")
                        }
                    }
                }
            }

            stage ("Deploy to target GCP Instance"){
		steps {
			sshPublisher(publishers: [sshPublisherDesc(configName: 'TargetDockerServer', 
			transfers: [sshTransfer(cleanRemote: false, excludes: '', 
			execCommand: 'docker run --name DeployedVATCalc -d -p 80:80 victorialloyd/vatcalc', 
                        execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, 
                        patternSeparator: '[, ]+', remoteDirectory: '', 
			remoteDirectorySDF: false, removePrefix: '', 
			sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, 
			verbose: false)])			
		}
	    }

            stage ("Clean up"){
                steps {
                    script {
                        sh 'docker image prune --all --force --filter "until=164h"'
                           }
                }
            }
        }
}
