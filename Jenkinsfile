#!groovy​
def gitUrl = 'https://github.com/ricardozanini/soccer-stats.git'
//def nexusUrl = 'http://nexus.local:8081/repository/ansible-meetup'
def nexusUrl = 'nexus.local:8081'

stage('Build') {
    node {
        git gitUrl
        withEnv(["PATH+MAVEN=${tool 'm3'}/bin"]) {
            def pom = readMavenPom file: 'pom.xml'
            sh "mvn -B versions:set -DnewVersion=${pom.version}-${BUILD_NUMBER}"
            sh "mvn -B -Dmaven.test.skip=true clean package"
            stash name: "artifact", includes: "target/soccer-stats-*.jar"
            stash name: "source", includes: "**", excludes: "target/"
        }
    }
}

/*
stage('Unit Tests') {
    node {
        unstash 'source'
        withEnv(["PATH+MAVEN=${tool 'm3'}/bin"]) {
            sh "mvn -B clean test"
            stash name: "unit_tests", includes: "target/surefire-reports/**"
        }
    }
}

stage('Integration Tests') {
    node {
        unstash 'source'
        withEnv(["PATH+MAVEN=${tool 'm3'}/bin"]) {
            sh "mvn -B clean verify -Dsurefire.skip=true"
            stash name: 'it_tests', includes: 'target/failsafe-reports/**'
        }
    }
}

stage('Static Analysis') {
    node {
        unstash 'source'
        withEnv(["PATH+MAVEN=${tool 'm3'}/bin"]) {
            withSonarQubeEnv('sonar'){
                unstash 'it_tests'
                unstash 'unit_tests'
                sh 'mvn sonar:sonar -DskipTests'
            }
        }
    }
}
*/

/*
stage('Artifact Upload') {
    node {
        unstash 'source'
        unstash 'artifact'

        def pom = readMavenPom file: 'pom.xml'
        def file = "${pom.artifactId}-${pom.version}"
        def jar = "target/${file}.jar"

        sh "cp pom.xml ${file}.pom"

        nexusArtifactUploader artifacts: [
                [artifactId: "${pom.artifactId}", classifier: '', file: "target/${file}.jar", type: 'jar'],
                [artifactId: "${pom.artifactId}", classifier: '', file: "${file}.pom", type: 'pom']
            ], 
            credentialsId: 'nexus', 
            groupId: "${pom.groupId}", 
            nexusUrl: "${nexusUrl}", 
            nexusVersion: 'nexus3', 
            protocol: 'http', 
            repository: 'ansible-meetup', 
            version: "${pom.version}"        
    }
}
*/

/*
stage('Approval') {
    timeout(time:3, unit:'DAYS') {
        input 'Do I have your approval for deployment?'
    }
}
*/

stage('Deploy') {
    node {
        unstash 'artifact'

        sh "ansible-galaxy install -vvv -r provision/requirements.yml -p provision/roles/"

        ansiblePlaybook colorized: true, 
        credentialsId: 'ssh-jenkins', 
        installation: 'ansible', 
        inventory: 'provision/inventory.ini', 
        playbook: 'provision/playbook.yml', 
        sudo: true,
        sudoUser: 'jenkins'
    }
}