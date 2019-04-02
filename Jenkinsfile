pipeline {
    agent any
    environment {
        // Endevor Details
        ENDEVOR_CONNECTION="--port 7080 --protocol http --reject-unauthorized false"
        ENDEVOR_LOCATION="--instance ENDEVOR --env DEV --sys MARBLES --sub MARBLES --ccid JENK --comment JENK_PDE"
        ENDEVOR="$ENDEVOR_CONNECTION $ENDEVOR_LOCATION"

        //ZOWE_OPT_HOSTNAME=credentials('eosHost')

        // z/OSMF Connection Details
        ZOWE_OPT_HOST=credentials('eosHost')
        ALTHOST=credentials('altHost')
        ZOWE_OPT_PORT="449"
        ZOWE_OPT_REJECT_UNAUTHORIZED=false

        // File Master Plus Connection Details
        FMP="--port 5194 --protocol http"

        // CICS Connection Details
        CICS="--port 10015 --region-name CICS510A --host $ALTHOST" 
    }
    stages {
        stage('build') {
            steps {
                bat 'node --version'
                bat 'npm --version'
                bat 'bright --version'
                bat 'npm config set @brightside:registry https://api.bintray.com/npm/ca/brightside'

                bat 'bright plugins install @brightside/endevor@lts-incremental'
                bat 'bright plugins install @brightside/filemasterplus@2.0.2'
                bat 'bright plugins install @brightside/ops@lts-incremental'

                bat 'bright plugins install @brightside/cics@lts-incremental'
                bat 'bright plugins install @brightside/db2@lts-incremental'

                bat 'bright plugins install @brightside/zos-extended-files@lts-incremental'
                bat 'bright plugins install @brightside/zos-extended-jobs@lts-incremental'
                bat 'bright plugins list'
                bat 'npm install gulp-cli -g'
                bat 'npm install'
                //ZOWE_OPT_USERNAME & ZOWE_OPT_PASSWORD are used to interact with Endevor 
                withCredentials([usernamePassword(credentialsId: 'eosCreds', usernameVariable: 'ZOWE_OPT_USER', passwordVariable: 'ZOWE_OPT_PASSWORD')]) {
                bat 'gulp build'  
                }
            }
        }

      stage('deploy') {
           steps {
                //ZOWE_OPT_USER & ZOWE_OPT_PASSWORD are used to interact with z/OSMF and CICS
              withCredentials([usernamePassword(credentialsId: 'eosCreds', usernameVariable: 'ZOWE_OPT_USER', passwordVariable: 'ZOWE_OPT_PASSWORD')]) {
                    //ZOWE_OPT_PASS is used by FMP plugin
                   withCredentials([usernamePassword(credentialsId: 'eosCreds', usernameVariable: 'ZOWE_OPT_USER', passwordVariable: 'ZOWE_OPT_PASS')]) {
                      bat 'gulp deploy'
                   }
               }
            }
        }
       stage('test') {
           environment {
               ZOWE_OPT_HOST=credentials('altHost')
           }
            steps {
                //ZOWE_OPT_USER & ZOWE_OPT_PASS are used to interact with z/OSMF
                withCredentials([usernamePassword(credentialsId: 'eosCreds', usernameVariable: 'ZOWE_OPT_USER', passwordVariable: 'ZOWE_OPT_PASSWORD')]) {
                    bat 'npm test' 
                }
            }
        }
    }
}
