pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }   



      stage('Unit Test - JUnit and Jacoco') {
            steps {
              sh "mvn test"
            }
            post{
                always{
                  junit 'target/surefire-reports/*.xml'
                  jacoco execPattern: 'target/jacoco.exec'
                }
            }
          }
            
     
      stage('SonarQube - SAST') {
            steps {
              sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://devsecopss-demo.eastus.cloudapp.azure.com:9000 -Dsonar.login=0197e8251f45b4c2fabd81dbc080ee0e7a41ca9c"
            }
      }
    
      stage('Docker Build and Push') {
            steps {
              withDockerRegistry([credentialsId: "docker-hub", url:""]) {
                sh 'printenv'
                sh 'docker build -t ideejn/numeric-app:""$GIT_COMMIT"" .'
                sh 'docker push ideejn/numeric-app:""$GIT_COMMIT""'
              }
            }
          }

      stage('Kubernetes Deployment -DEV!') {
            steps {
              withKubeConfig([credentialsId: 'kubeconfig',]) {
                sh "sed -i 's#replace#ideejn/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
                sh "kubectl apply -f k8s_deployment_service.yaml"
                
              }
            }
          }
             
    }
}
