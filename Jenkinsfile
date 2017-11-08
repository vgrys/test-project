#!/usr/bin/groovy

@Library('shared-library@release/version1')
//import com.epam.ArtifactoryToolsPlugin


String artifactoryRepo = 'bigdata-dss-automation'
String artifactoryUrl = 'http://192.168.56.105:8081'
String atfVersion = '0.0.1'
String projectVersion = '0.1'
String projectName = 'sample-project'
String frameworkVersion = "0.1"
String frameworkName = "framework"

String targetHostUser = 'vagrant'
String targetHost = "${targetHostUser}@192.168.56.21"
String targetGroup = "prod"

node {

    echo "DEBUG CODE -----> Running ${env.JOB_NAME} on ${env.JENKINS_URL} for branch ${env.BRANCH_NAME}"

    // --------------------------------------
    // DEVELOPER NOTE: DO NOT EDIT THIS STAGE
    // CLEAN WORKSPACE STEPS
    stage('Clean Workspace') {
        echo "********** Clean Jenkins workspace ***********"
        deleteDir()
        checkout scm
        gitInfo()
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
        projectName = zip.bundle(sourceFolder, ['.git', '.gitignore'])
        echo "created an archive '$projectName'"
        echo "********* End of create project archive **********"
    }
//
//    // --------------------------------------
//    // DEVELOPER NOTE: DO NOT EDIT THIS STAGE
//    // This stage is added for Jenkins to upload artifacts to Artifactory server
//    stage('Upload artifacts to Artifactory server') {
//        echo "********* Start to upload artifacts to Artifactory server **********"
//        GString projectArchivePath = "${WORKSPACE}/*tgz"
//        def artifactoryServer = Artifactory.newServer url: "${artifactoryUrl}", credentialsId: 'arifactoryID'
//        def artifactory = new ArtifactoryToolsPlugin()
//        artifactory.artifactoryConfig(env, artifactoryRepo, null, "${projectArchivePath}", null, projectName, projectVersion)
//        artifactoryServer.upload(env.uploadSpec)
//        echo "********* End of upload artifacts to Artifactory server **********"
//    }

    stage('Upload project to Artifactory server') {
        echo "********* Start to upload project to Artifactory server **********"
        artifactoryTools.projectUpload(artifactoryUrl, artifactoryRepo, projectName)
        echo "********* End of upload project to Artifactory server **********"
    }

    stage('Download artifacts from Artifactory server') {
        echo "********* Start to download artifacts from Artifactory server **********"
        artifactoryTools.ansibleDownload(artifactoryUrl, artifactoryRepo, frameworkName, frameworkVersion)
        echo "********* End of download artifacts from Artifactory server **********"
    }

    stage ('Extract Ansible archive') {
        echo pipelineConfig.pad("start to extract Ansible Archive")
        pipelineConfig.extractAnsible(frameworkName, frameworkVersion)
        echo pipelineConfig.pad("Ansible playbooks extracted")
    }


//    // --------------------------------------
//    // This stage is added to download Ansible from Artifactory and extract it
//    stage('Download artifacts from Artifactory server and extract it') {
//        echo "********* Start to download artifacts 'Ansible playbooks' from Artifactory server **********"
//        GString frameworkPath = "${WORKSPACE}/ansible/"
//        GString frameworkArtifactoryPath = "${artifactoryRepo}/${frameworkName}/${frameworkVersion}/*.tgz"
//        GString downloadSpec = """{"files": [{"pattern": "${frameworkArtifactoryPath}", "target": "${frameworkPath}"}]}"""
//        def server = Artifactory.newServer url: "${artifactoryUrl}/artifactory/", credentialsId: 'arifactoryID'
//        server.download(downloadSpec)
//        echo "start to extract '${frameworkName}-${frameworkVersion}.tgz'"
//        sh "tar -xzf ${frameworkPath}/${frameworkName}/${frameworkVersion}/${frameworkName}-${frameworkVersion}.tgz -C ${frameworkPath}"
//        echo "********* End of download artifacts 'Ansible playbooks' from Artifactory server **********"
//    }
//
//    // --------------------------------------
//    // DEVELOPER NOTE: DO NOT EDIT THIS STAGE
//    // PROJECT DEPLOYMENT STAGE
//    stage('Project deployment') {
//        echo "********* Start project deployment **********"
//        withCredentials([usernamePassword(credentialsId: 'arifactoryID', usernameVariable: 'artifactory_user', passwordVariable: 'artifactory_pwd')]) {
//            dir("${WORKSPACE}/ansible") {
//                sh "ansible-playbook --extra-vars 'server=prod user=artifactory_user password=artifactory_pwd artifactoryUrl=${artifactoryUrl} artifactoryRepo=${artifactoryRepo} projectVersion=${projectVersion} projectName=${projectName} workspace=${WORKSPACE}' projectDeployment.yml"
//            }
//        }
//        echo "********* End of project deployment **********"
//    }
//
//    // --------------------------------------
//    // DEVELOPER NOTE: DO NOT EDIT THIS STAGE
//    // ATF DEPLOYMENT STAGE
//    stage('ATF deploy') {
//        echo "********* Start to deploy AFT project **********"
//        withCredentials([usernamePassword(credentialsId: 'arifactoryID', usernameVariable: 'artifactory_user', passwordVariable: 'artifactory_pwd')]) {
//            withCredentials([file(credentialsId: 'zeph', variable: 'zephCred')]) {
//                dir("${WORKSPACE}/ansible") {
//                    sh "ansible-playbook --extra-vars 'server=prod user=artifactory_user password=artifactory_pwd artifactoryRepo=${artifactoryRepo} artifactoryUrl=${artifactoryUrl} atfVersion=${atfVersion} workspace=${WORKSPACE} zephCred=${zephCred}' ATFDeployment.yml"
//                }
//            }
//        }
//        echo "********* End of deploy AFT project **********"
//    }
//
//    stage('smoke tests') {
//        String commandToRun = 'source /tmp/ATFVENV/bin/activate; echo $USER; ls -l $HOME/zephCred; cat $HOME/zephCred; pwd; echo $ATF_CONF_FILE; echo END'
//        sh "ssh -o StrictHostKeyChecking=no ${targetHost} /bin/bash -c '\"${commandToRun}\"'"
//    }

    stage('Project deployment') {
        echo pipelineConfig.pad("Start project deployment")
        sshagent([sshKeyId]) {
            pipelineConfig.runDeployProject(artifactoryUrl, artifactoryRepo, projectName)
        }
        echo pipelineConfig.pad("End of project deployment")
    }


    stage('ATF deploy') {
        echo pipelineConfig.pad("Start to deploy AFT project **********")
//        sshagent([sshKeyId]) {
        pipelineConfig.runDeployATF(artifactoryRepo, artifactoryUrl, atfVersion, projectName, targetGroup)
//        }
        echo pipelineConfig.pad("End of deploy AFT project")
    }

    stage('Project cleanup') {
        echo pipelineConfig.pad("Start project cleanup")
        pipelineConfig.runProjectCleanup(projectName, targetGroup)
        echo pipelineConfig.pad("End of project cleanup")
    }
}
