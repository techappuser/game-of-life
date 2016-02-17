
docker.image('cloudbees/java-build-tools:0.0.7.1').inside {
    checkout scm

    def mavenSettingsFile = "${pwd()}/.m2/settings.xml"

    stage 'Build Web App'
    wrap([$class: 'ConfigFileBuildWrapper',
        managedFiles: [[fileId: 'maven-settings-for-gameoflife', targetLocation: "${mavenSettingsFile}"]]]) {

        sh "mvn -s ${mavenSettingsFile} clean package"

        step([$class: 'ArtifactArchiver', artifacts: 'gameoflife-web/target/*.war'])
    }
}

docker.withRegistry('', 'dockerhub-credentials') {
    // build docker image 'cleclerc/game-of-life' and push it to docker hub
    stage 'Build & Push Docker Image'
    def gameOfLifeImage = docker.build('cleclerc/game-of-life', 'gameoflife-web')
    gameOfLifeImage.push()
}

stage 'Deploy ECS Service'
mail \
    to: 'cleclerc@cloudbees.com',
    subject: "Deploy version #${env.BUILD_NUMBER} on http://gameoflife-ecs.beesshop.org:8080/ ?",
    body: """\
       Deploy game-of-life#${env.BUILD_NUMBER} and start web browser tests on http://gameoflife-ecs.beesshop.org:8080/ ?
       Approve/reject on ${env.BUILD_URL}.
       """


input "Deploy on http://gameoflife-ecs.beesshop.org/ and run Selenium tests?"
checkpoint 'Deploy to QA'

docker.image('cloudbees/java-build-tools:0.0.7.1').inside {
    wrap([$class: 'AmazonAwsCliBuildWrapper', credentialsId: 'aws-cleclerc-admin', defaultRegion: 'us-east-1']) {
        // TODO THESE ARE PROBABLY NOT THE BEST ECS CALLS
        sh "aws ecs update-service --service game-of-life --desired-count 0"
        sleep 60
        sh "aws ecs update-service --service game-of-life --desired-count 1"
        sleep 20
        echo "game-of-life#${env.BUILD_NUMBER} SUCCESSFULLY deployed to http://gameoflife-ecs.beesshop.org:8080/"
    }
}

stage 'Web Browser tests'
retry(3) { // web browser tests are fragile, test up to 3 times
    docker.image('cloudbees/java-build-tools:0.0.7.1').inside {
        def mavenSettingsFile = "${pwd()}/.m2/settings.xml"
        wrap([$class: 'ConfigFileBuildWrapper',
            managedFiles: [[fileId: 'maven-settings-for-gameoflife', targetLocation: "${mavenSettingsFile}"]]]) {

            sh """
                curl http://gameoflife-ecs.beesshop.org/
                cd gameoflife-acceptance-tests
                mvn -s ${mavenSettingsFile} verify -Dwebdriver.driver=remote -Dwebdriver.base.url=http://gameoflife-ecs.beesshop.org:8080/
            """
        }
    }
}
