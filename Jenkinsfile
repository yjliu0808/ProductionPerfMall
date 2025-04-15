pipeline {
    agent any

    triggers {
        githubPush()   // 👈 加上这一段
    }

    stages {
        stage('Triggered') {
            steps {
                echo '🎉 1Jenkins CI/CD 已被 GitHub Push 成功触发!'
            }
        }
    }
}
