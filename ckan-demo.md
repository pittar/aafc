# CKAN on OpenShift Demo

## Background

Following these steps will get [CKAN]() up and running on OpenShift based on the images they provide in [Docker Hub])().

However, there are a few trade-offs to ge this running:

1.  You have to run the containers with a `service account` that has the `anyuid` scc.  This is something you normally wouldn't allow in a production environemnt.
2.  The docker images don't seem to be well supported by CKAN.
3.  This demo is ephemeral.  No volumes have been created or mounted.  Restarting the apps (or crash) will lose state/data.

There Dockerfiles available for the main components, so it would be possible to create better (more secure) containers if running CKAN on OpenShift goes beyond a _proof of concept_ state.

Source documents:
* [CKAN docs for docker compose](https://docs.ckan.org/en/2.8/maintaining/installing/install-from-docker-compose.html)
* [Environment config docs](https://docs.ckan.org/en/2.8/maintaining/configuration.html)
* [CKAN source on Github](https://github.com/ckan/ckan)
* [CKAN docker-compose file](https://github.com/ckan/ckan/blob/master/contrib/docker/docker-compose.yml)

## Instructions

### Create a New Project and Service Account

Create the project.
```
$ oc new-project ckan
```

Have your OpenShift Administrator login with their admin credentials and create a new `serviceaccount` with the `anyuid` scc.
```
$ oc project ckan
$ oc adm policy add-scc-to-user anyuid -z ckan
```

This is required in order to run the CKAN contianers.

### Instantiate the Template (Command Line)

Run the following command.  Make sure to substitute `<your ip>` with your actual ip.
```
$ oc process -f \
    https://raw.githubusercontent.com/pittar/aafc/master/ocp/ckan-template.yaml \
    -p CKAN_URL="ckan.<your ip>.nip.io" \
    | oc create -f -
```

Depending on the order that the containers finish pulling and starting, CKAN may fail to start.  If this happens, skale down the ckan pod, wait until the other pods have started, then scale it back up.

### Instantiate the Template (UI)

* In your ckan project, click `Add to Project -> Import YAML/JSON`.
* Copy/Paste the contents of [https://raw.githubusercontent.com/pittar/aafc/master/ocp/ckan-template.yaml](https://raw.githubusercontent.com/pittar/aafc/master/ocp/ckan-template.yaml) into the text box.
* Click the *Create* button.
* Make sure *Process the Template* is checked and continue.
* Enter your desired CKAN url (e.g. `ckan.192.168.64.2.nip.io`) and click *Create*, then *Close*.

The containers should be starting up in the background.

### What Does This Create?

* One `ImageStream` per image.
* One `DeploymentConfig` per component.
* One `Service` per component.
* One `Route` for the CKAN app.
* One `ConfigMap` containing the environment variables used to configure CKAN.

### Clean-up

The easy way is to simply *delete the project*

However, if you only want to delete the contents of the project (so that you don't hvae to re-create the `ckan` service account), run:
```
$ oc delete all,cm -l app=ckan
```

This will delete all `ImageStreams`, `DeploymentConifgs`, `Pods', `Routes`, and `ConfigMaps` that have the `app=ckan` label attached to them. 

