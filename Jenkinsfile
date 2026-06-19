pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = "localhost:5000"
        IMAGE_NAME = "demo-ansible"

        // Jenkins expone  la variable GIT_COMMIT automáticamente
        IMAGE_TAG = "${env.GIT_COMMIT}"

        DOCKER_USER = "admin"
        DOCKER_PASS = "Adm1n!2026"
    }

    stages {

        stage('Check') {
            steps {
                script {
                    echo "🔍 [CHECK] Iniciando verificación de Docker..."
                    sh '''
                        docker info || echo "⚠️ Docker no disponible"
                        echo "✅ Runner operativo"
                    '''
                    echo "✔️ [CHECK] Finalizado"
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    echo "🔨 [BUILD] Construyendo imagen Docker..."
                    sh '''
                        cat <<EOF > Dockerfile
                        FROM alpine:latest
                        RUN echo "Imagen construida en CI/CD"
                        CMD ["sh"]
                        EOF

                        docker build -t $DOCKER_REGISTRY/$IMAGE_NAME:$IMAGE_TAG .
                    '''
                    echo "✔️ [BUILD] Imagen construida: $DOCKER_REGISTRY/$IMAGE_NAME:$IMAGE_TAG"
                }
            }
        }

        stage('Push') {
            steps {
                script {
                    echo "📦 [PUSH] Subiendo imagen al registry..."
                    sh '''
                        echo "$DOCKER_PASS" | docker login http://$DOCKER_REGISTRY \
                        -u "$DOCKER_USER" \
                        --password-stdin

                        docker push $DOCKER_REGISTRY/$IMAGE_NAME:$IMAGE_TAG
                    '''
                    echo "✔️ [PUSH] Imagen publicada en $DOCKER_REGISTRY"
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo "🚀 [DEPLOY] Ejecutando Ansible..."
                    sh '''
                        docker run --rm \
                          -v $(pwd):/ansible \
                          -w /ansible \
                          willhallonline/ansible:latest \
                          ansible-playbook -i hosts nginx.yml \
                          -e "frontend_version=$IMAGE_TAG"
                    '''
                    echo "✔️ [DEPLOY] Despliegue completado con Ansible"
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completado correctamente'
        }
        failure {
            echo '❌ El pipeline falló'
        }
    }
}
