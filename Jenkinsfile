pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("sonar quality check"){
            agent {
              docker {
                image 'openjdk:11'
              }
            }
            steps{
               script{
                withSonarQubeEnv(credentialsId: 'sonar-token') {
                       sh 'chmod +x gradlew'
                       sh './gradlew sonarqube'

                }
                timeout(time: 1, unit: 'HOURS') {
                    def qg = waitForQualityGate()
                    if (qg.status != 'OK') {
                          error "Pipeline aborted due to quality gate failure: ${qg.status}"
                    }
                  }
              } 
            }
        }

        //Stage 2 of the CI/CD Pipeline..
        stage("2. Building and pushing docker image "){
        steps{
            script{
              withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
              sh '''
                  docker build --rm -t 44.221.170.7:8083/springapp:${VERSION} .
                  docker login -u admin -p $docker_password 44.221.170.7:8083
                  docker push 44.221.170.7:8083/springapp:${VERSION}
                  docker rmi 44.221.170.7:8083/springapp:${VERSION}
              '''}       
            }
        }
      }
    }
}
//         //     //Stage 3 Using Datree to indentifing misconfig 

//             stage('3. Identify mis-config in helm using datree'){
//               steps{
//                 script{    
//                   dir('kubernetes/'){
//                     withEnv(['DATREE_TOKEN=db7d5b3b-4ce9-4973-8176-2cc9234cb97c']){
//                        sh 'helm datree test myapp'
//                   }     
//                 }
//             }
//          }
//         }
//         // Stage 4: Adding Helm chart to Nexus

//             stage("4. Pushing the helm chart to nexus repository"){
//             steps{
//                 script{
//                   withCredentials([string(credentialsId: 'nexus_pass', variable: 'docker_pass')]) {
//                     dir('kubernetes/'){
//                         sh '''
//                         helmversion=$( helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
//                         tar -czvf myapp-${helmversion}.tgz myapp/
//                         curl -u admin:$docker_pass http://3.131.154.78:8081/repository/helm-hosted-k8s/ --upload-file myapp-${helmversion}.tgz -v
//                         '''}       
//                         }
//                     }
//                 }
//             }

// } // to close stages
// } //to close pipeline 
