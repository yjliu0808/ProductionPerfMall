pipeline {
    agent any

    environment {
        REMOTE_HOST    = 'root@114.132.198.29'
        JMETER_HOME    = '/athena/Jmeter/apache-jmeter-5.5'
        JMETER_BASEDIR = '/athena/Jmeter'

        JMETER_SCRIPT  = "${JMETER_BASEDIR}/ProductionPerfMall.jmx"
        JMETER_OUTPUT  = "${JMETER_BASEDIR}/result.jtl"
        JMETER_REPORT  = "${JMETER_BASEDIR}/ResultHtml"
        JMETER_PLUGIN  = "${JMETER_HOME}/lib/ext/jmeter-plugins-casutg-2.9.jar"
        ALLURE_RESULTS = "${JMETER_BASEDIR}/allure-results"
    }

    triggers {
        githubPush()
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
                echo '🚀 开始执行远程 JMeter 测试脚本...'
                sh """
                    ssh -o StrictHostKeyChecking=no ${REMOTE_HOST} '
                        export JAVA_HOME=/athena/jdk/jdk1.8.0_371
                        export PATH=$JAVA_HOME/bin:$PATH

                        echo "🧹 清理旧报告..."
                        rm -rf ${JMETER_REPORT} ${ALLURE_RESULTS}
                        rm -f ${JMETER_OUTPUT}

                        echo "📊 执行 JMeter 压测（带 Allure 支持）..."
                        ${JMETER_HOME}/bin/jmeter \
                            -n -t ${JMETER_SCRIPT} \
                            -l ${JMETER_OUTPUT} \
                            -e -o ${JMETER_REPORT} \
                            -Jallure.results.directory=${ALLURE_RESULTS}
                    '
                """
            }
        }

        stage('拉取 HTML 报告') {
            steps {
                echo '📥 拉取 HTML 报告到 Jenkins 本地...'
                sh '''
                    rm -rf ResultHtml
                    scp -r -o StrictHostKeyChecking=no ${REMOTE_HOST}:${JMETER_REPORT} ./ResultHtml
                '''
            }
        }

        stage('展示 HTML 报告') {
            steps {
                publishHTML([
                    reportDir: 'ResultHtml',
                    reportFiles: 'index.html',
                    reportName: '📊 JMeter HTML 性能报告',
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true
                ])
            }
        }

        stage('拉取 Allure 报告原始数据') {
            steps {
                echo '📥 拉取 Allure 原始结果到 Jenkins 本地...'
                sh '''
                    rm -rf allure-results
                    scp -r -o StrictHostKeyChecking=no ${REMOTE_HOST}:${ALLURE_RESULTS} ./allure-results
                '''
            }
        }

        stage('展示 Allure 报告') {
            steps {
                allure includeProperties: false,
                       jdk: '',
                       results: [[path: 'allure-results']]
            }
        }

        stage('完成') {
            steps {
                echo "🎉 JMeter 性能测试完成，HTML + Allure 报告已集成 Jenkins 页面。"
            }
        }
    }
}
