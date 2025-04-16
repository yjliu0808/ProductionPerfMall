pipeline {
    agent any

    parameters {
        choice(name: 'ENV', choices: ['dev', 'test', 'prod'], description: '选择压测环境')
    }

    environment {
        // 按环境选择远程服务器
        REMOTE_HOST = credentials("ssh-${params.ENV}") // 建议配置 ssh 凭据id：ssh-dev, ssh-test...
        JMETER_HOME = '/athena/Jmeter/apache-jmeter-5.5'
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
        stage('拉取代码') {
            steps {
                echo "✅ 开始执行 ENV=${params.ENV} 的压测流程"
                echo '✅ 代码已拉取成功！'
            }
        }

        stage('上传 JMeter 脚本') {
            steps {
                echo '📤 上传 JMeter 脚本到远程服务器...'
                sh '''
                    scp -o StrictHostKeyChecking=no ProductionPerfMall.jmx ${REMOTE_HOST}:${JMETER_SCRIPT}
                '''
            }
        }

        stage('检查并安装 JMeter 插件') {
            steps {
                echo '🔍 检查 Stepping Thread Group 插件...'
                sh """
                    ssh -o StrictHostKeyChecking=no ${REMOTE_HOST} '
                        if [ ! -f "${JMETER_PLUGIN}" ]; then
                            echo "📦 插件不存在，正在下载..."
                            wget -O "${JMETER_PLUGIN}" https://repo1.maven.org/maven2/kg/apc/jmeter-plugins-casutg/2.9/jmeter-plugins-casutg-2.9.jar
                            chmod 644 "${JMETER_PLUGIN}"
                        else
                            echo "✅ 插件已存在，跳过下载。"
                        fi
                    '
                """
            }
        }

        stage('执行 JMeter 压测') {
            steps {
                script {
                    retry(2) {
                        echo '🚀 执行远程 JMeter 测试脚本...'
                        sh """
                            ssh -o StrictHostKeyChecking=no ${REMOTE_HOST} '
                                export JAVA_HOME=/athena/jdk/jdk1.8.0_371
                                export PATH=$JAVA_HOME/bin:$PATH

                                echo "🧹 清理旧文件..."
                                rm -rf ${JMETER_REPORT}
                                rm -f  ${JMETER_OUTPUT}

                                echo "📊 执行 JMeter 压测..."
                                ${JMETER_HOME}/bin/jmeter \
                                    -n -t ${JMETER_SCRIPT} \
                                    -l ${JMETER_OUTPUT} \
                                    -e -o ${JMETER_REPORT}
                            '
                        """
                    }
                }
            }
        }

        stage('拉取 HTML 报告') {
            steps {
                echo '📥 拉取 HTML 压测报告到 Jenkins 本地...'
                sh '''
                    rm -rf ResultHtml
                    scp -r -o StrictHostKeyChecking=no ${REMOTE_HOST}:${JMETER_REPORT} ./ResultHtml
                '''
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
                echo "🎉 测试完成！报告已集成到 Jenkins 构建页面。"
            }
        }
    }
}
