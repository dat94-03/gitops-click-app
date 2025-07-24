pipeline {
    agent any

    stages {
        stage('Checkout Manifests') {
            steps {
                git branch: 'main', url: 'https://github.com/yourorg/gitops-repo.git'
            }
        }

        stage('Validate Helm Charts') {
            steps {
                sh '''
                helm lint charts/app
                helm template charts/app --values charts/app/values-staging.yaml
                '''
            }
        }

        stage('Sync to Cluster (optional, GitOps)') {
            steps {
                withKubeConfig([credentialsId: 'eks-kubeconfig']) {
                    sh '''
                    helm upgrade --install app-staging charts/app \
                        --namespace app-staging \
                        --create-namespace \
                        --values charts/app/values-staging.yaml
                    '''
                }
            }
        }

        stage('Audit Rollout (optional)') {
            steps {
                withKubeConfig([credentialsId: 'eks-kubeconfig']) {
                    sh '''
                    kubectl get deployment app-staging -n app-staging
                    kubectl rollout status deployment/app-staging -n app-staging
                    '''
                }
            }
        }
    }
}
