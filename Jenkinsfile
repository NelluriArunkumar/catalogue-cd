pipeline {
    agent {
        label 'AGENT-1'
    }

    environment {
        appVersion = ''
        REGION = 'us-east-1'
        ACC_ID = '850960379432'
        PROJECT = 'roboshop'
        COMPONENT = 'catalogue'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds() //It is used to disable the parallel Builds of the job
    }

    parameters {
        string(name: 'appVersion', description: 'Image version of the appilication')
        choice(name: 'deploy_to', choices: ['dev','qa','prod'], description: 'Pick the environment')
        
    }


    stages{

        stage('Deploy'){
            steps{
                script{
                    withAWS(credentials: 'aws_creds', region: 'us-east-1'){
                        sh """
                            aws eks update-kubeconfig --region $REGION --name "$PROJECT-${params.deploy_to}"
                            kubectl get nodes
                            kubectl apply -f 01-namespaces.yaml
                            sed -i "s/IMAGE_VERSION/${params.appVersion}/g" values-${params.deploy_to}.yaml
                            helm upgrade --install $COMPONENT  -f values-${params.deploy_to}.yaml -n $PROJECT .
                        """
                    }
                }
            }
        }
    

        stage('Check Status'){
            steps{
                script{
                    withAWS(credentials: 'aws_creds', region: 'us-east-1'){
                        def deploymentStatus = sh(returnStdout: true, script: "kubectl rollout status deployment/catalogue --timeout=30s -n $PROJECT || echo FAILED").trim()
                        if (deploymentStatus.contains("successfully rolled out")) {
                            echo "Deployment is success"
                        } else {
                            sh """
                                helm rollback $COMPONENT -n $PROJECT
                                sleep 20
                            """
                            def rollbackStatus = sh(returnStdout: true, script: "kubectl rollout status deployment/catalogue --timeout=30s -n $PROJECT || echo FAILED").trim()
                            if (rollbackStatus.contains("successfully rolled out")){
                                error "Deployment is Failure, Rollback is Success"
                            }
                            else{
                                error "Deployment is Failure..Rollback  is Failue..Application is not running..." 
                            }
                        }
                       
                    }
                }
            }
        }



        

        
    }
    
    post {
        always {
            echo 'I will always say hello again'
            deleteDir()
        }

        success {
            echo 'Hello Success'
        }

        failure {
            echo "Hello Failure"
        }
    }
}
