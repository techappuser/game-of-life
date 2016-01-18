#!groovy

docker.image('cloudbees/java-build-tools:0.0.6').inside {

    checkout scm
    def mavenSettingsFile = "${pwd()}/.m2/settings.xml"

    stage 'Build Web App'
    wrap([$class: 'ConfigFileBuildWrapper',
        managedFiles: [[fileId: 'maven-settings-for-gameoflife', targetLocation: "${mavenSettingsFile}"]]]) {

        sh "mvn -s ${mavenSettingsFile} clean source:jar javadoc:javadoc checkstyle:checkstyle pmd:pmd findbugs:findbugs package"

        step([$class: 'WarningsPublisher', consoleParsers: [[parserName: 'Maven']]])
        step([$class: 'JUnitResultArchiver', testResults: 'gameoflife-core/target/surefire-reports/*.xml'])
        step([$class: 'JavadocArchiver', javadocDir: 'gameoflife-core/target/site/apidocs/', keepAll: false])

        // Use hudson.plugins.checkstyle.CheckStylePublisher if JSLint Publisher Plugin or JSHint Publisher Plugin is installed
        step([$class: 'hudson.plugins.checkstyle.CheckStylePublisher', pattern: '**/checkstyle-result.xml'])
        // In real life, PMD and Findbugs are unlikely to be used simultaneously
        step([$class: 'PmdPublisher', pattern: '**/pmd.xml'])
        step([$class: 'FindBugsPublisher', pattern: '**/findbugsXml.xml'])
        step([$class: 'AnalysisPublisher'])
    }
}
