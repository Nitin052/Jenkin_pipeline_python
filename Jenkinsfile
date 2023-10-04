pipeline {
    agent any
    
      environment {
        NEXUS_URL = '192.168.34.3:7090'
        DOCKER_IMAGE = '192.168.34.3:7090/flask-app:v1'
        DOCKER_CONTAINER = 'flask-app'
        DOCKER_USERNAME = 'admin'
        DOCKER_PASSWORD = credentials('nexus_cred')
    }

    stages{
    stage('Pre-req'){
        steps{
          //  sh 'minikube delete'
            sh 'minikube start --driver docker'
            sh 'eval $(minikube docker-env)'
            git url: 'https://github.com/Nitin052/Jenkin_pipeline_python.git',branch: 'main'
        }
    }
    
    stage('code check by sonarqube'){
        steps{
            withSonarQubeEnv("sonartest") {
                    tool name: 'sonartest', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                    sh "${tool("sonartest")}/bin/sonar-scanner"
                    //-Dsonar.projectKey=sonarqubetest -D sonar.projectName=sonarqubetest -D sonar.java.binaries=. -D sonar.sources=. -D sonar.exclusions=build/** "
                    
            }        
    }
  }
  stage('quality check'){
        steps{
        script{
                def qualitygate = waitForQualityGate()
                    if (qualitygate.status != "OK") {
                            error "Pipeline aborted due to quality gate coverage failure: ${qualitygate.status}"
                }
        }
    }        
  }
  stage('get detail'){
      steps{
          sh 'kubectl get all'
      }
  }
    stage('Getting image from Artifactory') {
        steps{
            echo 'Fetching docker image..'
            sh 'docker login ${NEXUS_URL} -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}'
            sh 'docker pull ${DOCKER_IMAGE}'
        }
    }
    stage('Image Scane'){
        steps{
            sh 'trivy image ${DOCKER_IMAGE}'
            //sh 'trivy image --exit-code 2 --severity CRITICAL ${DOCKER_IMAGE}'
            //sh 'trivy image --exit-code 3 --severity HIGH ${DOCKER_IMAGE}'

        }
    }
    stage('Deploy'){
        steps{
            echo 'Deploying ...'
           // sh 'kubectl apply -f secret.yml'
            sh 'kubectl -- create secret docker-registry nexus-secret --docker-server=${NEXUS_URL} --docker-username=${DOCKER_USERNAME} --docker-password=${DOCKER_PASSWORD}'
            sh 'kubectl apply -f deployment.yml'
            sh 'kubectl apply -f service.yml'

        }
    }
 }
}
