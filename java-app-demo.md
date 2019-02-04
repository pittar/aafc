# Demo: Java App Build

## Copy Source to Gogs

For this demo, we'll use the stock Spring Pet Clinic app from Github.

1. Login to your Gogs instance (`http://gogs.<cluster domain>` from CI/CD tools instructions).
2. Create a new Organization (e.g. `aafc`).
3. Click the `+` icon at the top-right and select `New Migration`
    * Clone address: `https://github.com/spring-projects/spring-petclinic`
    * Repository name: `petclinic`
    * Click `Migrate Repository`

This will run for a few seconds, then you will have your own copy of the Pet Clinic code in your Gogs repository!

## Add a Jenkinsfile and Jar Name

1. Clone the repo locally:
```
$ git clone <gogs url>
```
2. Edit `pom.xml` and add the following XML after the opening `<build>` tag:
```
    <finalName>app</finalName>
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.sonarsource.scanner.maven</groupId>
                <artifactId>sonar-maven-plugin</artifactId>
                <version>3.6.0.1398</version>
            </plugin>
        </plugins>
    </pluginManagement>
```
3. Add the `Jenkinsfile` found in this repo uner `Jenkins/Jenkinsfile` to the *root* of the spring repository.
4. Login to SonarQube with user/pass `admin/admin` and generate a token (give it a name like `jenkins`).
5. Copy/paste this token into your `Jenkinsfile` as the argument to the `-Dsonar.login` parameter.
4. Add/commit/push the changes.
```
$ git add --all
$ git commit -m "Added Jenkinsfile and jar final name."
$ git push origin master
```

## Create the Pipeline Template

Create a new template in your CI/CD project to create a new Jenkins build pipeline with a Java build.

```
$ oc process -f https://raw.githubusercontent.com/pittar/aafc/master/ocp/build-template.yaml \
    -p APP_NAME="petclinic" \
    -p GIT_SOURCE_URL="http://gogs-cicd.192.168.64.2.nip.io/pittar/petclinic.git" \
    | oc create -f -
```

## Start the Build

You can use the command line:
```
$ oc start-build petclinic
```

Or manually through the UI:
`Builds -> Pipelines -> Start Pipeline`

## Create DEV and TEST Environments

Create two new projects:
```
$ oc new-project petclinic-dev --display-name="Petclinic DEV" --description="Petclinic DEV."
$ oc policy add-role-to-user \
    system:image-puller system:serviceaccount:petclinic-dev:default \
    -n cicd
$ oc policy add-role-to-user \
    edit system:serviceaccount:cicd:jenkins \
    -n petclinic-dev

$ oc new-project petclinic-test --display-name="Petclinic TEST" --description="Petclinic TEST."
$ oc policy add-role-to-user \
    system:image-puller system:serviceaccount:petclinic-test:default \
    -n cicd
$ oc policy add-role-to-user \
    edit system:serviceaccount:cicd:jenkins \
    -n petclinic-test
```