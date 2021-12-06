pipeline {
    agent none
    environment {
        POM_XML_VERSION = ""
        BRANCH_DEFAULT = "master"
        POM_XML_FILE = "pom.xml"
        ARTIFACT_ID = ""
        ARTIFACT_NAME = ""
        JAR_NAME = ""
        //NOVA_VERSAO = false
    }
    stages {
        stage ('Build') {
            agent any
            steps {
                script {
                    echo 'Read pom file'
                    pom = readMavenPom file: "$POM_XML_FILE"

                    POM_XML_VERSION = "${pom.version}"
                    ARTIFACT_ID = "${pom.artifactId}"
                    ARTIFACT_NAME = "${ARTIFACT_ID}-${POM_XML_VERSION}"

                    //$NOVA_VERSAO = $NOVA_VERSAO
                    /*
                    novaVersao = ${NOVA_VERSAO}
                    if(novaVersao == "true"){
                        echo 'SIM'
                    } else {
                        echo 'NAO'
                    }
                    */
                }
            }
        }
        stage ('Release Candidate') {
            agent any
            environment {
                GITHUB_CREDENTIALS = credentials('github')
            }
            steps {
                script {
                    properties([
                        parameters([
                            booleanParam(
                                defaultValue: true,
                                description: '',
                                name: 'NOVA_VER_123'
                            )
                        ])
                    ])
                    echo 'Read pom file'
                    pom = readMavenPom file: "$POM_XML_FILE"

                    sh name: 'Checkout Git master branch', 
                    script: "git checkout -b ${BRANCH_DEFAULT} remotes/origin/${BRANCH_DEFAULT} || true"

                    sh name: 'Set remote origin url',
                    script: "git config remote.origin.url https://'${GITHUB_CREDENTIALS_USR}:${GITHUB_CREDENTIALS_PSW}'@github.com/${GITHUB_CREDENTIALS_USR}/${ARTIFACT_ID}.git"
                    
                    if(params.NOVA_VER_123 == true){
                        echo 'SIM'
                    } else {
                        echo 'NAO'
                    }
                    
                    sh name: "Create local Git tag for ${POM_XML_VERSION}",
                    script: "git tag -a '${POM_XML_VERSION}' -m \"tag ${POM_XML_VERSION} gerada\""

                    sh name: "Push local tag to Bitbucket",
                    script: "git push origin '${pom.version}'"

                    def version = pom.version.toString().split("\\.")
                    version[0] = version[0].toInteger()+1
                    pom.version = version.join('.')
                    echo "Incremented POM project version to ${pom.version}"

                    writeMavenPom model: pom, file: "${POM_XML_FILE}", name: 'Write Maven POM file'

                    sh name: "Add POM file to Git",
                    script: "git add ${POM_XML_FILE}"

                    sh name: "Commit POM file to Git",
                    script: 'git commit -m "POM version increased"'

                    echo 'Push local changes to GitHub'
                    sh name: '', script: "git push origin ${BRANCH_DEFAULT}"
                }
            }
        }
    }
}