# Porter Manifest

```yaml
#
# Standard CNAB Metadata
#
name: porter-azure-wordpress
version: 0.1.0
invocationImage: deislabs/porter-azure-wordpress:latest

credentials:
- name: SUBSCRIPTION_ID
  env: AZURE_SUBSCRIPTION_ID
- name: CLIENT_ID
  env: AZURE_CLIENT_ID
- name: TENANT_ID
  env: AZURE_TENANT_ID
- name: CLIENT_SECRET
  env: AZURE_CLIENT_SECRET
- name: kubeconfig
  path: /root/.kube/config

parameters:
- name: mysql_user
  type: string
  default: azureuser

- name: mysql_password
  type: string
  default: "!Th1s1s4p4ss!"

- name: database_name
  type: string
  default: "wordpress"

#
# Porter specific configuration ahead
#

# Mixins can be pulled in to make working with other systems easier
mixins:
- azure
- helm

# Steps to execute during the CNAB install action
install:
- description: "Create Azure MySQL"
  azure:
    type: mysql
    name: mysql-azure-porter-demo-wordpress
    resourceGroup: "porter-test"
    parameters:
      administratorLogin:
        source: bundle.parameters.mysql_user
      administratorLoginPassword:
        source: bundle.parameters.mysql_password
      location: "eastus"
      serverName: "mysql-jeremy-porter-test"
      version: "5.7"
      sslEnforcement: "Disabled"
      databaseName:
        source: bundle.parameters.database_name
  # Outputs from this step that should be made available to other steps in
  # the bundle        
  outputs:
  - name: "MYSQL_HOST"
    key: "MYSQL_HOST"

- description: "Helm Install Wordpress"
  helm:
    name: porter-ci-wordpress
    chart: stable/wordpress
    repalce: true
    set:
      mariadb.enabled: "false"
      externalDatabase.port: 3306
      readinessProbe.initialDelaySeconds: 120
      externalDatabase.host:
        # Sets externalDatabase.host to the MYSQL_HOST created in the previous step
        source: bundle.outputs.MYSQL_HOST
      externalDatabase.user:
        # Sets externalDatabase.user to the bundle's mysql_user parameter
        source: bundle.parameters.mysql_user
      externalDatabase.password:
        source: bundle.parameters.mysql_password
      externalDatabase.database:
        source: bundle.parameters.database_name

# Steps to execute during the CNAB uninstall action
uninstall:
  - description: "Uninstall Wordpress"
    azure:
      name: porter-ci-wordpress
```
