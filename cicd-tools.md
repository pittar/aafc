# CI/CD Tools Demo

## Standard Pipeline

Follow the instructions in [this CI/CD tools example](https://github.com/siamaksade/openshift-cd-demo) to setup a full CI/CD pipeline that includes:
* Sonatype Nexus - Maven Proxy
* Jenkins
* SonarQube - Static Code Analysis
* Gogs - Git Repository Manager
* Eclipse Che (optional) - Web-based IDE running on OpenShift
* [Quay.io](quay.io) (optional) - External docker registry that can scan images for vulnerabilities.

This demo builds an application from source (from the Gogs repository), runs the unit tests, analyzis the code with SonarQube, then deploys the application to `dev` and `staging` environments.

The application that is deployed has a few built-in _features_, such as:
* Generate load - can be used to demo autoscaling
* Kill the application - watch OpenShift heal the app when it detects the container has crashed.

## Add OWASP Dependency Track

[OWASP Dependency Track](https://dependencytrack.org/) analyzes your projects dependencies for for known vulnerabilities and creates reports that can be used to audit your applications.

In your CI/CD project, execute the following command:
```
$ oc process -f \
    https://raw.githubusercontent.com/pittar/openshift-dependency-track/master/dependency-track.yaml \
    -p IMAGESTREAM_PROJECT="cicd-developer" \
    | oc create -f -
```

While that is deploying and updating it's internal vulnerability database (this can take upwards of 10-15min), you can update Jenkins to include the [Dependency Track Jenkins Plugin](https://plugins.jenkins.io/dependency-track).

In the Deployment Config of Jenkins, add the following `ENVIRONMENT` variable:
`INSTALL_PLUGINS` with value `dependency-track:2.1.0`

When you save the configuration, Jenkins will restart (this may take a minute or two).  Once Jenkins starts, open Jenkins in a new browser tab and navigate to `Manage Jenkins -> Configure System`.

There should now be a `Dependency Track` section.  
* Add in the service URL for the Dependency Track server (e.g. http://dependency-track:8080)
* Add a token (find this by logging into Dependency Track, then navigating to `Administration -> Access Management -> Teams)
    * Click on `Administrators`, then generate a new token, copy it, and paste it into the token input in the Jenkins config.
* Test your connection.  If it works, save and you're done.

## Add the CycloneDX Plugin to your project.

Open the `tasks` repo in Gogs and edit the `pom.xml` file in the root of the project.  Add the following plugin inside the *build plugins* block.
```
    <plugin>
        <groupId>org.cyclonedx</groupId>
        <artifactId>cyclonedx-maven-plugin</artifactId>
        <version>1.3.1</version>
    </plugin>
```

Then, edit your Jenkins pipeline in order to build the `bom.xml` file and upload it to Dependency Track for processing.  Replace the `Code Analysis` stage with this one:
```
    stage('Code Analysis') {
      steps {
        script {
          sh "${mvnCmd} sonar:sonar -Dsonar.host.url=http://sonarqube:9000 -DskipTests=true"
          sh "${mvnCmd} org.cyclonedx:cyclonedx-maven-plugin:makeBom"
          dependencyTrackPublisher(artifact: 'target/bom.xml', artifactType: 'bom', projectId: '<project uuid>', synchronous: false)
        }
      }
    }
```

Next time your pipeline runs, it will upload the new `bom.xml` file to Dependency Track for processing.  You can view the report there.