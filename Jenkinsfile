node {
    def app

    stage('Clone repository') {
        /* Let's make sure we have the repository cloned to our workspace */

        checkout scm
    }

    stage('Build image') {
        /* This builds the actual image; synonymous to
         * docker build on the command line */

        app = docker.build("sardaens/hellonode")
    }

    stage('Test image') {
        /* Ideally, we would run a test framework against our image.
         * For this example, we're using a Volkswagen-type approach ;-) */

        def container = app.run('-p 8000:8000')
        def contport = container.port(8000)
        println app.id + " container is running at host port, " + contport


        def ip = hostIp(container)

        println "container run on ip :" + ip

        app.inside {
           sh "sleep 10"
           sh "curl -f http://${ip}:8000"
        }
        /*
        * def resp = sh(returnStdout: true,
        *              script: """
        *              curl --connect-timeout 60 -w "%{http_code}" -o /dev/null -s http://\"${contport}\"
        *              """
        *              ).trim()
        * if ( resp == "200" ) {
        *    println " hello world is alive and kicking!"
        *    currentBuild.result = "SUCCESS"
        * } else {
        *    println "Humans are mortals."
        *    currentBuild.result = "FAILURE"
        * }
        */
    }

    stage('Push image') {
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */

         app.inside {
             sh 'echo "Push image latest on registry"'
         }

        docker.withRegistry('https://registry.hub.docker.com', 'DockerhubCredentials') {
            /*
            * app.push("${env.BUILD_NUMBER}")
            */
            app.push("latest")
        }
    }
}

def hostIp(container) {
  sh "docker inspect -f {{.NetworkSettings.IPAddress}} ${container.id} > host.ip"
  readFile('host.ip').trim()
}
