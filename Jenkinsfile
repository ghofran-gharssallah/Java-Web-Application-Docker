node {
    def IMAGE   = "ghofan/java-web-app-cd"                       // image Docker Hub (examen réel : <compte>/java-web-app)
    def SERVICE = "java-web-app-cd"                              // service Swarm (examen réel : java-web-app)
    def TAG     = "${env.BUILD_NUMBER}"                          // numéro du build = version de l'image
    def APP_URL = "http://192.168.198.128:8090/java-web-app/"    // IP de la VM, pas localhost (Jenkins est dans un conteneur)

    stage('SCM Checkout') {
        checkout scm                                             // prendre le code du commit qui a lancé le build
    }
    stage('Maven Clean Package') {
        def mavenHome = tool name: "Maven-3.9", type: "maven"    // chemin du Maven installé par Jenkins
        sh "${mavenHome}/bin/mvn clean package"                  // compiler et créer target/java-web-app-1.0.war
        sh "ls -l target/*.war"                                  // afficher l'archive (demandé Q2)
    }
    stage('Build Docker Image') {
        sh "docker build -t ${IMAGE}:${TAG} ."                   // construire l'image avec le numéro du build
        sh "docker tag ${IMAGE}:${TAG} ${IMAGE}:latest"          // ajouter l'étiquette latest à la même image
    }
    stage('Push Docker Image') {
        withCredentials([string(credentialsId: 'docker_hub_pwd', variable: 'docker_hub_pwd')]) {
            sh 'echo $docker_hub_pwd | docker login -u ghofan --password-stdin'   // login Docker Hub sans afficher le mot de passe
        }
        sh "docker push ${IMAGE}:${TAG}"                         // envoyer la version N
        sh "docker push ${IMAGE}:latest"                         // envoyer latest
    }
    stage('Deploy') {
        sh """
            if docker service inspect ${SERVICE} > /dev/null 2>&1; then
                docker service update --image ${IMAGE}:${TAG} ${SERVICE}
            else
                docker service create --name ${SERVICE} --replicas 2 -p 8090:8080 --update-failure-action rollback ${IMAGE}:${TAG}
            fi
        """
    }
    stage('Smoke') {
        sh """
            for i in \$(seq 1 10); do
                if curl -s ${APP_URL} | grep -q 'Spring MVC'; then
                    echo "Smoke OK a la tentative \$i"
                    exit 0
                fi
                echo "Tentative \$i/10 : pas encore OK, attente 5 s"
                sleep 5
            done
            echo "Smoke KO : retour arriere"
            docker service update --rollback ${SERVICE}
            exit 1
        """
    }
}
