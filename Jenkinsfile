pipeline {
    agent {
        label "maven"
    }
    options { 
        skipDefaultCheckout()
        disableConcurrentBuilds()
    }
    stages {
        stage("Checkout") {
            steps {
                checkout(scm)   
            }
        }
        stage("Test") {
            steps {
                sh "mvn test"
            }
        }
        stage("Build") {
            steps {
                script {
                    openshift.withCluster {
                        openshift.withProject {
                            env.TAG = readMavenPom().getVersion()
                            openshift.selector("bc", "hello-openshift").startBuild("--wait=true")
                            openshift.tag("telefonica-dev/hello-openshift:latest", "hello-openshift:${env.TAG}")
                        }
                    }
                }
            }
        }
        stage("Deploy DEV") {
            steps {
                script {
                    openshift.withCluster {
                        openshift.withProject {
                            if (!openshift.selector("dc", "hello-openshift").exists()) {
                                openshift.apply(openshift.process(readFile("src/main/resources/deploy.yaml"), 
                                                                  "-p APPLICATION_NAME=hello-openshift", 
                                                                  "-p IMAGE_NAME=hello-openshift", 
                                                                  "-p IMAGE_TAG_NAME=${env.TAG}"))
                            } else {
                                openshift.set("triggers", "dc/hello-openshift", "--remove-all")
                                openshift.set("triggers", "dc/hello-openshift", "--from-image=hello-openshift:${env.TAG}", "-c hello-openshift")
                            }
                            
                            openshift.selector("dc", "hello-openshift").rollout().status()
                        }
                    }
                }
            }
        }
    }
}
