def registryUrl = 'registry.marathon.l4lb.thisdcos.directory:5000'
def appImage = '/demo/demoapp'
def dbImage = '/demo/demodb'
def marathonId = 'demo/spring-boot-demo'

def gitCommit() {
    sh "git rev-parse HEAD > GIT_COMMIT"
    def gitCommit = readFile('GIT_COMMIT').trim()
    sh "rm -f GIT_COMMIT"
    return gitCommit
}

node {
    // Checkout source code from Git
    stage 'Checkout'
    checkout scm

    // Build the Java project
    stage 'Build: Code'
//    sh '/bin/sh ./mvnw package'

    // Build Docker image
    stage 'Build: Docker Image'
    sh "docker build -t ${registryUrl}${appImage}:${gitCommit()} -t ${registryUrl}${appImage}:latest docker/app"
    sh "docker build -t ${registryUrl}${dbImage}:${gitCommit()} -t ${registryUrl}${dbImage}:latest docker/db"

    // Puch images
    stage 'Publish'
    sh "docker push ${registryUrl}${appImage}:${gitCommit()}"
    sh "docker push ${registryUrl}${appImage}:latest"
    sh "docker push ${registryUrl}${dbImage}:${gitCommit()}"
    sh "docker push ${registryUrl}${dbImage}:latest"

    // Deploy
    stage 'Deploy'
    marathon(
        url: 'http://marathon.mesos:8080',
        forceUpdate: true,
        filename: 'marathon_db.json',
        docker: "${registryUrl}${dbImage}:${gitCommit()}".toString()

    )

    marathon(
        url: 'http://marathon.mesos:8080',
        forceUpdate: true,
        filename: 'marathon_app.json',
        docker: "${registryUrl}${appImage}:${gitCommit()}".toString()
    )
}