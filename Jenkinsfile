pipeline {
    agent any

    environment {
        TYK_DASH_URL = credentials('tyk-dash-url')
        TYK_ORG_ID = credentials('tyk-org-id')
        TYK_DASH_SECRET = credentials('tyk-dash-secret')
    }

    stages {
        // stage('test') {
        //     steps {
        //         script {
        //             def apiDefs = findFiles(glob: 'api-*.json')

        //             apiDefs.each { apiDef ->
        //                 def pJson = readJSON file: env.WORKSPACE + "/" + apiDef.name
        //                 println "Testing ${pJson.name}: ${pJson.api_id}"

        //                 // println "ensuring api is authenticated"
        //                 // assertAuthenticated(pJson)

        //                 // println "ensuring api has appropriate tags"
        //                 // assertWhitelistedTag(pJson)
        //             }
        //         }
        //     }
        // }
        stage('deploy') {
            when {
                expression { env.BRANCH_NAME == 'master' }
            }
            steps {
                echo "Deploying, because we are on ${env.BRANCH_NAME}"
                sh "wget https://github.com/davegarvey/tyk-jenkins-cicd-sync/releases/download/1/tyk-sync-1.0.0"
                sh "chmod +x tyk-sync-1.0.0"
                sh "./tyk-sync-1.0.0 sync -d ${env.TYK_DASH_URL} -s ${env.TYK_DASH_SECRET} -o ${env.TYK_ORG_ID} -p ."
            }
        }
    }

    post {
        always {
            deleteDir()
        }
    }
}

def static assertAuthenticated(api) {
    assert api.use_keyless == false : "api ${api.name} should have authentication enabled"
}

def static assertWhitelistedTag(api) {
    assert api.tags.size() > 0 : "api ${api.name}  should be tagged either internal, external or both"

    api.tags.each { tag ->
        assert ["internal", "external"].contains(tag) : "api ${api.name} contains unknown tag ${tag}"
    }
}
