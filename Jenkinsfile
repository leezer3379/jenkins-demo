node('k8s-slave') {
    input message: '', parameters: [choice(choices: ['deploy', 'rollback'], description: 'xu', name: 'Action'), text(defaultValue: '', description: '', name: 'Tag')]
    if(parameters.Action=="delopy"){
        stage('Prepare') {
            echo "1.Prepare Stage"
            checkout scm
            script {
                build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
            }
        }
        stage('Build') {
            echo "2.Build Docker Image Stage"
            sh "docker build -t registry-vpc.cn-beijing.aliyuncs.com/sy-ops/jenkins-demo:${build_tag} ."
        }
        stage('Push') {
            echo "3.Push Docker Image Stage"
            withCredentials([usernamePassword(credentialsId: 'dockerAliyun', passwordVariable: 'dockerAliyunPassword', usernameVariable: 'dockerAliyunUser')]) {
                sh "docker login -u ${dockerAliyunUser} -p ${dockerAliyunPassword} registry-vpc.cn-beijing.aliyuncs.com"
                sh "docker push registry-vpc.cn-beijing.aliyuncs.com/sy-ops/jenkins-demo:${build_tag}"
            }
        }
        stage('Deploy') {
            echo "4. Deploy Stage"
            sh "sed -i 's/<BUILD_TAG>/${build_tag}/' k8s.yaml"
            sh "kubectl apply -f k8s.yaml --record"
        }
    } else if(parameters.Action=="rollback" && parameters.Tag!="") {
        stage("Rollback"){
            echo "${Tag} rollbacking...."
            sh "sed -i 's/<BUILD_TAG>/${Tag}/' k8s.yaml"
            sh "kubectl apply -f k8s.yaml --record"
        }
    } else if (parameters.Tag=="123"){
        stage("Debug"){
            echo "Tag: ${Tag}"
            echo "Action: ${Action}"
        }
    } else {

        stage("Debug"){
            echo "Tag: ${env.Tag}"
            echo "Action: ${env.Action}"
            echo "else"
        }
    }
}
