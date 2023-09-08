# Camunda Swarm Stack
This stack can be used as base to production Camunda 7 Run deployment.
## Prerequisites
- Server with at least 4vCPU and 10GB and *sudo* installed
- Docker Swarm started configured to use rootless mode
- A configured reverse proxy behind this stack, pointing <CAMUNDA_HOSTNAME> to <CAMUNDA_STACK_INGRES_IP>:8080
- An external PostgresSQL (15 tested) database
- An external on-premise (.local) LDAP directory for authentication and authorization users and groups

## Steps to deploy
### Database
1) On Postgres host, create database and user (which need to be owner on Postgres 15):
````
sudo -u postgres psql
````
````
create database camunda;
````
````
create user camunda with encrypted password '<CAMUNDA_DB_PASS>';
````
````
grant all privileges on database camunda to camunda;
````
````
ALTER DATABASE camunda OWNER TO camunda;
````
2) Enable remote connections on Postgres updating _postgresql.conf_ and _pg_hba.conf_. To find the files location, use this:
````
sudo -u postgres psql -c 'SHOW config_file'
````
For more information, consult this:
>https://stackoverflow.com/questions/18580066/how-to-allow-remote-access-to-postgresql-database

### LDAP Directory
1) Create a group named "camunda-admins" and include all camunda admin users.
2) Create a service account to be the *manager_dn* who will connect to LDAP server (like Active Directory) - 
_service.camunda@<YOUR_DOMAIN>.local_ - with read only rights.

### Camunda Stack
1) Create directories for project and change to "/app" directory:
````
sudo mkdir /app; \
sudo chown -R $USER:docker /app; \
sudo chmod -R 770 /app; \
cd /app
````
2) Clone this repository
````
git clone <REPO_URL>.git
````
3) Go to camunda stack folder:
````
cd /app/camunda-stack
````
3) Update the values on _camunda-stack.yml_ and _default.yml_ accordingly to your infrastructure.
4) Create secret to encrypt database password:
````
echo "<CAMUNDA_DB_PASS>" | docker secret create camunda_db_pass -
````
5) Adjust directory/file permissions on docker host(s):
````
sudo chown -R 1000:1000 /app/camunda-stack; \
sudo chmod -R 440 /app/camunda-stack; \
````
6) Enable (only) necessary communications on your hosts and network gateways. This stack use port 8080.
For example, to enable this port on your docker host using firewalld:
````
sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent; \
sudo firewall-cmd --reload
````
7) You can do more things in order to secure your deployment based on your infrastucture resources. Please, see this for more tips. 
>https://docs.camunda.org/manual/latest/user-guide/security/
8) Run the stack:
````
docker stack deploy -c camunda-stack.yml camunda
````
**Web apps endpoint**
>https://<CAMUNDA_HOSTNAME>/camunda

**REST Endpoint**
>https://<CAMUNDA_HOSTNAME>/engine-rest
