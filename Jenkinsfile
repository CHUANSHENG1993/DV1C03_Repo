groovy
pipeline {
    agent any
    tools {
        jdk 'jdk11'
        maven 'maven3'
    }
    environment{
        SCANNER_HOME=tool 'sonar-scanner'
    }
    options {
        buildDiscarder logRotator( 
                    daysToKeepStr: '14', 
                    numToKeepStr: '7'
            )
        }
    stages {
        stage('Stage 1') {
            steps {
                cleanWs()
                checkout([$class: 'GitSCM', branches: [[name: 'master']], userRemoteConfigs: [[url: 'https://github.com/CHUANSHENG1993/TestDV1C03.git']]])
                sh 'mvn clean compile'
                sh 'mvn test'
                echo 'Stage1_5683554u : Release Environment Preparation Completed'
            }
        }
        stage('Stage 2') {
            steps {
                script {
                    sh 'docker stop WebApp_5683554u || true'
                    sh 'docker rm WebApp_5683554u || true'
                    sh 'docker run -d -p 31200:80 --name WebApp_5683554u wb1_image_5683554u'
                    echo 'Stage2_5683554u : Release Container WebApp_5683554u Created Completed'
                }
            }
        }
        stage('Stage 3') {
            parallel {
                stage('S3 API TEST') {
                    steps {
                        sh 'jmeter -n -t test.jmx -l results.jtl'
                        junit 'target/surefire-reports/*.xml'
                    }
                }
                stage('S3 SCAN TEST'){
                    steps {
                        withSonarQubeEnvironment('sonar-server') {
                        sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=5683554u_DV1C03 \
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=5683554u_DV1C03 '''
                        dependencyCheck additionalArguments: '--scan ./ --format XML ', odcInstallation: 'DC'
                        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'                        
                        sh 'mvn clean install -DskipTests=true'
                        }
                    }
                }
            }
            post {
                always {
                    script {
                        waitForQualityGateway abortPipeline: false, credentialsId: 'Sonar-token' 
                    }
                    perfReport percentiles: '0,50,90,100', sourceDataFiles: 'results.jtl'
                    junit testResults: 'target/surefire-reports/*.xml', allowEmptyResults: true, healthScaleFactor: 1.0, unstable: 80.0, healthy: 90.0
                    echo "Stage3_5683554u: API test completed"
                    echo "Stage3_5683554u: SCAN test completed"
                }
              }
            }
        stage('Stage 4') {
            steps {
                step([
                    stage: 'Stage 4',
                    task: 'UserInput',
                    inputMode: 'Prompt',
                    parametersStr: 'Prompt: 5683554u, proceed to release the work to next phase?\nAbort'
                ])
                script {
                    if (currentBuild.parameters.proceed) {
                        echo 'Stage 5_5683554u: Work Release - Proceed to Next Phase'
                    } else {
                        echo 'Stage 5_568354u : Work Release - Stops'
                    }
                }
            }
        }
        stage('Stage 5') {
            steps {
                script {
                    if (currentBuild.parameters.proceed) {
                        echo 'Stage 5_568354u: Work Release - Proceed to Next Phase'
                        docker.withDockerRegistry(credentialsId:'docker', toolName: 'docker') {
                        sh."docker build -t 5683554u_DV1C03 ."
                        sh "docker tag 5683554u_DV1C03 sim1993/5683554u_DV1C03:latest"
                        sh "docker push sim1993/5683554u_DV1C03:latest"
                        }
                        sh "trivy sim1993/5683554u_DV1C03:latest > trivy.txt"
                    } else {
                        echo 'Stage 5_568354u : Work Release - Stops'
                    }
                }
            }
        }
    }
    post {
        always {
           emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'zephyrgamesiao@gmail.com',
            attachmentsPattern: 'trivy.txt'
        }
        cleanup {
            sh "docker rm -f WebAPP_5683554u || true"
        }
    }
}