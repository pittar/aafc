# Grails Demo

## Gradle/Grails Demo App

For the demo, we'll use this simple app from GitHub.
[https://github.com/grails-samples/music](https://github.com/grails-samples/music)

You can either migrate this repo into your Gogs instance, or you can just use the GitHub repo directly.

## Create a Gradle Jenkins Slave

 The [Red Hat Community of Practice](https://github.com/redhat-cop) Github org has you covered with this [Jenkins Gradle Slave](https://github.com/redhat-cop/containers-quickstarts/tree/master/jenkins-slaves/jenkins-slave-gradle).

To build this slave in your OpenShift environment, simply run the following command in the project where your Jenkins master instance lives:

```
$ oc process -f https://raw.githubusercontent.com/redhat-cop/containers-quickstarts/master/jenkins-slaves/.openshift/templates/jenkins-slave-generic-template.yml \
    -p NAME=jenkins-slave-gradle \
    -p SLAVE_IMAGE_TAG=latest \
    -p SOURCE_CONTEXT_DIR=jenkins-slaves/jenkins-slave-gradle \
    | oc create -f -
```

Ok!  When that build completes, you will have a new [ImageStream](https://docs.okd.io/latest/architecture/core_concepts/builds_and_image_streams.html#image-streams) that is labeled with `role=jenkins-slave` so that Jenkins master will automatically be able to pick up and use this new agent type.

*NOTE:* If Jenkins is already running, you will have to restart Jenkins for it to become aware of the new agent.

## Let DEV and STAGE Project Pull!

This is different than the other example.  This time, all the images will remain in the `ci/cd` project.  The `dev` and `stage` projects will simply pull images from the `ci/cd` project.  To do this, these projects need permission!

```
$ oc adm policy add-role-to-user system:image-puller system:serviceaccount:dev-developer:default -n cicd-developer
$ oc adm policy add-role-to-user system:image-puller system:serviceaccount:stage-developer:default -n cicd-developer
```

## Add a Jenkinsfile

Add a file named `Jenkinsfile` to the root of your repository with the the following contents:

```
try {
    def appName=env.APP_NAME
    def gitSourceUrl=env.GIT_SOURCE_URL
    def gitSourceRef=env.GIT_SOURCE_REF
    def project=""
    node {
        stage("Initialize") {
            project = env.PROJECT_NAME
            echo "project: ${project}"
            echo "appName: ${appName}"
            echo "gitSourceUrl: ${gitSourceUrl}"
            echo "gitSourceUrl: ${gitSourceUrl}"
            echo "gitSourceRef: ${gitSourceRef}"
        }
    }
    node("jenkins-slave-gradle") {
        stage("Checkout") {
            git url: "${gitSourceUrl}", branch: "${gitSourceRef}"
        }
        stage("Build JAR") {
            sh "gradle build"
            sh "cp build/libs/*.jar build/libs/app.jar"
            stash name:"jar", includes:"build/libs/app.jar"
        }
    }
    node {
        stage("Build Image") {
            unstash name:"jar"
            sh "oc start-build ${appName}-build --from-file=build/libs/app.jar -n ${project}"
        }
    }
    node {
        stage("Deploy DEV") {
            openshift.withCluster() {
                openshift.withProject("${project}") {
                    openshift.tag("${appName}:latest", "${appName}:dev")
                }
            }
        }
    }
    node {
        stage("Deploy TEST") {
            input "Deploy to TEST?"
            openshift.withCluster() {
                openshift.withProject("${project}") {
                    openshift.tag("${appName}:dev", "${appName}:test")
                }
            }
        }
    }
} catch (err) {
    echo "in catch block"
    echo "Caught: ${err}"
    currentBuild.result = 'FAILURE'
    throw err
}
```

## To Use Nexus

In your `gradle.build`, update the `maven` url to point to your instance of Nexus.  For example:

```
    repositories {
        mavenLocal()
        maven { url "http://nexus:8081/repository/maven-public/" }
    }
```

If your build was using a non-standard Maven repo (e.g. https://repo.grails.org/grails/core), then you need
to add it to Nexus.

1. Log in to Nexus as `admin/admin123`
2. Go to "Server Administration" (gear icon at top of screen).
3. `Repositories -> Create repository`
4. Select `maven2 (proxy)`
5. Give the repo a name (e.g. `grails`)
6. Add the repository url (e.g. https://repo.grails.org/grails/core)
7. Click `Create repository`

Now, add this repo to the `maven-public` repository group.

1. Click `Repositories`
2. Click on `maven-public`
3. Under *Group*, select the repo you just created in the left box (`grails`)
4. Click `>` to add add the repo to the *Members* list.
5. Click `Save`

You're build should now use your Maven proxy!

## Alternative: Build with Java s2i

The Fabric8 Java s2i upstream image can build Grails/Gradle apps.

To start a build, run the following command, substituting your gogs service if it is different.

```
$ oc new-app fabric8/s2i-java:3.0-java8~http://gogs:3000/gogs/music
```

When the app has deployed, create a route and you should be able to access your grails app.