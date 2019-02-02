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

## Create the Pipeline Template

Create a new template in your CI/CD project to create a new Jenkins build pipeline with a Java build.

```
$ oc create -f 
```

