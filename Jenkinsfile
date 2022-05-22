pipeline {
    agent any
    environment {
        PROJECT_ID = 'devops-sre-rosh'
        CLUSTER_NAME = 'autopilot-cluster-1'
        CLUSTER_LOCATION = 'us-central1'
        REGISTRY_LOCATION = 'northamerica-northeast2'
        REPOSITORY = 'project2'
        CREDENTIALS_ID = 'b75e1aba126f43673dc0e0d79ed5e596ec307c1d'
    }
    stages {
        stage ('Docker Build'){
            steps {
                script {
                    echo "Docker Build"
                    sh "docker images prune"
                    sh "docker build -t directionsapi api2"
                    sh "docker build -t notificationapi notificationApi"
                    sh "docker build -t deliveryapi deliveryApi"
                }
            }
        }
        stage ('Docker tag and push to Google Artifact Repository'){
            steps {
                script {
                    echo "Docker push"
                    sh "docker tag directionsapi northamerica-northeast2-docker.pkg.dev/devops-sre-rosh/project2/directionsapi"
                    sh "docker tag deliveryapi northamerica-northeast2-docker.pkg.dev/devops-sre-rosh/project2/deliveryapi"
                    sh "docker tag notificationapi northamerica-northeast2-docker.pkg.dev/devops-sre-rosh/project2/notificationapi"
                    sh "docker push northamerica-northeast2-docker.pkg.dev/devops-sre-rosh/project2/directionsapi"
                    sh "docker push northamerica-northeast2-docker.pkg.dev/devops-sre-rosh/project2/deliveryapi"
                    sh "docker push northamerica-northeast2-docker.pkg.dev/devops-sre-rosh/project2/notificationapi"
                }
            }
        }
        stage ('Deploy to GKE'){
            steps{
                echo "Deploying to GKE"
                sh "sed -i 's|image: directionsapi|image: northamerica-northeast2-docker.pkg.dev/devops-sre-rosh/project2/directionsapi|g' deployment/directionsapi-deployment.yml"
                sh "sed -i 's|image: deliveryapi|image: northamerica-northeast2-docker.pkg.dev/devops-sre-rosh/project2/deliveryapi|g' deployment/deliveryapi-deployment.yml"
                sh "sed -i 's|image: notificationapi|image: northamerica-northeast2-docker.pkg.dev/devops-sre-rosh/project2/notificationapi|g' deployment/notificationapi-deployment.yml"
                step([$class: 'KubernetesEngineBuilder',
                    projectId: env.PROJECT_ID,
                    clusterName: env.CLUSTER_NAME,
                    location: env.CLUSTER_LOCATION,
                    manifestPattern: 'deployment',
                    credentialsId: env.CREDENTIALS_ID,
                    verifyDeployments: true])
            }
        }
    }
}
