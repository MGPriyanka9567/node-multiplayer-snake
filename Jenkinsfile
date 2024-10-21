node ('Ubuntu-app-agent') {  
    def app

    stage('Cloning Git') {
        /* Let's make sure we have the repository cloned to our workspace */
        checkout scm
    }  

    stage('SAST') {
        build 'SECURITY-SAST-SNYK'
    }

    stage('Build-and-Tag') {
        /* This builds the actual image; synonymous to
         * docker build on the command line */
        app = docker.build("mgpriyanka028/snake")
    }

    stage('Post-to-dockerhub') {
        docker.withRegistry('https://registry.hub.docker.com', 'training_creds') {
            app.push("latest")
        }
    }

    /* Commented out stage for security image scanner
    stage('SECURITY-IMAGE-SCANNER') {
        build 'SECURITY-IMAGE-SCANNER-AQUAMICROSCANNER'
    }*/

    stage('Pull-image-server') {
        sh "docker-compose down"
        sh "docker-compose up -d"	
    }

    stage('DAST') {
        script {
            // Set up ZAP configurations
            def zapHome = '/path/to/zap' // Adjust to your actual ZAP installation path
            def zapHost = '10.0.2.15'
            def zapPort = '8090'
            def sessionFilename = 'Session_7'
            def targetUrl = 'http://10.0.2.15'

            // Start ZAP with specified configurations
            zapStart zapHome: zapHome, host: zapHost, port: zapPort, sessionFilename: sessionFilename

            // Run the ZAP scan
            zapScan targetUrl: targetUrl

            // Generate reports
            zapReport reportName: 'JENKINS_ZAP_VULNERABILITY_REPORT'

            // Archive and publish reports
            archiveArtifacts artifacts: "${env.WORKSPACE}/reports/JENKINS_ZAP_VULNERABILITY_REPORT.html"
            publishHTML target: [
                reportName: 'ZAP Report',
                reportDir: "${env.WORKSPACE}/reports",
                reportFiles: 'JENKINS_ZAP_VULNERABILITY_REPORT.html',
                alwaysLinkToLastBuild: true,
                keepAll: true
            ]
        }
    }
}
