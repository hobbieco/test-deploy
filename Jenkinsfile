#!groovy

import org.jenkinsci.plugins.pipeline.modeldefinition.Utils

// chart list
def CHART = [
    "test-chart1",
    "test-chart2"
]

// function: image search, pull and push
def search(CODE_GIT_SUBFOLDER, CHART, PREFIX, SRC_PREFIX) {
    sh """
        cd ${CODE_GIT_SUBFOLDER}
        rm -f chartlist 
        touch chartlist
    """

    CHART.each { ITEM ->
        sh """
            cd ${CODE_GIT_SUBFOLDER}
            helm template ${ITEM} | yq -r '.spec | .template | .spec | .containers | .[] .image' >> chartlist
        """
    }
    // helm template ${ITEM} | yq -r -o=json | jq -r '.spec | .template | .spec | .containers | .[] .image' >> chartlist
    def CHECK_LIST = sh (
        script: """
            cd ${CODE_GIT_SUBFOLDER}
            sort -u chartlist > imagelist
            cat imagelist
        """,
        returnStdout: true
    ).trim()
    
    println("----------")
    println(PREFIX)
    println(CHECK_LIST)
    println(PREFIX)
    println("----------")
    
//    sh """
//        aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin ${ECR_URL}
//    """

    CHECK_LIST.split('\n').each { LINE ->
        if (LINE?.trim()) {
            def (NAME, TAG) = "${LINE}".split(':')
            if (NAME?.trim() && TAG?.trim()) {
                def SRC_NAME = "${NAME}".replaceAll("^${PREFIX}/","${SRC_PREFIX}/")
                println("name : ${NAME} / tag : ${TAG} / before : ${SRC_NAME}")
//                sh """
//                    META = "\$( aws ecr describe-images --repository-name=${NAME} --image-ids=imageTag=${TAG} 2> /dev/null )"
//                    if [[ \$? == 0 ]]; then
//                        echo "${NAME}:${TAG} found - skip"
//                    else
//                        echo "${NAME}:${TAG} not found"
//                        docker pull ${ECR_URL}/${SRC_NAME}:${TAG}
//                        docker tag ${ECR_URL}/${SRC_NAME}:${TAG} ${ECR_URL}/${NAME}:${TAG}
//                        docker push ${ECR_URL}/${NAME}:${TAG}
//                    fi
//                """
                def CHECK = sh (
                    script: """
                        rm test
                    """,
                    returnStatus: true
                )
                println("##################### ${CHECK}")
                if (CHECK == 0) {
                    println("---------------------- exist")
                } else {
                    println("---------------------- not exist")
                }
            }
        }
    }
}

node {
    
    def IS_STAGE = true
    def IS_PROD = true

    // Code GIt
    def CODE_GIT_SOURCE_REPO_URL = "ssh://git@puttico.asuscomm.com:8345/test/test-project07.git"
    def CODE_GIT_CREDENTIAL_ID = "git-tester01-ssh" // source repo ssh key credential
    def CODE_GIT_SUBFOLDER = "code" // soucrce repo checkout sub-directory
    def CODE_GIT_TARGET_BRANCH = "main"

    // Build
    def BUILD_HOME = "${WORKSPACE}/${CODE_GIT_SUBFOLDER}" // build home directory

    // ECR
    def AWS_ID = "148758220912"
    def ECR_URL = "${AWS_ID}.dkr.ecr.ap-northeast-2.amazonaws.com"


    
    // Code Checkout
    stage('Checkout') {
        dir (CODE_GIT_SUBFOLDER) {
            codeGit = git(branch: CODE_GIT_TARGET_BRANCH, credentialsId: CODE_GIT_CREDENTIAL_ID, url: CODE_GIT_SOURCE_REPO_URL)
        }
    }

    // STAGE - IMAGE CHECK
    stage('STAGE - IMAGE CHECK') {
        if (IS_STAGE) {

            search(CODE_GIT_SUBFOLDER, CHART, 'stg', 'dev')
            
            println("STAGE - ECR repo checked")
        } else {
            println("STAGE - ECR repo check passed")
            // stage skip
            Utils.markStageSkippedForConditional('STAGE - IMAGE CHECK')
        }
    }

    // PROD - IMAGE CHECK
    stage('PROD - IMAGE CHECK') {
        if (IS_PROD) {

            search(CODE_GIT_SUBFOLDER, CHART, 'stg', 'prd')
            
            sh """
                cd ${CODE_GIT_SUBFOLDER}
                while read p; do
                    echo \$p
                    IMAGE_NAME=\$(echo \$p | awk -F':' '{print \$1}')
                    IMAGE_TAG=\$(echo \$p | awk -F':' '{print \$2}')
                    BF_IMAGE_NAME=\$(echo \$IMAGE_NAME | sed 's/^stg/dev/')
                    echo "NAME" \$IMAGE_NAME " / TAG" \$IMAGE_TAG " / BF" \$BF_IMAGE_NAME
                    echo "----------"
                    echo ""
                done < chartlist
            """
//            sh """
//                # sed 's/^prod/stg/' filename # view only
//                # sed -i 's/^prod/stg/' filename # replace file
//
//                IMAGE_META = "$( aws ecr describe-images --repository-name=${IMAGE_NAME} --image-ids=imageTag=${IMAGE_TAG} 2> /dev/null )"
//                if [[ $? == 0 ]]; then
//                    echo "${IMAGE_NAME}:${IMAGE_TAG} found - skip"
//                else
//                    echo "${IMAGE_NAME}:${IMAGE_TAG} not found"
//                    docker pull ${ECR_URL}/${BF_IMAGE_NAME}:${IMAGE_TAG}
//                    docker tag ${ECR_URL}/${BF_IMAGE_NAME}:${IMAGE_TAG} ${ECR_URL}/${IMAGE_NAME}:${IMAGE_TAG}
//                    docker push ${ECR_URL}/${IMAGE_NAME}:${IMAGE_TAG}
//                fi
//            """
            println("PROD - ECR repo checked")
        } else {
            println("PROD - ECR repo check passed")
            // stage skip
            Utils.markStageSkippedForConditional('PROD - IMAGE CHECK')
        }
    }
    cleanWs()
}
