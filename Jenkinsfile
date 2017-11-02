#!/usr/bin/groovy

@Library('shared-library@master')
import com.epam.ArtifactoryToolsPlugin


def pad(str) {
    return "********** ${str} ***********"
}

def createProjectBundle(sourceFolder) {
    def zip = new ZipTools()
    def bundlePath = zip.bundle(env, sourceFolder, ['.git', '.gitignore'])
    echo "created an archive $bundlePath"
}


def runATFCommand(targetHost, user, projectName, command) {
    String commandToRun = "cd /home/${user}/${projectName}; source ./ATFVENV/bin/activate; ${command}"
    sh sshCli(targetHost, commandToRun)
}

def sshCli(host, commandToRun) {
    return "ssh -o StrictHostKeyChecking=no ${host} /bin/bash -c '\"${commandToRun}\"'"
}

def executeAnsible(ansibleCommand) {
    withCredentials([usernamePassword(credentialsId: 'artifactoryIDVG', usernameVariable: 'artifactory_user', passwordVariable: 'artifactory_pwd')]) {
        dir("${WORKSPACE}/ansible") {
            sh ansibleCommand
        }
    }
}

def reportGitParams() {
    echo "Git Origin: ${env.GIT_ORIGIN}, Git User: ${env.GIT_USER}, Git Project: ${env.GIT_PROJECT}, Git Branch: ${env.GIT_BRANCH}, Git Repo: ${env.GIT_REPO}, Git Feature Name (optional): ${env.GIT_FEATURE_NAME}"
}

def ansible(command) {
    return "ansible-playbook --extra-vars 'server=dev user=artifactory_user password=artifactory_pwd ${command}"
}

def runDeployATF(String artifactoryRepo, String artifactoryUrl, String atfVersion, String projectName) {
    withCredentials([usernamePassword(credentialsId: 'artifactoryIDVG', usernameVariable: 'artifactory_user', passwordVariable: 'artifactory_pwd')]) {
        sh "cp $WORKSPACE/ci-cd-framework/requirements.txt $WORKSPACE/requirements.txt"
        withCredentials([file(credentialsId: 'atf-config', variable: 'atfConf')]) {
            dir("${WORKSPACE}/ci-cd-framework/ansible") {
                sh ansible("artifactoryRepo=${artifactoryRepo} artifactoryUrl=${artifactoryUrl} atfVersion=${atfVersion} projectName=${projectName} workspace=${WORKSPACE} atfConf=${atfConf}' ATFDeployment.yml")
            }
        }
    }
}

def runDeployProject(artifactoryUrl, artifactoryRepo, projectVersion, projectName) {
    def cmd = ansible("artifactoryUrl=${artifactoryUrl} artifactoryRepo=${artifactoryRepo} projectVersion=${projectVersion} projectName=${projectName} workspace=${WORKSPACE}' projectDeployment.yml")
    executeAnsible(cmd)
}


def runProjectCleanup(projectName) {
    cmd = ansible("projectName=${projectName}' projectCleanup.yml")
        executeAnsible(cmd)
}

// +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++


String artifactoryRepo = 'bigdata-dss-automation'
String artifactoryUrl = 'http://192.168.56.105:8081'
String atfVersion = '0.0.1'
String projectVersion = '0.1'
String projectName = 'sample-project'
String frameworkVersion = "0.1"
String frameworkName = "framework"

String targetHostUser = 'vagrant'
String targetHost = "${targetHostUser}@192.168.56.21"

