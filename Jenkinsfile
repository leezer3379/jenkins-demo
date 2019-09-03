node('k8s-slave') {
    if("${env.Action}"=="sdeploy"){
        stage('Prepare') {
            echo "1.Prepare Stage"
            checkout scm
            script {
                build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
                echo "${build_tag} sdeploying...."
                echo "\033[32m[ tomcat started success on ${build_tag} !  ]\033[0m"
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
            sh "sed -i 's/<BUILD_TAG>/${build_tag}/g' k8s.yaml"
            sh "kubectl apply -f k8s.yaml --record"
        }
    } else if("${env.Action}"=="rollback" && Tag!="") {
        stage("Rollback"){
            checkout scm
            echo "${Tag} rollbacking...."
            sh "sed -i 's/<BUILD_TAG>/${Tag}/g' k8s.yaml"
            sh "kubectl apply -f k8s.yaml --record"
        }
    }
}
