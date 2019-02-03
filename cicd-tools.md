# CI/CD Tools Demo

One potential set of tools to use in a DevSecOps pipeline for Java projects.

## 0: Connect with OC CLI Tool

1.  Download the [oc cli tool](https://github.com/openshift/origin/releases/tag/v3.11.0) for your OS and add it to your `PATH`
2.  Connect to your OpenShift instance: `$ oc login -u <user> <cluster url>`
    * (If using Minishift: `$oc login -u developer`)

## 1: DevSecOps Infrastructure

The next few instructions will be to install the basic infra required for the pipeline.

### CI/CD Project

Create a new project for your CI/CD tools.
```
$ oc new-project cicd --display-name="CI/CD Environment" --description="CI/CD Environment."
```

### Nexus: Maven Repository

SonaType Nexus [official OpenShift templates](https://github.com/sonatype-nexus-community/deployment-reference-architecture/tree/master/OpenShift)

We'll use the version from OpenShiftDemos (works on Minishift).

```
$ oc process -f \
    https://raw.githubusercontent.com/OpenShiftDemos/nexus/master/nexus3-persistent-template.yaml \
    -p NEXUS_VERSION="3.15.2" \
    -p MAX_MEMORY="2Gi" \
    | oc create -f -
```

Default admin username and password: `admin/admin123`

### SonarQube: Code Quality

Read about SonarQube:
* [Blog: Code Analysis with SonarQube](https://www.baeldung.com/sonar-qube)
* [SonarQube: Official Docs](https://www.sonarqube.org/)

SonarQube [OpenShift template](https://github.com/OpenShiftDemos/sonarqube-openshift-docker)

We'll use the template that includes a PostgreSQL database, instead of embedded H2.

Default admin username and password: `admin/admin`

```
$ oc process -f \
    https://raw.githubusercontent.com/OpenShiftDemos/sonarqube-openshift-docker/master/sonarqube-postgresql-template.yaml \
    -p SONARQUBE_VERSION="7.0" \
    | oc create -f -
```

### Gogs: Git Repository manager

Read about Gogs:
* [Gogs home](https://gogs.io/)

Gogs [OpenShift template](https://github.com/OpenShiftDemos/gogs-openshift-docker/tree/master/openshift)

We'll use the template that includes a PostgresSQL database to persist state across restarts.

```
$ oc process -f \
    https://raw.githubusercontent.com/OpenShiftDemos/gogs-openshift-docker/master/openshift/gogs-persistent-template.yaml \
    -p GOGS_VERSION="0.11.34" \
    -p HOSTNAME="gogs-cicd.<cluster url>" \
    | oc create -f -
```

Default admin username and password:  First registered user is the admin.

### Clean-up

#### Deleting Gogs

```
$ oc delete all -l app=gogs
$ oc delete cm -l app=gogs
$ oc delete pvc -l app=gogs
$ oc delete sa -l app=gogs
```
