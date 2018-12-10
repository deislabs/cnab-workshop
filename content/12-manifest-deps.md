# Bundle Dependencies

## Reusable MySQL Bundle

```yaml
name: mysql
version: 0.1.0
invocationImage: porter-mysql:latest

credentials:
- name: kubeconfig
  path: /root/.kube/config

parameters:
- name: database-name
  type: string
  default: mydb
  destination:
    env: DATABASE_NAME
- name: mysql-user
  type: string
  destination:
    env: MYSQL_USER

mixins:
- helm

install:
- description: "Install MySQL"
  helm:
    name: porter-ci-mysql
    chart: stable/mysql
    version: 0.10.2
    replace: true
    set:
      mysqlDatabase:
        source: bundle.parameters.database-name
      mysqlUser:
        source: bundle.parameters.mysql-user
  outputs:
  - name: mysql-root-password
    secret: porter-ci-mysql
    key: mysql-root-password
  - name: mysql-password
    secret: porter-ci-mysql
    key: mysql-password
uninstall:
- description: "Uninstall MySQL"
  helm:
    name: porter-ci-mysql
    purge: true
```

## Wordpress Bundle

```yaml
mixins:
- helm

name: wordpress
version: 0.1.0
invocationImage: porter-wordpress:latest

dependencies:
- name: mysql
  parameters:
    database_name: wordpress
    mysql_user: wordpress

credentials:
- name: kubeconfig
  path: /root/.kube/config

parameters:
- name: wordpress-name
  type: string
  default: porter-ci-wordpress
  destination:
    env: WORDPRESS_NAME

install:
- description: "Install Wordpress"
  helm:
    name:
      source: bundle.parameters.wordpress-name
    chart: stable/wordpress
    replace: true
    set:
      externalDatabase.database:
        source: bundle.dependencies.mysql.parameters.database-name
      externalDatabase.host: # NOTE: Not implemented yet!
        source: bundle.dependencies.mysql.services.mysql
      externalDatabase.user:
        source: bundle.dependencies.mysql.parameters.mysql-user
      externalDatabase.password:
        source: bundle.dependencies.mysql.outputs.mysql-password

uninstall:
- description: "Uninstall Wordpress"
  helm:
    name:
      source: bundle.parameters.wordpress-name
```
