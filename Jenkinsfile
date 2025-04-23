pipeline {
    agent any

    environment {
        // Визначення глобальних змінних середовища
        CONFIG_FILE_ID = 'azure_config'
        ACCESS_LOG = '/var/log/apache2/access.log'
        ERROR_LOG = '/var/log/apache2/error.log'
        ACCESS_TMP = '/tmp/access_errors.log'
        ERROR_TMP  = '/tmp/errorlog_recent.log'
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

        stage('Check access.log for 4xx/5xx errors') {
           steps {
               sh '''
               START=$(date -d '2 hours ago' '+%d/%b/%Y:%H:%M:%S')
               END=$(date '+%d/%b/%Y:%H:%M:%S')

               awk -v start="$START" -v end="$END" '
                   $4 ~ /^\\[/ {
                       gsub(/\\[|\\]/, "", $4);
                       if ($4 >= start && $4 <= end && $9 ~ /^[45][0-9][0-9]$/)
                           print
                   }
               ' "$ACCESS_LOG" > "$ACCESS_TMP"

               if [ -s "$ACCESS_TMP" ]; then
                   echo "Found 4xx/5xx errors in access.log:"
                   cat "$ACCESS_TMP"
               else
                   echo "No errors found in access.log"
               fi
               '''
           }
       }

       stage('Check error.log for any logs') {
           steps {
               sh '''
               START=$(date -d '2 hours ago' '+%Y-%m-%d %H:%M:%S')
               END=$(date '+%Y-%m-%d %H:%M:%S')

               awk -v start="$START" -v end="$END" '
                   {
                       datetime = substr($0, 2, 19);
                       gsub("T", " ", datetime);
                       if (datetime >= start && datetime <= end)
                           print
                   }
               ' "$ERROR_LOG" > "$ERROR_TMP"

               if [ -s "$ERROR_TMP" ]; then
                   echo "Recent errors in error.log:"
                   cat "$ERROR_TMP"
               else
                   echo "No entries found in error.log for the last 2 hours"
               fi
               '''
           }
       }
    }

     post {
        always {
            sh '''
            echo "Cleaning up temp files..."
            rm -f "$ACCESS_TMP" "$ERROR_TMP"
            '''
        }
    }
}
