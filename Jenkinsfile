pipeline {
  
 
  
    agent {
    kubernetes {
      yaml """
apiVersion: v1
kind: Pod
metadata:
          labels:
            label: mylabel
spec:
  # Use service account that can deploy to all namespaces
  ServiceAccountName: jenkins
  containers:
 
  - name: docker
    image: docker:latest
    command:
    - cat
    tty: true
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: dockersock
  volumes:
    - name: dockersock
      hostPath:
        path: /var/run/docker.sock
    - name: jenkins-data
      persistentVolumeClaim:
          claimName: jenkins
    
"""
              }
   }
  
  tools {
        maven "MAVEN"
        jdk "JDK"
  }
  stages {
      stage('Initialize'){
            steps{
                echo "PATH = ${M2_HOME}/bin:${PATH}"
                echo "M2_HOME = /opt/maven"
            }
        }
        stage('Build') {
            steps {
                
                sh 'mvn -Dmaven.test.failure.ignore=true clean package'
               
            
            }
        }
       stage('Building image') {
          steps {
             container('docker') {
          
             sh 'docker build -t zubairbhat722/my-app:latest .'
             }
          }
      }
      stage('Publish image to Docker Hub') {
          
        steps {
           container('docker') {
           withDockerRegistry([ credentialsId: "dockerhub", url: "" ]) {
           sh  'docker push zubairbhat722/my-app:latest'
           
              }
        }
                  
          }
    }
    stage('Scan') {
      steps {
        withSonarQubeEnv(installationName: 'jenkinsSonar') {
          sh 'mvn sonar:sonar'
        }
      }
    }  
    stage('Deploy App') {
      steps {
        script {
          kubernetesDeploy(configs: "java.yaml", kubeconfigId: "mykubeconfigfile")
        }
      }
    }
  }  
           
}
