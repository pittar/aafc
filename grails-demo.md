# Grails Demo

## Migrate Grails app to Gogs

https://github.com/grails-samples/music

## Build with Java s2i

The Fabric8 Java s2i upstream image can build Grails/Gradle apps.

To start a build, run the following command, substituting your gogs service if it is different.

```
$ oc new-app fabric8/s2i-java:3.0-java8~http://gogs:3000/gogs/music
```

When the app has deployed, create a route and you should be able to access your grails app.

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