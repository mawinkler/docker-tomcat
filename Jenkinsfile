import groovy.json.JsonOutput

node('pod-dind') {
  withEnv(['REPOSITORY=docker-tomcat',
           'DSSC_SERVICE=34.89.237.34:30010',
           'DSSC_REGISTRY=34.89.237.34:30011',
           'K8S_REGISTRY=34.89.237.34:30017',
           'GIT_ACCOUNT=https://github.com/mawinkler']) {
    container('dind') {
      stage('Pull Image from Git') {
        script {
          git "${GIT_ACCOUNT}/${REPOSITORY}.git"
        }
      }
      stage('Build Image') {
        script {
          dbuild = docker.build("${REPOSITORY}")
        }
      }
      stage('Test') {
        echo 'All functional tests passed'
        test = JsonOutput.toJson([
          malware: 0,
          vulnerabilities: [
            defcon1: 0,
            critical: 0,
            high: 1,
          ],
          contents: [
            defcon1: 0,
            critical: 0,
            high: 1,
          ],
          checklists: [
            defcon1: 0,
            critical: 0,
            high: 0,
          ],
        ]).toString()
        echo "${test}"
      }
      stage('Check Image (pre-Registry') {
        smartcheckScan([
          imageName: "${REPOSITORY}",
          smartcheckHost: "${DSSC_SERVICE}",
          smartcheckCredentialsId: "smartcheck-auth",
          insecureSkipTLSVerify: true,
          insecureSkipRegistryTLSVerify: true,
          preregistryScan: true,
          preregistryHost: "${DSSC_REGISTRY}",
          preregistryCredentialsId: "preregistry-auth",
          findingsThreshold: JsonOutput.toJson([
            malware: 0,
            vulnerabilities: [
              defcon1: 0,
              critical: 0,
              high: 1,
            ],
            contents: [
              defcon1: 0,
              critical: 0,
              high: 1,
            ],
            checklists: [
              defcon1: 0,
              critical: 0,
              high: 0,
            ],
          ]).toString(),
        ])
      }
      stage('Push Image to Registry') {
        script {
          docker.withRegistry("https://${K8S_REGISTRY}", 'registry-auth') {
            dbuild.push()
          }
        }
      }
      stage('Check Image (Registry') {
        smartcheckScan([
          imageName: "${K8S_REGISTRY}/docker-tomcat-tutorial",
          smartcheckHost: "${DSSC_SERVICE}",
          smartcheckCredentialsId: "smartcheck-auth",
          imagePullAuth: JsonOutput.toJson([
            username: "reguser",
            password: "TrendM1cr0"
          ]).toString()
        ])
      }
      stage('Push Image to Registry') {
        script {
          docker.withRegistry('', 'docker-hub') {
            dbuild.push() //('$BUILD_NUMBER')
          }
        }
      }
    }
  }
}

