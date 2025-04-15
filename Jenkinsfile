pipeline {
    agent any

    triggers {
        githubPush()  // GitHub Push 触1发
    }

    stages {
        stage('Run JMeter on Remote') {
            steps {
                script {
                    // SSH 命令远程执行 JMeter
                    sh """
                        ssh root@114.132.198.29 '
                            /athena/Jmeter/apache-jmeter-5.5/bin/jmeter \
                            -n \
                            -t /athena/testjmeter001/ProductionPerfMall.jmx \
                            -l /athena/testjmeter001/result.jtl \
                            -e -o /athena/testjmeter001/report_test
                        '
                    """
                }
            }
        }
    }
}
