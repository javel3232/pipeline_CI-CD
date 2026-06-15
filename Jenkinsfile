/**
 * Pipeline de Entrega Continua (CD) con Jenkins
 * 
 * Este pipeline define los stages necesarios para:
 * 1. Clonar el repositorio desde GitHub
 * 2. Ejecutar pruebas para validación pre-despliegue
 * 3. Construir la imagen Docker de la aplicación
 * 4. Publicar la imagen en DockerHub
 * 5. Desplegar en un cluster de Kubernetes (entorno agnóstico)
 *
 * Herramientas utilizadas:
 * - Jenkins: Orquestador del pipeline CD
 * - Docker: Containerización de la aplicación
 * - kubectl: Despliegue en Kubernetes
 * - DockerHub: Registro de imágenes de contenedores
 */

pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "usuario/labtech-app"
        DOCKER_TAG = "${BUILD_NUMBER}"
        REGISTRY_CREDENTIALS = credentials('dockerhub-credentials')
        KUBECONFIG_CREDENTIALS = credentials('kubeconfig-credentials')
    }

    stages {
        // Stage 1: Clonar el repositorio desde GitHub
        stage('Checkout') {
            steps {
                echo 'Clonando repositorio desde GitHub...'
                git branch: 'main',
                    url: 'https://github.com/usuario/labTec.git'
            }
        }

        // Stage 2: Instalar dependencias y ejecutar pruebas
        stage('Test') {
            steps {
                echo 'Ejecutando pruebas pre-despliegue...'
                sh '''
                    python -m pip install -r requirements.txt
                    pytest tests/ -v
                '''
            }
        }

        // Stage 3: Construir imagen Docker
        stage('Build Docker Image') {
            steps {
                echo 'Construyendo imagen Docker...'
                sh """
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                    docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                """
            }
        }

        // Stage 4: Publicar imagen en DockerHub
        stage('Push to Registry') {
            steps {
                echo 'Publicando imagen en DockerHub...'
                sh """
                    echo \$REGISTRY_CREDENTIALS_PSW | docker login -u \$REGISTRY_CREDENTIALS_USR --password-stdin
                    docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                    docker push ${DOCKER_IMAGE}:latest
                """
            }
        }

        // Stage 5: Despliegue en Kubernetes (entorno agnóstico)
        stage('Deploy to Kubernetes') {
            steps {
                echo 'Desplegando en cluster Kubernetes...'
                sh """
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    kubectl rollout status deployment/labtech-app --timeout=120s
                """
            }
        }
    }

    post {
        success {
            echo 'Pipeline CD ejecutado exitosamente. Aplicación desplegada.'
        }
        failure {
            echo 'Error en el pipeline CD. Revisar logs.'
        }
        always {
            sh 'docker logout || true'
        }
    }
}
