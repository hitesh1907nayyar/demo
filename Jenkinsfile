podTemplate(
    label: 'mypod', 
    inheritFrom: 'default',
    containers: [
        containerTemplate(
            name: 'golang', 
            image: 'golang:1.10-alpine',
            ttyEnabled: true,
            command: 'cat'
        ),
        containerTemplate(
            name: 'docker', 
            image: 'docker:18.02',
            ttyEnabled: true,
            command: 'cat'
        ),
        containerTemplate(
            name: 'helm', 
            image: 'ibmcom/k8s-helm:v2.6.0',
            ttyEnabled: true,
            command: 'cat'
        )
    ]
) {
    node('mypod') {
        def commitId
        stage ('Extract') {
            checkout scm
            commitId = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        }
        stage ('Build') {
            container ('golang') {
                sh 'CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .'
				sh 'ls -ltr'
            }
        }
        def repository
        stage ('Docker') {
            container ('docker') {
			    withCredentials([[$class: 'UsernamePasswordMultiBinding',
						credentialsId: '123456',
						usernameVariable: 'DOCKER_HUB_USER',
						passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
					sh "docker build -t ${env.DOCKER_HUB_USER}/demo:${env.BUILD_NUMBER} ."
					sh "docker login -u ${env.DOCKER_HUB_USER} -p ${env.DOCKER_HUB_PASSWORD} "
					sh "docker push ${env.DOCKER_HUB_USER}/demo:${env.BUILD_NUMBER} "
				}
            }
        }
        stage ('Deploy') {
            container ('helm') {
			    withCredentials([[$class: 'UsernamePasswordMultiBinding',
						credentialsId: '123456',
						usernameVariable: 'DOCKER_HUB_USER',
						passwordVariable: 'DOCKER_HUB_PASSWORD']]){	
                    sh "/helm init --client-only --skip-refresh"
                    sh "/helm upgrade --install --wait --set image.repository=${env.DOCKER_HUB_USER}/demo,image.tag=${env.BUILD_NUMBER} demo hello"
				}
            }
        }
    }
}

