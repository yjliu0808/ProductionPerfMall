pipeline {
    agent any

    environment {
        REMOTE_HOST    = 'root@114.132.198.29'                      // 远程服务器
        JMETER_HOME    = '/athena/Jmeter/apache-jmeter-5.5'         // JMeter 安装路径
        JMETER_BASEDIR = '/athena/Jmeter'                           // 所有测试资源存放目录

        JMETER_SCRIPT  = "${JMETER_BASEDIR}/ProductionPerfMall.jmx"      // 脚本路径
        JMETER_OUTPUT  = "${JMETER_BASEDIR}/result.jtl"                  // 结果文件路径
        JMETER_REPORT  = "${JMETER_BASEDIR}/ResultHtml"                 // HTML 报告输出路径
        JMETER_PLUGIN  = "${JMETER_HOME}/lib/ext/jmeter-plugins-casutg-2.9.jar"  // 插件路径
    }

    triggers {
        githubPush()  // GitHub push 自动触发构建
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
                sh '''
                    ssh -o StrictHostKeyChecking=no ${REMOTE_HOST} '
                        if [ ! -f "${JMETER_PLUGIN}" ]; then
                            echo "📦 插件不存在，正在下载..."
                            wget -O "${JMETER_PLUGIN}" https://repo1.maven.org/maven2/kg/apc/jmeter-plugins-casutg/2.9/jmeter-plugins-casutg-2.9.jar
                            chmod 644 "${JMETER_PLUGIN}"
                        else
                            echo "✅ 插件已存在，跳过下载。"
                        fi
                    '
                '''
            }
        }

        stage('执行 JMeter 压测') {
            steps {
                echo '🚀 开始执行远程 JMeter 测试脚本...'
                sh '''
                    ssh -o StrictHostKeyChecking=no ${REMOTE_HOST} '
                        export JAVA_HOME=/athena/jdk/jdk1.8.0_371
                        export PATH=$JAVA_HOME/bin:$PATH
                        
                        # 清理旧数据
                        rm -rf ${JMETER_REPORT}
                        rm -f  ${JMETER_OUTPUT}

                        # 执行压测
                        ${JMETER_HOME}/bin/jmeter \
                            -n -t ${JMETER_SCRIPT} \
                            -l ${JMETER_OUTPUT} \
                            -e -o ${JMETER_REPORT}
                    '
                '''
            }
        }

        stage('完成') {
            steps {
                echo '🎉 JMeter 性能测试执行完成！报告已生成于远程：${JMETER_REPORT}'
            }
        }
    }
}
