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

        /*
        * app.inside {
        *     sh "sleep 10000"
        *     sh "curl -f http://127.0.0.1:8000"
        * }
        */

        def container = app.run('-p 8000')
        def contport = container.port(8000)
        println app.id + " container is running at host port, " + contport
        def resp = sh(returnStdout: true,
                      script: """
                      curl -w "%{http_code}" -o /dev/null -s http://\"${contport}\"
                      """
                      ).trim()

    }

    stage('Push image') {
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */

         app.inside {
             sh 'echo "Push image on registry"'
         }

        docker.withRegistry('https://registry.hub.docker.com', 'DockerhubCredentials') {
            app.push("${env.BUILD_NUMBER}")
            app.push("latest")
        }
    }
}
