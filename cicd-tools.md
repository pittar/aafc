# CI/CD Tools Demo

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

