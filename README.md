# Environment-Configurator


### Description
This is a tool for managing configuration inside containers.
It is an Ansible playbook that is executed before the containers server component is started.

Uses jinja2-syntax to create templates for configuration files.  

An example container start-up procedure would look like this:
  - Container starts and run start-up shell script.
  - Shell script executes ansible-playbook.
  - Playbook searches for .j2 files and does variable substitution. Saves them without the .j2 suffix.
  - Start the server.

### Example

In this example we have an configuration file that is pointing out the database for an application.
We have two environments, qa and production.

#### Original configuration in QA
server.xml
```xml
<server>
   <!- .... ->
   <dataSource jndiName="jdbc/hello">
    <properties.oracle databaseName="qadbinst" serverName="qa-db.middleware.se" portNumber="1522"></properties.oracle>
   </dataSource>
</server>
```
#### Templated configuration
server.xml.j2
```xml
<server>
   <!- .... ->
   <dataSource jndiName="jdbc/hello">
    <properties.oracle databaseName="{{inst}}" serverName="{{server}}" portNumber="1522"></properties.oracle>
   </dataSource>
</server>
```
#### Configuration
The configuration is set by creating an json object that contains key:value pairs and then base64-encode it.

```json
cat qa.json
{
  "inst":"qadbinst",
  "server":"qa-db.middleware.se"
}

cat prod.json
{
  "inst":"proddbinst",
  "server":"prod-db.middleware.se"
}
```

Base64 encode the files.

```shell
base64 -w0 qa.json
ewogICJpbnN0IjoicWFkYmluc3QiLAogICJzZXJ2ZXIiOiJxYS1kYi5taWRkbGV3YXJlLnNlIgp9Cg==

base64 -w0 prod.json
ewogICJpbnN0IjoicHJvZGRiaW5zdCIsCiAgInNlcnZlciI6InByb2QtZGIubWlkZGxld2FyZS5zZSIKfQo=
```

Set the base64 encoded objects as the environment variable APP_CONFIG in the environment.

#### Result
After the container has started and the Ansible playbook has been executed the following files are available in the two different environments:

```xml
server.xml in qa:
<server>
   <!- .... ->
   <dataSource jndiName="jdbc/hello">
    <properties.oracle databaseName="qadbinst" serverName="qa-db.middleware.se" portNumber="1522"></properties.oracle>
   </dataSource>
</server>

server.xml in prod:
<server>
   <!- .... ->
   <dataSource jndiName="jdbc/hello">
    <properties.oracle databaseName="proddbinst" serverName="prod-db.middleware.se" portNumber="1522"></properties.oracle>
   </dataSource>
</server>
```


### Pre-Requirements

Ansible 2.x installed in the container.

### Installation

Put the configurator.yml script somewhere in the container and execute it as a part of the container startup process.

```shell
ansible-playbook /path/to/configurator.yml
```


### Configuring configurator.yml

The configuration of the configurator is done by setting operating system environment variables.

| Variable name     | Mandatory?| Description                                                        |
|-------------------|-----------|--------------------------------------------------------------------|
| CONFIG_DIR        | Required  | Base dir that will be searched recursive for j2-files              |
| CONFIG_ZIP        | Optional  | The name of the configuration zip file(-s)                         |
| APP_CONFIG        | Optional  | Base64 Encoded JSON object                                         |
| PLATFORM_CONFIG   | Optional  | Base64 Encoded JSON object                                         |
