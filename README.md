# Gitops Environments

## Folder structure

The base directory holds configuration which is common to all environments. It is not expected to change often. If you want to do changes to multiple environments at the same time it is best to use the “variants” folder.

The variants folder holds common characteristics between environments. It is up to you to define what exactly you think is “common” between your environments after researching your application as discussed in the previous section.

In this example we have variants for staging environment and its regions. Here is an example deployment which applies to **ALL** staging environments (without region specification):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    codefresh.io/app: simple-go-app
  name: simple-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: trivial-go-web-app
  template:
    metadata:
      labels:
        app: trivial-go-web-app
    spec:
      containers:
      - env:
        - name: ENV_TYPE
          value: staging
        - name: DB_USER
          value: "staging_username"
        - name: DB_PASSWORD
          value: "staging_password"       
        image: docker.io/kostiscodefresh/simple-env-app:1.0
        imagePullPolicy: Always
        name: webserver-simple
        ports:
        - containerPort: 8080
```

In the example above we make sure that all staging environments are using the staging DB credentials and have set the `ENV_TYPE` variable.

With the base and variants ready we can now define every final environment with a combination of those properties.
Here is an example of the staging `nane1` environment:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- ../../../base/app-1
- secret.sops.yaml

components:
  - ../../../variants/staging/app-1
  - ../../../variants/staging-nane1/app-1


patchesStrategicMerge:
- deployment.yaml
```

First we define some common properties. We inherit all configuration from base, from non-prod environments and for all environments in North America North East.

