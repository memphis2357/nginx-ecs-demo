pipeline {
 agent any
 parameters {
 string(name: 'AWS_REGION', defaultValue: 'sa-east-1')
 string(name: 'ECR_REPO', defaultValue: 'nginx-ecs-demo')
 string(name: 'ECS_CLUSTER', defaultValue: 'ecs-lab-cluster')
 string(name: 'ECS_SERVICE', defaultValue: 'nginx-lab-svc')
 string(name: 'TASK_FAMILY', defaultValue: 'nginx-lab-task')
 string(name: 'ACCOUNT_ID', defaultValue: 'TU_ACCOUNT_ID')
 }
 stages {
 stage('Checkout') { steps { checkout scm } }
 stage('Build & Push') { steps { ... build y push a ECR ... } }
 stage('Deploy') { steps { ... actualizar Task Definition y Service ECS ... } }
 }
}