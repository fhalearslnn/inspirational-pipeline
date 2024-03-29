pipeline{
    agent any
    tools {
        jdk 'Java21'
        maven 'Maven3'
    }
    environment {
        APP_NAME = "completepipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "fhalearslan"
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")

    }
    stages{
        stage("Cleanup Workspace"){
            steps {
                cleanWs()
            }

        }
    
        stage("Checkout from SCM"){
            steps {
                git branch: 'main', url: 'https://github.com/fhalearslnn/inspirational-pipeline.git'
            }

        }

        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }

        }

        stage("Test Application"){
            steps {
                sh "mvn test"
            }

        }
        stage("Code Coverage with JaCoCo") {
            steps {
                sh "mvn jacoco:prepare-agent test jacoco:report"
                archiveArtifacts 'target/site/jacoco/index.html'
            }
        }
        
        // stage("Sonarqube Analysis") {
        //     steps {
        //         script {
        //             withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
        //                 sh "mvn sonar:sonar"
        //             }
        //         }
        //     }

        // }

        // stage("Quality Gate") {
        //     steps {
        //         script {
        //             waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
        //         }
        //     }

        // }

        // stage("Build & Push Docker Image") {
        //     steps {
        //         script {
        //             // Retrieve DockerHub token from Jenkins credentials
        //             withCredentials([string(credentialsId: 'jenkins-token', variable: 'DOCKER_TOKEN')]) {
                        
        //                 def dockerRegistryUrl = 'https://hub.docker.com/'

        //                 // Build the Docker image
        //                 docker.withRegistry(dockerRegistryUrl, DOCKER_TOKEN) {
        //                     docker_image = docker.build "${IMAGE_NAME}"
        //                 }

        //                 // Push the Docker image to DockerHub
        //                 docker.withRegistry(dockerRegistryUrl, DOCKER_TOKEN) {
        //                     docker_image.push("${IMAGE_TAG}")
        //                     docker_image.push('latest')
        //                 }
        //             }
        //         }
        //     }
        // }
        stage('Build App Docker Image') {
            steps {
                echo 'Building App Image'                
                sh 'docker build --force-rm -t "$IMAGE_NAME" -f ./Dockerfile .'
                sh 'docker image ls'
            }
       }

        stage('Push Image to Dockerhub Repo') {
            steps {
                echo 'Pushing App Image to DockerHub Repo'
                withCredentials([string(credentialsId: 'jenkins-token', variable: 'DOCKER_TOKEN')]) {
                sh 'docker login -u $DOCKERHUB_USER -p $DOCKER_TOKEN'
                sh 'docker push "$IMAGE_NAME"'
                
            }
          }
        }

        stage("Trivy Scan") {
            steps {
                script {
		   sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${IMAGE_NAME} --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
                }
            }

        }

        stage ('Cleanup Artifacts') {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}"
                    sh "docker rmi ${IMAGE_NAME}"
                }
            }
        }

        stage("Trigger CD Pipeline") {
            steps {
                script {
                    def localJenkinsUrl = 'http://localhost:8080'
                    def localJenkinsJobPath = 'job/gitops-complete-pipeline'
                    def localJenkinsToken = 'gitops-token'
                    
                    sh "curl -v -k --user yasin_devops:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' '${localJenkinsUrl}/${localJenkinsJobPath}/buildWithParameters?token=${localJenkinsToken}'"
                }
            }
        }


        // stage("Trigger CD Pipeline") {
        //     steps {
        //         script {
        //             sh "curl -v -k --user haley:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'https://jenkins.dev.dman.cloud/job/gitops-complete-pipeline/buildWithParameters?token=gitops-token'"
        //         }
        //     }

        // }

    }

    // post {
    //     failure {
    //         emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
    //                 subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Failed", 
    //                 mimeType: 'text/html',to: "dmistry@yourhostdirect.com"
    //         }
    //      success {
    //            emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
    //                 subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Successful", 
    //                 mimeType: 'text/html',to: "dmistry@yourhostdirect.com"
    //       }      
    // }
}
