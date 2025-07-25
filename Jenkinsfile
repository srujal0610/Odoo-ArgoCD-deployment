pipeline {
    agent any

    parameters {
        string(name: 'GIT_REPO_URL', defaultValue: 'https://github.com/srujal0610/Odoo-ArgoCD-deployment.git', description: 'GitHub repo with custom addons')
        string(name: 'BRANCH', defaultValue: 'main', description: 'Branch name to pull')
        string(name: 'K8S_NAMESPACE', defaultValue: 'odoo', description: 'Kubernetes namespace')
        string(name: 'POD_LABEL', defaultValue: 'app=odoo', description: 'Pod label selector')
        string(name: 'KUBE_DEPLOYMENT', defaultValue: 'odoo', description: 'Deployment Name')
        string(name: 'MOUNT_PATH', defaultValue: '/mnt/extra-addons/custom-addons', description: 'Target path inside Odoo pod')
    }

    environment {
        ADDONS_DIR = 'custom-addons'
    }

    stages {
        stage('Clone Git Repo') {
            steps {
                git url: "${params.GIT_REPO_URL}", branch: "${params.BRANCH}"
            }
        }

        stage('Get Odoo Pod') {
            steps {
                script {
                    env.ODOO_POD = sh(
                        script: "kubectl get pods -n ${params.K8S_NAMESPACE} -l ${params.POD_LABEL} -o jsonpath='{.items[0].metadata.name}'",
                        returnStdout: true
                    ).trim()
                    echo "🟢 Odoo Pod: ${env.ODOO_POD}"
                }
            }
        }

        stage('Copy Addons to Pod') {
            steps {
                echo "📁 Copying ${env.ADDONS_DIR} to ${params.MOUNT_PATH} in pod ${env.ODOO_POD}"
                sh """
                    kubectl cp ${env.ADDONS_DIR} ${params.K8S_NAMESPACE}/${env.ODOO_POD}:${params.MOUNT_PATH}
                """
            }
        }

        stage('Restart Odoo deployment') {
            steps {
                sh "kubectl rollout restart deployment/${params.KUBE_DEPLOYMENT} -n ${params.K8S_NAMESPACE}"
            }
        }
    }

    post {
        success {
            echo '✅ Custom addons copied and deployed successfully!'
        }
        failure {
            echo '❌ Something went wrong.'
        }
    }
}
