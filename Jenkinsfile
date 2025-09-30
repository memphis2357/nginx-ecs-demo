pipeline {
    agent any
    parameters {
        string(name: 'AWS_REGION', defaultValue: 'sa-east-1')
        string(name: 'ECR_REPO', defaultValue: 'nginx-ecs-demo')
        string(name: 'ECS_CLUSTER', defaultValue: 'ecs-lab-cluster')
        string(name: 'ECS_SERVICE', defaultValue: 'nginx-lab-svc')
        string(name: 'TASK_FAMILY', defaultValue: 'nginx-lab-task')
        string(name: 'ACCOUNT_ID', defaultValue: '197461532451')
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build & Push') {
            steps {
                script {
                    // Exponer las claves de AWS (Solo para el login de ECR)
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-lab', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                        
                        // 1. Login a ECR
                        sh "aws ecr get-login-password --region ${params.AWS_REGION} | docker login --username AWS --password-stdin ${params.ACCOUNT_ID}.dkr.ecr.${params.AWS_REGION}.amazonaws.com"
                    }

                    // 2. Construir la imagen
                    sh "docker build -t ${params.ECR_REPO}:${env.BUILD_NUMBER} ."
                    
                    // 3. Etiquetar la imagen
                    sh "docker tag ${params.ECR_REPO}:${env.BUILD_NUMBER} ${params.ACCOUNT_ID}.dkr.ecr.${params.AWS_REGION}.amazonaws.com/${params.ECR_REPO}:${env.BUILD_NUMBER}"

                    // 4. Subir la imagen a ECR
                    sh "docker push ${params.ACCOUNT_ID}.dkr.ecr.${params.AWS_REGION}.amazonaws.com/${params.ECR_REPO}:${env.BUILD_NUMBER}"
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    // OBLIGATORIO: Envolvemos la lógica de AWS CLI en withCredentials para autenticar los comandos.
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-lab', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                        
                        // 1. Obtener la Task Definition más reciente
                        sh "aws ecs describe-task-definition --task-definition ${params.TASK_FAMILY} --region ${params.AWS_REGION} > task-definition.json"

                        // 2. Crear una nueva definición de tarea actualizando la imagen URI
                        sh """
                        NEW_TASK_DEFINITION=\$(jq '.taskDefinition | del(.taskDefinitionArn) | del(.revision) | del(.status) | del(.requiresAttributes) | .containerDefinitions[0].image = "${params.ACCOUNT_ID}.dkr.ecr.${params.AWS_REGION}.amazonaws.com/${params.ECR_REPO}:${env.BUILD_NUMBER}" | .' task-definition.json)
                        aws ecs register-task-definition --region ${params.AWS_REGION} --cli-input-json "\${NEW_TASK_DEFINITION}"
                        """

                        // 3. Actualizar el servicio ECS
                        sh "aws ecs update-service --cluster ${params.ECS_CLUSTER} --service ${params.ECS_SERVICE} --force-new-deployment --region ${params.AWS_REGION}"
                    }
                }
            }
        }
    }
}
