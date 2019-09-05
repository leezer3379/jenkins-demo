node('k8s-slave') {
    if("${env.Action}"=="sdeploy"){
        stage('下载代码') {
            ansiColor('xterm') {
                echo '\033[32m下载代码\033[0m'
            }
            checkout scm
            script {
                build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
                ansiColor('xterm') {
                    echo '\033[32m======================================\n'+"${build_tag} sdeploying...."+'\n====================================== \033[0m'
                }
            }
        }
        stage('构建镜像') {
            ansiColor('xterm') {
                echo '\033[32m构建镜像\033[0m'
            }
            sh "docker build -t registry-vpc.cn-beijing.aliyuncs.com/sy-ops/jenkins-demo:${build_tag} ."
        }
        stage('上传镜像到阿里云') {
            ansiColor('xterm') {
                echo '\033[32m上传镜像到阿里云\033[0m'
            }
            withCredentials([usernamePassword(credentialsId: 'dockerAliyun', passwordVariable: 'dockerAliyunPassword', usernameVariable: 'dockerAliyunUser')]) {
                sh "docker login -u ${dockerAliyunUser} -p ${dockerAliyunPassword} registry-vpc.cn-beijing.aliyuncs.com"
                sh "docker push registry-vpc.cn-beijing.aliyuncs.com/sy-ops/jenkins-demo:${build_tag}"
            }
        }
        stage('部署项目到k8s') {
            ansiColor('xterm') {
                echo '\033[32m部署项目到k8s\033[0m'
            }
            sh "sed -i 's/<BUILD_TAG>/${build_tag}/g' k8s.yaml"
            sh "kubectl apply -f k8s.yaml --record"
        }
    } else if("${env.Action}"=="rollback" && Tag!="") {
        stage("回滚"){
            ansiColor('xterm') {
                echo '\033[32m回滚\033[0m'
            }
            checkout scm
            ansiColor('xterm') {
                echo '\033[32m======================================\n'+"${Tag} rollbacking...."+'\n====================================== \033[0m'
            }
            sh "sed -i 's/<BUILD_TAG>/${Tag}/g' k8s.yaml"
            sh "kubectl apply -f k8s.yaml --record"
        }
    }
}
