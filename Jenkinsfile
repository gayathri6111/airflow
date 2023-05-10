pipeline{
    agent any
    environment{
        GIT_URL = "https://conduent-ghs-02.visualstudio.com/CMdS%20EDI/_git/edi-devops"
        GIT_CREDS = "azuregit"
         ACR_ID     = credentials('acr-id')
        ACR_SECRET = credentials('acr-secret')  
    }
    parameters{
        booleanParam(name: 'DEV_DEPLOY', defaultValue: false, description: 'Select this to deploy the dev  kafka in AWS')
        booleanParam(name: 'SIT_DEPLOY', defaultValue: false, description: 'Select this to deploy the sit kafka in AWS')
        gitParameter(name: 'BRANCH', defaultValue: 'none', description: 'Select the name of the branch you need to build and deploy', branchFilter: 'origin/(.*)', type: 'PT_BRANCH')
    }
    stages{
        stage('Checkout'){
            steps{
                cleanWs()
                script{
                    def now = new Date()
                    env.TIMESTAMP = now.format("yyyyMMddHHmmss",TimeZone.getTimeZone("America/New_York"))
                    def myGitBranch = "*/" + "${params.BRANCH}"
                    def scmVars = checkout([
                        $class: 'GitSCM', 
                        branches: [[name: "${myGitBranch}"]],
                        userRemoteConfigs: [[credentialsId: "${GIT_CREDS}",url: "${GIT_URL}"]]
                    ])
                }
            }
        }
        stage('DEV_DEPLOY'){
            when{
                expression{params.DEV_DEPLOY}
            }
            steps{
                script{
                    sh """
                        cd airflow
                        
                        kubectl apply -n edi-airflow -f ./airflow_namespace.json --validate=false                       
                        sleep 1m
                        kubectl apply -n edi-airflow -f airflow-configmap.yaml --validate=false
                        sleep 1m
                        kubectl apply -n edi-airflow -f ./airflow_redis_deployment.yml --validate=false
                        sleep 1m
                        kubectl apply -n edi-airflow -f ./airflow_flower_deployment.yml --validate=false
                        sleep 1m
                        kubectl apply -n edi-airflow -f ./airflow_scheduler_deployment.yaml --validate=false
                        sleep 1m
                        kubectl apply -n edi-airflow -f ./airflow_service.yaml --validate=false
                        sleep 1m
                        kubectl apply -n edi-airflow -f ./airflow_webserver_deployment.yaml --validate=false
                        sleep 1m
                        kubectl apply -n edi-airflow -f ./airflow_worker_deployment.yaml --validate=false
                        sleep 1m
                    """
                }
            }
        }
         stage('SIT_DEPLOY'){
            when{
                expression{params.SIT_DEPLOY}
            }
            steps{
                script{
                    sh """
                        cd kubernetes-kafka-azure/sit
                        
                        
                        kubectl apply -n sit-edi-kafka -f ./zookeeperdeployment.yml  --validate=false                       
                        sleep 1m
                        kubectl apply -n sit-edi-kafka -f ./kafkadeployment.yml --validate=false
                       
                    """
                }
            }
        }
    }
}