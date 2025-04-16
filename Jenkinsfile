pipeline {
    agent any

    parameters {
        choice(name: 'ENV', choices: ['dev', 'test', 'prod'], description: '选择压测环境')
    }

    environment {
        JMETER_HOME    = '/athena/Jmeter/apache-jmeter-5.5'
        JMETER_BASEDIR = '/athena/Jmeter'

        JMETER_SCRIPT  = "${JMETER_BASEDIR}/ProductionPerfMall.jmx"
        JMETER_OUTPUT  = "${JMETER_BASEDIR}/result.jtl"
        JMETER_REPORT  = "${JMETER_BASEDIR}/ResultHtml"
        JMETER_PLUGIN  = "${JMETER_HOME}/lib/ext/jmeter-plugins-casutg-2.9.jar"
    }

    triggers {
        githubPush()
    }

    stages {
        stage('压测流程') {
            steps {
                script {
                    def sshCredentialId = "ssh-${params.ENV}"
                    def remoteHost = "114.132.198.29" // 你也可以按 ENV 来切换 IP

                    withCredentials([sshUserPrivateKey(credentialsId: sshCredentialId, keyFileVariable: 'SSH_KEY')]) {

                        echo "✅ 当前构建环境：${params.ENV}"

                        // 上传 jmx
                        sh """
                            scp -i $SSH_KEY -o StrictHostKeyChecking=no ProductionPerfMall.jmx root@${remoteHost}:${JMETER_SCRIPT}
                        """

                        // 插件检查
                        sh """
                            ssh -i $SSH_KEY -o StrictHostKeyChecking=no root@${remoteHost} '
                                if [ ! -f "${JMETER_PLUGIN}" ]; then
                                    echo "📦 插件不存在，正在下载..."
                                    wget -O "${JMETER_PLUGIN}" https://repo1.maven.org/maven2/kg/apc/jmeter-plugins-casutg/2.9/jmeter-plugins-casutg-2.9.jar
                                    chmod 644 "${JMETER_PLUGIN}"
                                else
                                    echo "✅ 插件已存在，跳过下载。"
                                fi
                            '
                        """

                        // 执行压测
                        retry(2) {
                            sh """
                                ssh -i $SSH_KEY -o StrictHostKeyChecking=no root@${remoteHost} '
                                    export JAVA_HOME=/athena/jdk/jdk1.8.0_371
                                    export PATH=$JAVA_HOME/bin:$PATH
                                    rm -rf ${JMETER_REPORT}
                                    rm -f ${JMETER_OUTPUT}
                                    ${JMETER_HOME}/bin/jmeter -n -t ${JMETER_SCRIPT} -l ${JMETER_OUTPUT} -e -o ${JMETER_REPORT}
                                '
                            """
                        }

                        // 拉回报告
                        sh """
                            rm -rf ResultHtml
                            scp -i $SSH_KEY -r -o StrictHostKeyChecking=no root@${remoteHost}:${JMETER_REPORT} ./ResultHtml
                        """
                    }
                }
            }
        }

        stage('展示报告') {
            steps {
                publishHTML([
                    reportDir: 'ResultHtml',
                    reportFiles: 'index.html',
                    reportName: '📊 JMeter 性能测试报告',
                    alwaysLinkToLastBuild: true,
                    keepAll: true
                ])
            }
        }

        stage('完成') {
            steps {
                echo "🎉 测试完成！报告已集成到 Jenkins 页面。"
            }
        }
    }
}
