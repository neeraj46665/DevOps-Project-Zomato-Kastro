pipeline {
    agent any
    tools {
       
        nodejs 'node23'
    
    }
    environment {
        SCANNER_HOME=tool 'sonar6.2'
        imageName = "neeraj46665/zomato"
    }
    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Enter Docker image tag')
    }
    
    stages {
        stage ("clean workspace") {
            steps {
                cleanWs()
            }
        }
        stage ("Git Checkout") {
            steps {
                git url: "https://github.com/neeraj46665/DevOps-Project-Zomato-Kastro.git", branch: "master"
            }
        }
        stage("Sonarqube Analysis"){
            steps{
                withSonarQubeEnv('sonarserver') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=zomato \
                    -Dsonar.projectKey=zomato '''
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=zomato -Dsonar.projectKey=zomato -Dsonar.verbose=true
'''
                }
            }
        }
        stage("Code Quality Gate"){
          steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
            } 
        }
        stage("Install NPM Dependencies") {
            steps {
                sh 'rm -rf node_modules'
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit -n', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
    }
}
        // stage ("Trivy File Scan") {
        //     steps {
        //         // sh "trivy fs . > trivy.txt"
        //         sh "trivy fs --scanners vuln --skip-files dependency-check-report.xml --format json --output trivy.txt ."


        //     }
        // }
        stage ("Build Docker Image") {
            steps {
                sh "docker build -t ${imageName}:${params.IMAGE_TAG} ."
            }
        }
        stage ("Push to DockerHub") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerlogin') {
                       
                        sh "docker push ${imageName}:${params.IMAGE_TAG}"
                    }
                }
            }
        }
         stage ("Update Kubernetes Manifest") {
            steps {
                script {
                    // Update the image tag in deployment.yaml
                    dir("/var/lib/jenkins/workspace/zomato-cicd/Kubernetes") {
                        sh """
                            
                            sed -i 's|image: neeraj46665/zomato:[^[:space:]]*|image: neeraj46665/zomato:${params.IMAGE_TAG}|' deployment.yaml

                          
                        """
                    }
                }
            }
        }


        
        stage ("Git Commit and Push") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Github-cred', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    script {
                        sh """
                        git config user.name "neeraj"
                        git config user.email "na46665@gmail.com"
                        git add Kubernetes/deployment.yaml
        
                        # Check if there are changes to commit
                        if git diff --cached --quiet; then
                            echo "No changes to commit."
                        else
                            git commit -m "Update image tag to ${params.IMAGE_TAG}"
                            git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/neeraj46665/DevOps-Project-Zomato-Kastro.git master
                        fi
                        """
                    }
                }
            }
        }


        stage ("Trigger ArgoCD Sync") {
            steps {
                echo "ArgoCD will detect the change and sync the application automatically."
            }
        }
    
    }
    post {
        // always {
        //     cleanWs()
        // }
        success {
            echo 'Build succeeded!'
        }
        failure {
            echo 'Build failed!'
        }
    }
    
}
