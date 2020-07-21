# Switch Manager

Este es el repositorio central de la aplicación Switch Manager.

Los componentes de la misma están concentrados en 4 repositorios
independientes:

1. `sm-api`: API para la interacción entre los clientes y AWX.
2. `sm-dashboard`: Dashboard para la administración de la aplicación.
3. `sm-infrastructure`: Archivos, scripts, y playbooks necesarios para levantar
   ambientes de producción y testing de la aplicación.
4. `sm-playbooks`: Playbooks que consume el AWX para interactuar con los
   equipos de red de los clientes.

Para simplificar la gestíon de estos proyectos se utilizará la herramienta
[`meta`](https://github.com/mateodelnorte/meta). Para utilizarla, es necesario
instalarla con `npm`.

```bash
npm install -g meta
```

Luego, clonamos este repositorio, corriendo con `meta git clone`:

```bash
meta git clone https://github.com/conatel-i-d/sm
```

## Logica del proyecto para correrlo en diferentes ambientes

El proyecto cuenta inicialmente con tres estructuras principales,

- La red de docker sobre la cual se montan y se exponen todos los servicios
- La base de datos, es un servicio independiente al resto que se encuentra colgado en la red docker

**NOTA:** si se baja la red docker y no la db, y al volver a levantar la docker-net es necesario recrear la base para que se reconecte

- El resto de los servicios que conforman la solución (api, dashboard y servicios auxiliares como keycloak, awx, etc)

## Pasos para levantar ambientes

Una vez que tenemos la esctuctura de repositorios clonada es necesario obtener o crear los siguentes archivos:

Dentro de la carpeta `sm-infraestructure` hay que crear un archivo password idealmente que sea una llave privada.
Esta llave nos permite crear por medio de `ansible-vault` un archivo con las variables 
"secretas" que necesitamos y lo cifra.
Una vez que tenemos el archivo y parado dentro de `sm-infraestructure`, para crear ese archivo secret.yml ejecutamos

```bash
make create_secrets
```

este comando crea y abre el archivo para colocar nuestras propias variables, las variables con las que
debe contar ese archivo son las siguientes:

```yml
# Main vars
project_name: sm
project_folder: '~/Documents/Projects/sm'
build_folder: '{{ project_folder }}/sm-infrastructure/lib' # se puede usar por defecto
output_folder: '{{ build_folder }}/ouputs' # se puede usar por defecto
certificates_folder: '{{ build_folder }}/certificates' # se puede usar por defecto
docker_compose_dir: '{{ build_folder }}/docker' # se puede usar por defecto
project_data_dir: '{{ project_folder }}/sm-playbooks' # se puede usar por defecto
api_folder: '{{ project_folder  }}/sm-api' # se puede usar por defecto
dashboard_folder: '{{ project_folder }}/sm-dashboard' # se puede usar por defecto
public_domain: sm.conatest.click
aws_access_key: 
aws_secret_key: 
aws_region: us-east-1
key_name: '{{ project_name }}_key'
email_address: scattonec@saltogrande.org
smtp_username: 
smtp_password: 
smtp_server: email-smtp.us-east-1.amazonaws.com
smtp_port: 25
smtp_tls: Yes
smtp_from: sm@mail.conatel.cloud
smtp_from_display_name: Conatel Switch Manager
# AWX Vars
awx_version: 7.0.0
awx_database_name: awx
awx_admin_username: awx
awx_admin_password: awxpassword
secret_key: C0n4t3lC0n4t3l
# PostgreSQL
pg_admin_username: postgres
pg_admin_password: postgrespassword
pg_port: 5432
pg_hostname: postgres
# Keycloak
keycloak_version: 7.0.0
keycloak_admin_username: keycloak
keycloak_admin_password: keycloakpassword
keycloak_database_name: keycloak
keycloak_realm: sm
# API
api_admin_username: admin
api_admin_password: adminpassword
api_database_name: api
# Frontend
ansible_python_interpreter: /home/ibarreto/.pyenv/shims/python
# Cisco Prime
cisco_prime_is_desable: True
cisco_prime_base_url: https://172.16.1.217
cisco_prime_user: ignaciob
cisco_prime_password: 2020Conatel
prime_switches_ssh_user: swmanager
prime_switches_ssh_pass: "@tencioN20"
#ldap
ldap_user: serviciosco
ldap_password: CoSoft20
# Dockerhub credentials
dockerhub_user: conatelid
dockerhub_password: C0n4t3lC0n4t3l
#switch publico
sw_public_ip: 200.58.153.100
sw_public_os: ios
sw_public_ssh_user: ialmandos
sw_public_ssh_pass: ialmandos
sw_public_ssh_port: 2222
#switch privado
sw_private_ip: 172.16.1.55
sw_private_os: ios
sw_private_ssh_user: netconf_username
sw_private_ssh_pass: "C0n4t3l."
sw_private_ssh_port: 2222
```


Si quisiéramos volver a editar los secretos, estando parado en `sm-infraestructura` ejecutar `make secret`

Una vez que configuramos todas las variables necesarias para el proyecto, estamos en condiciones 
de levantar los servicios para esto ejecutamos los siguientes commandos

```bash
make setup #crea la estructura de directorios y la red de docker
make certs # solo si estamos en producción, crea los certificados para https
make db_up # levanta la base de datos
make dev # si estamos en dev - levanta todos los servicios en modo dev para poder
# debuggear y modificar sin necesidad de recargar manual api ni dashboard
make prod # si estamos en prod - previamente se debe haber compilado la version
# de api y dashboard que se quiere iniciar.
# <<<Proceso de compilación mas adelante>>>, necesario haber creado los certs
make build-tower-cli # crea la imagen de el contenedor utilizado para enviar las configuraciones al awx
make awx-send # envia las configuraciones al awx
```

**DEV-SHORTCUT:** `make setup && make db_up && make dev && make build-tower-cli && make awx-send`

**PROD-SHORTCUT:** `make setup && make certs && make db_up && make prod && make build-tower-cli && make awx-send`

**NOTA:** es importante respetar el orden en que se ejecutan los comandos porque cada uno depende de los anteriores.
 Por ejemplo db se debe colgar de docker network que se crea en setup.

### Bajar un ambiente completo

Para bajar un ambiente completo es necesario ejecutar:

```bash
make _db_down # baja la base de datos y borra el volumen <<<<<<<<<<<<<<<< DANGERRRR OJO!!!!!!!
make tear_down # baja el resto de los servicios, borra los templates, borra la red docker
```

El mismo procedimiento es necesario para levantar un ambiente de producción, la
única diferencia siendo el nombre de la tarea: `make deploy`.
