node("master"){

// Ensure that it Clone the repository.
    stage ("Clone") {
        dir("/var/www/") {
            sh "git clone https://github.com/shahidshaikh1005/microservices-demo.git"
        }
    }

// Ensure branch is Checkedout and take latest pull from the specified branch.
    stage("SCM Checkout") {
        dir("/var/www/microservices-demo/src/loadgenerator") {
            sh "git fetch"
            sh "git checkout ${env.BRANCH_NAME}"
            sh "git pull origin ${env.BRANCH_NAME}"
        }
    }
    
// Ensure that Docker image is build and push it into the repository.
    stage("Build") {
        dir("/var/www/microservices-demo/src/loadgenerator") {
            sh "docker build -t gcr.io/google-samples/microservices-demo/loadgenerator:v0.2.${env.BUILD_ID} ."
            sh "docker push gcr.io/google-samples/microservices-demo/loadgenerator:v0.2.${env.BUILD_ID}"
        }
    }

// Ensure that this stage download all the dependencies that required for the application for deployment.
    // stage ("Install") {
    //     dir("/var/www/microservices-demo/src/loadgenerator") {
    //         sh "npm install"
    //         sh "npm audit-fix"
    //     }  
    // }

// Ensure that it download the yaml file with the current jenkins build number and deploy it in its services.
    stage("Deploy") {
        dir("/var/www/microservices-demo/kubernetes-manifests/") {
            sh "curl -XGET --header 'PRIVATE-TOKEN: <private-token>' 'https://github.com/shahidshaikh1005/microservices-demo/blob/master/kubernetes-manifests/loadgenerator.yaml' -o ./ad-service.yaml"
            sh "sed 's/__VERSION__/'${env.BUILD_ID}'/g' ./ad-service.yaml > ./loadgenerator.yaml"
            sh "kubectl config use-context ${env.CLUSTER_NAME}"
            sh "kubectl apply -f loadgenerator.yaml"
        }
    }   

// Ensure that it Cleans the docker-image that was build in last 48-hours. 
    stage("Clean") {
        dir("/var/www/microservices-demo/tmp") {
            sh "docker image prune -a --filter 'until=48h' -f"
        }
    }  
}