node("hello-ops"){
    stage('Clone') {
        echo "1.Clone Stage"
        git url: "https://github.com/st22ab889/jenkins-demo.git"
        script {
            BRANCH_NAME = "master"
            build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
            if (env.BRANCH_NAME != 'master') {
                build_tag = "${env.BRANCH_NAME}-${build_tag}"
            }
        }
    }
    stage('Test') {
        echo "2.Test Stage"
    }
    stage('Build') {
        echo "3.Build Docker Image Stage"
        sh "docker build -t aaron/jenkins-demo:${build_tag} ."
    }
    stage('Push') {
        echo "4.Push Docker Image Stage"
        withCredentials([usernamePassword(credentialsId: 'docekrAuth', passwordVariable: 'dockerPassword', usernameVariable: 'dockerUser')]) {
            sh "docker login -u ${dockerUser} -p ${dockerPassword}"
            sh "docker push aaron/jenkins-demo:${build_tag}"
        }
    }
    stage('Deploy') {
        echo "6. Change YAML File Stage And Deploy Stage"
        def userInput = input(
            id: 'userInput',
            message: 'Choose a deploy environment',
            parameters: [
                [
                    $class: 'ChoiceParameterDefinition',
                    choices: "Dev\nQA\nProd",
                    name: 'Env'
                 ]
            ]
        )
        echo "This is a deploy step to ${userInput.Env}"
        sh "sed -i 's/<BUILD_TAG>/${build_tag}/' k8s-jenkins-go-demo.yaml"
        
        if (userInput.Env == "Dev") {
            input "please confirm deploy to DEV env ?"
        } else if (userInput.Env == "QA"){
            input "please confirm deploy to QA env ?"
        } else {
             input "please confirm deploy to Prod env ?"
        }
        sh "sed -i 's/<BRANCH_NAME>/${env.BRANCH_NAME}/' k8s.yaml"

        sh "kubectl apply -f k8s-jenkins-go-demo.yaml"
    }
}