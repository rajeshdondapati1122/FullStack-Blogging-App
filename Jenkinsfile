pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
        
    }
    
    environment {
        SCANNER_HOME= tool'sonar-scanner'
    }

    stages {
        stage('git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/rajeshdondapati1122/FullStack-Blogging-App.git'
            }
        }
        
        stage('compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('test') {
            steps {
                sh "mvn test"
            }
        }
        
        stage('trivy file scan') {
            steps {
                sh "trivy fs --format table -o fs.html ."
            }
        }
        stage('sonarqube analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=blog-app -Dsonar.projectKey=blog-app \
                       -Dsonar.java.binaries=target'''
    
               }
            }
        }
        
    
        
        stage('build artifact') {
            steps {
                sh "mvn package"
            }
        }
        
        stage('publish artifact') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy"
               }
            }
        }
        
        stage('docker build') {
            steps {
                script{
                    
                withDockerRegistry(credentialsId: 'docker-cred') {
                
                sh "docker build -t rajeshdondapati309/bloggingapp:latest ."
    
                 }
            }
          }
        }
        
        stage('trivy image scan') {
            steps {
                sh "trivy image --format table -o image.html rajeshdondapati309/bloggingapp:latest "
            }
        }
        
        stage('docker push') {
            steps {
                script{
                    
                withDockerRegistry(credentialsId: 'docker-cred') {
                
                sh "docker push rajeshdondapati309/bloggingapp:latest"
    
                 }
            }
          }
        }
        
        
        stage('k8-deploy') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://FF8921903E1533377C67A277D86176B4.gr7.us-west-1.eks.amazonaws.com') {
                    sh "kubectl apply -f deployment-service.yml"
                    sleep 20
               }
            }
        }
        
        stage('k8-verify') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://FF8921903E1533377C67A277D86176B4.gr7.us-west-1.eks.amazonaws.com') {
                    sh "kubectl get pods"
                    sh "kubectl get svc"
               }
            }
        }
        
        

        
        
        
        
    }
    
    post {
    always { 
        script { 
            def jobName = env.JOB_NAME
            def buildNumber = env.BUILD_NUMBER
            def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
            def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

            def body = """
                <html>
                <body>
                <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                <h2>${jobName} - Build ${buildNumber}</h2>
                <div style="background-color: ${bannerColor}; padding: 10px;">
                <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                </div>
                <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                </div>
                </body>
                </html>
            """

            emailext (
                subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                body: body,
                to: 'rajeshdondapati309@gmail.com',
                from: 'jenkins@example.com',
                replyTo: 'jenkins@example.com',
                mimeType: 'text/html',
                attachmentsPattern: 'image.html'
            )
        }
    }
  }
}
