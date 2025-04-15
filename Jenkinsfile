pipeline {
    agent any

    triggers {
        githubPush() // GitHub push 自动触发
    }

    stages {
        stage('拉取代码') {
            steps {
                echo '✅ 代码已拉取成功！'
            }
        }

        stage('上传 JMeter 脚本') {
            steps {
                echo '📤 上传 JMeter 脚本到远程服务器...'
                sh '''
                    scp -o StrictHostKeyChecking=no ProductionPerfMall.jmx root@114.132.198.29:/athena/testjmeter001/
                '''
            }
        }

        stage('执行 JMeter 压测') {
            steps {
                echo '🚀 开始远程执行 JMeter 测试脚本...'
                sh '''
                    ssh -o StrictHostKeyChecking=no root@114.132.198.29 '
                        export JAVA_HOME=/athena/jdk/jdk1.8.0_371
                        export PATH=$JAVA_HOME/bin:$PATH
                        rm -rf /athena/testjmeter001/report_test &&
                        /athena/Jmeter/apache-jmeter-5.5/bin/jmeter \
                        -n \
                        -t /athena/testjmeter001/ProductionPerfMall.jmx \
                        -l /athena/testjmeter001/result.jtl \
                        -e -o /athena/testjmeter001/report_test
                    '
                '''
            }
        }

        stage('完成') {
            steps {
                echo '🎉 JMeter 性能测试执行完成！请查看远程报告路径。'
            }
        }
    }
}