node {

    echo "DEBUG CODE -----> Running ${env.JOB_NAME} on ${env.JENKINS_URL} for branch ${env.BRANCH_NAME}"

    // --------------------------------------
    // DEVELOPER NOTE: DO NOT EDIT THIS STAGE
    // CLEAN WORKSPACE STEPS
    stage('Clean Workspace') {
        echo "********** Clean Jenkins workspace ***********"
        deleteDir()
        checkout scm
    }

//    stage('Check out "cd-cd-framework" repo') {
//        echo "********* Check out 'framework' repo **********"
//        dir('framework') {
//            git branch: 'Artifactory-with-plugin', url: 'https://github.com/vgrys/VAULT.git'
//        }
//        echo "********* End of check out 'framework' repo **********"
//    }


    // --------------------------------------
    // This stage is added to perform project build
    stage('Create project archive') {
        echo "********* Start to create project archive **********"
        GString sourceFolder = "${WORKSPACE}"
        def zip = new ZipTools()
        def bundlePath = zip.bundle(env, sourceFolder, ['.git', '.gitignore'])
        echo "created an archive $bundlePath"
        echo "********* End of create project archive **********"
    }

    // --------------------------------------
    // DEVELOPER NOTE: DO NOT EDIT THIS STAGE
    // This stage is added for Jenkins to upload artifacts to Artifactory server
    stage('Upload artifacts to Artifactory server') {
        echo "********* Start to upload artifacts to Artifactory server **********"
        def atfArchivePath = null
        GString projectArchivePath = "${WORKSPACE}/*tgz"
        def artifactoryServer = Artifactory.newServer url: "${artifactoryUrl}", credentialsId: 'arifactoryID'
        def artifactory = new ArtifactoryToolsPlugin()
        artifactory.artifactoryConfig(env, artifactoryRepo, "${atfArchivePath}", "${projectArchivePath}", atfVersion, projectName, projectVersion)
        artifactoryServer.upload(env.uploadSpec)
        echo "********* End of upload artifacts to Artifactory server **********"
    }

    // --------------------------------------
    // This stage is added to download Ansible from Artifactory and extract it
    stage('Download artifacts from Artifactory server and extract it') {
        echo "********* Start to download artifacts 'Ansible playbooks' from Artifactory server **********"
        GString frameworkPath = "${WORKSPACE}/ansible/"
        GString frameworkArtifactoryPath = "${artifactoryRepo}/${frameworkName}/${frameworkVersion}/*.tgz"
        GString downloadSpec = """{"files": [{"pattern": "${frameworkArtifactoryPath}", "target": "${frameworkPath}"}]}"""
        def server = Artifactory.newServer url: "${artifactoryUrl}/artifactory/", credentialsId: 'arifactoryID'
        server.download(downloadSpec)
        echo "start to extract '${frameworkName}-${frameworkVersion}.tgz'"
        sh "tar -xzf ${frameworkPath}/${frameworkName}/${frameworkVersion}/${frameworkName}-${frameworkVersion}.tgz -C ${frameworkPath}"
        echo "********* End of download artifacts 'Ansible playbooks' from Artifactory server **********"
    }

    // --------------------------------------
    // DEVELOPER NOTE: DO NOT EDIT THIS STAGE
    // PROJECT DEPLOYMENT STAGE
    stage('Project deployment') {
        echo "********* Start project deployment **********"
        withCredentials([usernamePassword(credentialsId: 'arifactoryID', usernameVariable: 'artifactory_user', passwordVariable: 'artifactory_pwd')]) {
            dir("${WORKSPACE}/ansible") {
                sh "ansible-playbook --extra-vars 'server=prod user=artifactory_user password=artifactory_pwd artifactoryUrl=${artifactoryUrl} artifactoryRepo=${artifactoryRepo} projectVersion=${projectVersion} projectName=${projectName} workspace=${WORKSPACE}' projectDeployment.yml"
            }
        }
        echo "********* End of project deployment **********"
    }

    // --------------------------------------
    // DEVELOPER NOTE: DO NOT EDIT THIS STAGE
    // ATF DEPLOYMENT STAGE
    stage('ATF deploy') {
        echo "********* Start to deploy AFT project **********"
        withCredentials([usernamePassword(credentialsId: 'arifactoryID', usernameVariable: 'artifactory_user', passwordVariable: 'artifactory_pwd')]) {
            withCredentials([file(credentialsId: 'zeph', variable: 'zephCred')]) {
                dir("${WORKSPACE}/ansible") {
                    sh "ansible-playbook --extra-vars 'server=prod user=artifactory_user password=artifactory_pwd artifactoryRepo=${artifactoryRepo} artifactoryUrl=${artifactoryUrl} atfVersion=${atfVersion} workspace=${WORKSPACE} zephCred=${zephCred}' ATFDeployment.yml"
                }
            }
        }
        echo "********* End of deploy AFT project **********"
    }

    stage('smoke tests') {
        String commandToRun = 'source /tmp/ATFVENV/bin/activate; echo $USER; ls -l $HOME/zephCred; cat $HOME/zephCred; pwd; echo $ATF_CONF_FILE; echo END'
        sh "ssh -o StrictHostKeyChecking=no ${targetHost} /bin/bash -c '\"${commandToRun}\"'"
    }

    stage('Project cleanup') {
        echo pad("Start project cleanup")
        runProjectCleanup(projectName)
        echo pad("End of project cleanup")
    }

}
