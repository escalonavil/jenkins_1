pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = "localhost:5000"
        IMAGE_NAME = "demo-ansible"

        // Jenkins expone la variable GIT_COMMIT automáticamente
        IMAGE_TAG = "${env.GIT_COMMIT}"

        DOCKER_USER = "admin"
        DOCKER_PASS = "Adm1n!2026"
    }

    stages {

        stage('Check') {
            steps {
                script {
                    sh '''
                        echo "🔍 Verificando conexión con Docker..."
                        docker info || echo "Docker no disponible"
                        echo "✅ Runner operativo"
                    '''
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    sh '''
                        cat <<EOF > Dockerfile
                        FROM alpine:latest
                        RUN echo "Imagen construida en CI/CD"
                        CMD ["sh"]
                        EOF

                        docker build -t $DOCKER_REGISTRY/$IMAGE_NAME:$IMAGE_TAG .
                    '''
                }
            }
        }

        stage('Push') {
            steps {
                script {
                    sh '''
                        echo "📦 Iniciando push al Nexus local..."

                        echo "$DOCKER_PASS" | docker login http://$DOCKER_REGISTRY \
                        -u "$DOCKER_USER" \
                        --password-stdin

                        docker push $DOCKER_REGISTRY/$IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh '''
                        echo "🚀 Desplegando con Ansible..."

                        docker run --rm \
                          -v $(pwd):/ansible \
                          -w /ansible \
                          willhallonline/ansible:latest \
                          ansible-playbook -i hosts nginx.yml \
                          -e "frontend_version=$IMAGE_TAG"
                    '''
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
