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
                    // 1. Login a ECR usando la credencial 'aws-lab' (Paso 5.2)
                    withAWS(region: "${params.AWS_REGION}", credentials: 'aws-lab') {
                        sh 'aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com'
                    }

                    // 2. Construir la imagen con la etiqueta de la compilación de Jenkins
                    sh 'docker build -t ${ECR_REPO}:${BUILD_NUMBER} .'
                    
                    // 3. Etiquetar la imagen con el repositorio de ECR
                    sh 'docker tag ${ECR_REPO}:${BUILD_NUMBER} ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${BUILD_NUMBER}'

                    // 4. Subir la imagen a ECR
                    sh 'docker push ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${BUILD_NUMBER}'
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    // 1. Obtener la Task Definition más reciente para usarla como plantilla
                    sh "aws ecs describe-task-definition --task-definition ${TASK_FAMILY} --region ${AWS_REGION} > task-definition.json"

                    // 2. Crear una nueva definición de tarea actualizando la imagen URI
                    sh """
                    NEW_TASK_DEFINITION=\$(jq '.taskDefinition | del(.taskDefinitionArn) | del(.revision) | del(.status) | del(.requiresAttributes) | .containerDefinitions[0].image = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${BUILD_NUMBER}" | .' task-definition.json)
                    aws ecs register-task-definition --region ${AWS_REGION} --cli-input-json "\${NEW_TASK_DEFINITION}"
                    """

                    // 3. Actualizar el servicio ECS para usar la nueva imagen
                    sh "aws ecs update-service --cluster ${ECS_CLUSTER} --service ${ECS_SERVICE} --force-new-deployment --region ${AWS_REGION}"
                }
            }
        }
    }
}
