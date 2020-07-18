node('ubuntu-vm') {
   def mvnHome
   stage('Preparation') { // for display purposes
      // Get some code from a GitHub repository
      git url: 'http://10.39.5.74:8000/root/spring-petclinic', credentialsId: 'GitLab-root'
      // Get the Maven tool.
      // ** NOTE: This 'mvn-ubuntu-vm' Maven tool must be configured
      // **       in the global configuration.
      mvnHome = tool 'mvn-ubuntu-vm'
   }
   stage('Build') {
        sh "'${mvnHome}/bin/mvn' clean package findbugs:findbugs"
   }
   stage('Results') {
      junit '**/target/surefire-reports/TEST-*.xml'
      findbugs canComputeNew: false, defaultEncoding: '', excludePattern: '', healthy: '', includePattern: '', pattern: 'target/findbugsXml.xml', unHealthy: ''
      archive 'target/*.jar'
      dir('src/test/jmeter') {
          stash includes: 'petclinic_test_plan.jmx', name: 'performance'
      }
   }
}

node('master') {
    stage('Deployment') {
        build job: 'Petclinic Deployment', parameters: [string(name: 'ARTIFACT_PROJECT', value: 'Petclinic Pipeline')]
        build job: 'HealthCheck', parameters: [string(name: 'HOST', value: 'http://10.39.5.74:8080')]
    }
    stage('Test'){
        parallel(Frontend:{
            git url: 'http://10.39.5.74:8000/root/geb-tests-petclinic', credentialsId: 'GitLab-root'
            bat 'gradlew.bat chromeTest'
            junit 'build/test-results/chromeTest/*.xml'
        }, Performance: {
            unstash 'performance'
            bzt 'petclinic_test_plan.jmx'
        })
    }
}

stage('Production') {
    // Benutzereingabe
    input 'Ready for PRODUCTION?'
    // ältere Builds können nicht überholen
    milestone()
    // nur ein Build gleichzeitig
    lock('PROD') {
        node {
            echo 'Deploying master branch!'
        }
    }
}
