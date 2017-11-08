#!/usr/bin/groovy

@Library('shared-library@release/version1')
//import com.epam.ArtifactoryToolsPlugin


String artifactoryRepo = 'bigdata-dss-automation'
String artifactoryUrl = 'http://192.168.56.105:8081'
String atfVersion = '0.1.0'
String atfRelease = 'release'
String projectName = 'sample-project'
String playbooksName = 'ci-cd-playbooks'
String playbooksVersion = '0.1'
String playbooksRelease = 'release'

String targetHostUser = 'vagrant'
String targetHost = "${targetHostUser}@192.168.56.21"
String targetGroup = "prod"
String projectArchiveName

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

    // --------------------------------------
    // This stage is added to perform project build
    stage('Create project archive') {
        echo "********* Start to create project archive **********"
        GString sourceFolder = "${WORKSPACE}"
        def zip = new ZipTools()
        projectArchiveName = zip.bundle(sourceFolder, ['.git', '.gitignore'])
        echo "created an archive '$projectArchiveName'"
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
        artifactoryTools.projectUpload(artifactoryUrl, artifactoryRepo, projectArchiveName)
        echo "********* End of upload project to Artifactory server **********"
    }

    stage('Download artifacts from Artifactory server') {
        echo "********* Start to download artifacts from Artifactory server **********"
        artifactoryTools.ansibleDownload(artifactoryUrl, artifactoryRepo, playbooksName, playbooksRelease, playbooksVersion)
        echo "********* End of download artifacts from Artifactory server **********"
    }

    stage ('Extract Ansible archive') {
        echo pipelineConfig.pad("start to extract Ansible Archive")
        def zip = new ZipTools()
        zip.extractAnsible(playbooksName, playbooksRelease, playbooksVersion)
        echo pipelineConfig.pad("Ansible playbooks extracted")
    }

    stage('Project deployment') {
        echo pipelineConfig.pad("Start project deployment")
//        sshagent([sshKeyId]) {
            pipelineConfig.runDeployProject(artifactoryUrl, artifactoryRepo, env.GIT_REPO, projectArchiveName ,targetGroup)
//        }
        echo pipelineConfig.pad("End of project deployment")
    }


    stage('ATF deploy') {
        echo pipelineConfig.pad("Start to deploy AFT project **********")
//        sshagent([sshKeyId]) {
        pipelineConfig.runDeployATF(artifactoryUrl, artifactoryRepo, atfVersion, atfRelease, projectName, targetGroup)
//        }
        echo pipelineConfig.pad("End of deploy AFT project")
    }

//    stage('smoke tests') {
//        String commandToRun = 'source /tmp/ATFVENV/bin/activate; echo $USER; ls -l $HOME/zephCred; cat $HOME/zephCred; pwd; echo $ATF_CONF_FILE; echo END'
//        sh "ssh -o StrictHostKeyChecking=no ${targetHost} /bin/bash -c '\"${commandToRun}\"'"
//    }

    stage('Project cleanup') {
        echo pipelineConfig.pad("Start project cleanup")
        pipelineConfig.runProjectCleanup(projectName, targetGroup)
        echo pipelineConfig.pad("End of project cleanup")
    }
}


