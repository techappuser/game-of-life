#!groovy

docker.image('cloudbees/java-build-tools:0.0.6').inside {

    checkout scm

    def mavenSettingsFile = "${pwd()}/.m2/settings.xml"

    stage 'Build Web App'
    wrap([$class: 'ConfigFileBuildWrapper',
        managedFiles: [[fileId: 'maven-settings-for-gameoflife', targetLocation: "${mavenSettingsFile}"]]]) {

        sh "mvn -s ${mavenSettingsFile} clean source:jar javadoc:javadoc checkstyle:checkstyle pmd:pmd findbugs:findbugs package"

        step([$class: 'ArtifactArchiver', artifacts: 'gameoflife-web/target/*.war'])
        step([$class: 'WarningsPublisher', consoleParsers: [[parserName: 'Maven']]])
        step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
        step([$class: 'JavadocArchiver', javadocDir: 'gameoflife-core/target/site/apidocs/'])

        // Use fully qualified hudson.plugins.checkstyle.CheckStylePublisher if JSLint Publisher Plugin or JSHint Publisher Plugin is installed
        step([$class: 'hudson.plugins.checkstyle.CheckStylePublisher', pattern: '**/target/checkstyle-result.xml'])
        // In real life, PMD and Findbugs are unlikely to be used simultaneously
        step([$class: 'PmdPublisher', pattern: '**/target/pmd.xml'])
        step([$class: 'FindBugsPublisher', pattern: '**/findbugsXml.xml'])
        step([$class: 'AnalysisPublisher'])
    }

    stage name:'Deploy Web App To AWS Beanstalk', concurrency: 1

    wrap([$class: 'AmazonAwsCliBuildWrapper', credentialsId: 'aws-beanstalk-credentials', defaultRegion: 'us-east-1']) {

        // DEPLOY TO BEANSTALK
        def destinationWarFile = "gameoflife-${env.BUILD_NUMBER}.war"
        def versionLabel = "game-of-life#${env.BUILD_NUMBER}"
        def description = "${env.BUILD_URL}"
        sh """\
           aws s3 cp gameoflife-web/target/gameoflife.war s3://cloudbees-apps/$destinationWarFile
           aws elasticbeanstalk create-application-version --source-bundle S3Bucket=cloudbees-apps,S3Key=$destinationWarFile --application-name game-of-life --version-label $versionLabel --description \\\"$description\\\"
           aws elasticbeanstalk update-environment --environment-name game-of-life-qa --application-name game-of-life --version-label $versionLabel --description \\\"$description\\\"
         """
        sleep 10L // wait for beanstalk to update the HealthStatus

        // WAIT FOR BEANSTALK DEPLOYMENT
        timeout(time: 5, unit: 'MINUTES') {
            waitUntil {
                sh "aws elasticbeanstalk describe-environment-health --environment-name game-of-life-qa --attribute-names All > .beanstalk-status.json"

                // parse `describe-environment-health` output
                def beanstalkStatusAsJson = readFile(".beanstalk-status.json")
                def beanstalkStatus = new groovy.json.JsonSlurper().parseText(beanstalkStatusAsJson)
                println "$beanstalkStatus"
                return beanstalkStatus.HealthStatus == "Ok" && beanstalkStatus.Status == "Ready"
            }
        }
    }

    stash name: 'acceptance-tests', includes: 'gameoflife-acceptance-tests/,gameoflife-web/target/gameoflife.war'
}

stage 'Test Web App with Selenium'
mail body: "Start web browser tests on http://game-of-life-qa.elasticbeanstalk.com/ ?", subject: "Start web browser tests on http://game-of-life-qa.elasticbeanstalk.com/ ?", to: 'cleclerc@cloudbees.com'
input "Start web browser tests on http://game-of-life-qa.elasticbeanstalk.com/ ?"

checkpoint 'Web Browser Tests'

node {
    unstash 'acceptance-tests'

    // web browser tests are fragile, test up to 3 times
    retry(3) {
        docker.image('cloudbees/java-build-tools:0.0.6').inside {
            def mavenSettingsFile = "${pwd()}/.m2/settings.xml"

            wrap([$class: 'ConfigFileBuildWrapper',
                managedFiles: [[fileId: 'maven-settings-for-gameoflife', targetLocation: "${mavenSettingsFile}"]]]) {

                sh """\
                   # debug info
                   # curl http://game-of-life-qa.elasticbeanstalk.com/
                   # curl -v http://localhost:4444/wd/hub

                   cd gameoflife-acceptance-tests
                   mvn -B -V -s ${mavenSettingsFile} verify -Dwebdriver.driver=remote -Dwebdriver.remote.url=http://localhost:4444/wd/hub -Dwebdriver.base.url=http://game-of-life-qa.elasticbeanstalk.com/
                """
            }
        }
    }
}