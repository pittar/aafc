# Dependency Track

[Dependency Track Image](https://hub.docker.com/r/owasp/dependency-track)
Currently using tag `3.4.0`

## To Make This Work

1. Create a PVC and mount to `/data`
2. Add the following env vars (internal H2 database):
    * `ALPINE_DATA_DIRECTORY=/data/.dependency-track
    * `ALPINE_DATABASE_URL=jdbc:h2:/data/.dependency-track/db`

This will get dependency-track up and running.

For Jenkins, include the env var:
`INSTALL_PLUGINS=dependency-track`

Once Jenkins is up and running, open the Jenkins URL, go to `Manage Jenkins` and configure the `Dependency Track` plugin.

Add the service url (e.g. http://dependency-track.cicd.svc:8080)
Get the API key from the "Automation" team listing in Dependency Track and paste it in: `Administration -> Teams -> Automation`
Save your config.