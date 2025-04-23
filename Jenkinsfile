pipeline {
    agent any

    environment {
        // Визначення глобальних змінних середовища
        CONFIG_FILE_ID = 'azure_config'
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Load Configuration') {
            steps {
                configFileProvider([configFile(fileId: CONFIG_FILE_ID, variable: 'CONFIG_FILE')]) {
                    script {
                        // Завантаження змінних з конфіг файлу
                        def props = load CONFIG_FILE
                        env.AZURE_VM_USER = sh(script: "grep AZURE_VM_USER ${CONFIG_FILE} | cut -d'=' -f2", returnStdout: true).trim()
                        env.AZURE_VM_HOST = sh(script: "grep AZURE_VM_HOST ${CONFIG_FILE} | cut -d'=' -f2", returnStdout: true).trim()
                    }
                }
            }
        }

        stage('Fetch Logs') {
            steps {
                sshagent(['server_key']) {
                    script {
                        sh """
                            ssh ${AZURE_VM_USER}@${AZURE_VM_HOST} '
                                find /var/log/apache2/ -name "access.log" -type f -mmin -120 | xargs tail -n 1000
                            '
                        """
                    }
                }
            }
        }
    }
}
