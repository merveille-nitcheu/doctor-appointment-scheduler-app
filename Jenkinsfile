pipeline{
    agent any

    tools {
    maven 'Maven3'
    jdk 'Java17'
    }

    environment {

        IMAGE_TAG = "version-${env.BUILD_NUMBER}"
        IMAGE_NAME = "${APP_NAME}:${IMAGE_TAG}"
        IMAGE_LATEST = "${APP_NAME}:latest"

    }
    stages{
        stage('Clean Workspace') {
            steps {
                echo 'Cleaning workspace...'
                echo "Current GIT Branch: ${env.GIT_BRANCH}"
                cleanWs(cleanWhenNotBuilt: true, notFailBuild: true, deleteDirs: true)
            }
        }

        // stage('Clone Repositorie') {
        //     // when {
        //     //     expression { env.GIT_BRANCH == 'origin/develop' }
        //     // }
        //     steps {
        //         echo 'Cloning repositories....'
        //         script {
        //             try {
        //                 checkout scm


        //             } catch (Exception e) {
        //                 error "Error cloning repositories: ${e.message}"
        //             }
        //         }
        //     }
        // }

        // stage('Build Maven Project') {
        //     // when {
        //     //     expression { env.GIT_BRANCH == 'origin/develop' }
        //     // }
        //     steps {
        //         echo 'Build Maven Project....'
        //         script {
        //             try {
        //                 bat 'mvn clean install -DskipTests'


        //             } catch (Exception e) {
        //                 error "Error build project: ${e.message}"
        //             }
        //         }
        //     }
        // }

        // stage('Run units Tests') {
        //     // when {
        //     //     expression { env.GIT_BRANCH == 'origin/develop' }
        //     // }
        //     steps {
        //         echo 'Run units Tests....'
        //         script {
        //             try {
        //                 bat 'mvn test'


        //             } catch (Exception e) {
        //                 error "Error build project: ${e.message}"
        //             }
        //         }
        //     }
        // }

        // stage( 'OWASP Dependecy-Check' ) { 
        //     steps { 
        //         echo 'OWASP Dependecy-Check....'
        //         script{
        //             try {
        //                 dependencyCheck additionalArguments: ''' 
        //                     -o './' 
        //                     -s './' 
        //                     -f 'ALL' 
        //                     --prettyPrint''' , odcInstallation: 'dependecy_ckeck'
                
        //                 dependencyCheckPublisher pattern: 'dependency-check-report.xml'
        //             }
        //             catch (Exception e) {
        //                 error "Error dependacy-check project: ${e.message}"
        //             }
        //         }
                
        //     } 
        // }
        
        // stage('SonarQube Analysis') {
        //     steps{

        //         echo 'Run SonarQube Analysis...'
        //         script{
        //             try {
        //                 withSonarQubeEnv() {
        //                     bat "mvn clean verify sonar:sonar -Dsonar.projectKey=merveille-nitcheu_doctor-appointment-scheduler-app_AZfQiTNqKa9jn88UUm0-"
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

        stage('Build Docker Images') {
            
            steps {
                echo 'Building Docker images...'
                script {
                    try {

                        bat "docker build --no-cache -t ${env.IMAGE_NAME} ."

                        bat "docker tag ${env.IMAGE_NAME} ${env.IMAGE_LATEST}"

                        



                    } catch (Exception e) {
                        error "Error building Docker images: ${e.message}"
                    }
                }
            }
        }


    }
    // post{
    //     always{
    //         echo "========always========"
    //     }
    //     success{
    //         echo "========pipeline executed successfully ========"
    //     }
    //     failure{
    //         echo "========pipeline execution failed========"
    //     }
    // }
}