pipeline{
    agent any

    tools {
    maven 'Maven3'
    jdk 'Java17'
    }

    environment {

        IMAGE_TAG = "version-${env.BUILD_NUMBER}"
        SONAR_PROJECT_KEY = "merveille-nitcheu_doctor-appointment-scheduler-app_AZfQiTNqKa9jn88UUm0-"
        ECR_REGISTRY = "890742601171.dkr.ecr.us-east-2.amazonaws.com"
        ECR_REPO = "java_project/doctor_appointment"
        IMAGE_NAME = "${env.ECR_REGISTRY}/${env.ECR_REPO}"
        AWS_DEFAULT_REGION= "us-east-2"

    }
    stages{
        stage('Clean Workspace') {
            steps {
                echo 'Cleaning workspace...'
                echo "Current GIT Branch: ${env.GIT_BRANCH}"
                cleanWs(cleanWhenNotBuilt: true, notFailBuild: true, deleteDirs: true)
            }
        }



        stage('Clone Repositorie') {
            // when {
            //     expression { env.GIT_BRANCH == 'origin/develop' }
            // }
            steps {
                echo 'Cloning repositories....'
                script {
                    try {
                        
                        checkout scm
                    } catch (Exception e) {
                        error "Error cloning repositories: ${e.message}"
                    }
                }
            }
        }

        stage('Build Maven Project') {
            // when {
            //     expression { env.GIT_BRANCH == 'origin/develop' }
            // }
            steps {
                echo 'Build Maven Project....'
                script {
                    try {
                        bat 'mvn clean install -DskipTests'


                    } catch (Exception e) {
                        error "Error build project: ${e.message}"
                    }
                }
            }
        }

        stage('Run units Tests') {
            // when {
            //     expression { env.GIT_BRANCH == 'origin/develop' }
            // }
            steps {
                echo 'Run units Tests....'
                script {
                    try {
                        bat 'mvn test'


                    } catch (Exception e) {
                        error "Error build project: ${e.message}"
                    }
                }
            }
        }

        stage( 'OWASP Dependecy-Check' ) { 
            steps { 
                echo 'OWASP Dependecy-Check....'
                script{
                    try {
                        dependencyCheck additionalArguments: ''' 
                            -o './' 
                            -s './' 
                            -f 'ALL' 
                            --prettyPrint''' , odcInstallation: 'dependecy_ckeck'
                
                        dependencyCheckPublisher pattern: 'dependency-check-report.xml'
                    }
                    catch (Exception e) {
                        error "Error dependacy-check project: ${e.message}"
                    }
                }
                
            } 
        }
        
        // stage('SonarQube Analysis') {
        //     steps{

        //         echo 'Run SonarQube Analysis...'
        //         script{
        //             try {
        //                 withSonarQubeEnv() {
        //                     bat "mvn clean verify sonar:sonar -Dsonar.projectKey=${env.SONAR_PROJECT_KEY}"
        //                 }
        //                 timeout(time: 2, unit: 'MINUTES') {
        //                     def qg = waitForQualityGate() 
        //                     if (qg.status != 'OK') {
        //                         error "Pipeline aborted due to quality gate failure: ${qg.status}"
        //                     }
        //                 }
        //             }
        //             catch (Exception e) {
        //                 error "Error build project: ${e.message}"
        //             }
        //         } 
        //     }                  
        // }

        stage('Login to ECR') {
            
            steps {
                echo 'Login to ECR...'
                script {
                    try {
                            

                        bat ''' aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 890742601171.dkr.ecr.us-east-2.amazonaws.com '''

                    } catch (Exception e) {
                        error "Error Login to ECR: ${e.message}"
                    }
                }
            }
        }

        stage('Build Docker Images') {
            
            steps {
                echo 'Building Docker images...'
                script {
                    try {

                        bat "docker build --no-cache -t ${env.ECR_REPO}:${env.IMAGE_TAG} ."

                        bat "docker tag ${env.ECR_REPO}:latest ${env.IMAGE_NAME}:latest "

                    } catch (Exception e) {
                        error "Error building Docker images: ${e.message}"
                    }
                }
            }
        }



        // stage('Push Images on ECR') {
            
        //     steps {
        //         echo 'Push Images on ECR...'
        //         script {
        //             try {

        //                 bat "docker push ${env.IMAGE_NAME}:latest"

        //                 bat "docker push ${env.IMAGE_NAME}:${env.IMAGE_TAG}"

        //             } catch (Exception e) {
        //                 error "Error pushing Docker images: ${e.message}"
        //             }
        //         }
        //     }
        // }

        stage('Generate Sbom & Push on Dependency Track') {
            
            steps {
                echo 'Generate Sbom & Push on Dependency Track...'
                script {
                    try {

                        bat 'mvn org.cyclonedx:cyclonedx-maven-plugin:makeAggregateBom'

                        withCredentials([string(credentialsId: 'Dependency_track', variable: 'API_KEY')]) {
                            dependencyTrackPublisher(
                                artifact: 'target/bom.xml',
                                projectName: "${env.APP_NAME}",
                                projectVersion: "${env.BUILD_NUMBER}",
                                synchronous: true,
                                dependencyTrackApiKey: API_KEY,
                                
                            )
                        }



                    } catch (Exception e) {
                        error "Error pushing Docker images: ${e.message}"
                    }
                }
            }
        }


    }
    post{
        always{
            cleanWs()
        }
        success {
        echo "======== Pipeline executed successfully ========"
            emailext(
                to: "merveillenitcheu12@gmail.com",
                subject: "✅ Jenkins Pipeline Succeeded: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """\
                    The GitOps pipeline has completed successfully.

                    ✔ Job: ${env.JOB_NAME}
                    ✔ Build: #${env.BUILD_NUMBER}
                    ✔ Result: ${currentBuild.currentResult}
                    🔗 Jenkins URL: ${env.BUILD_URL}

                    Everything looks good!
                    """
            )
        }

        failure {
            emailext(
                to: "merveillenitcheu12@gmail.com",
                subject: "❌ Jenkins Pipeline Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """\
                    The GitOps pipeline has failed.

                    ✖ Job: ${env.JOB_NAME}
                    ✖ Build: #${env.BUILD_NUMBER}
                    ✖ Result: ${currentBuild.currentResult}
                    🔗 Jenkins URL: ${env.BUILD_URL}

                    Please check the Jenkins logs for more details.
                    """
            )
        }
    }
}