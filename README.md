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

```vim
# Main vars
project_folder: '/ruta/a/tu/proyecto/sm'
build_folder: '{{ project_folder }}/sm-infrastructure/lib' # se puede usar por defecto
output_folder: '{{ build_folder }}/ouputs' # se puede usar por defecto
certificates_folder: '{{ build_folder }}/certificates' # se puede usar por defecto
docker_compose_dir: '{{ build_folder }}/docker' # se puede usar por defecto
project_data_dir: '{{ project_folder }}/sm-playbooks' # se puede usar por defecto
api_folder: '{{ project_folder  }}/sm-api' # se puede usar por defecto
dashboard_folder: '{{ project_folder }}/sm-dashboard' # se puede usar por defecto
project_name: sm # se puede usar por defecto
public_domain: sm.conatest.click # esta variable cambia entre entornos de dev y prod, ademas en dev se debe aplicar una regla
`/etc/hosts` para que vaya a localhost
aws_access_key: #  clave de acceso de aws - usado para pruebas
aws_secret_key: #  clave secreta de aws - usado para pruebas
aws_region: us-east-1 # region aws - usado para pruebas
key_name: '{{ project_name }}_key' # creo esta sin uso - TODO: verificar
# Configuracion SMTP para el AWX
email_address: gmonne@conatel.com.uy
smtp_username: # usuario smtp para el awx
smtp_password: # password smtp para el awx
smtp_server: email-smtp.us-east-1.amazonaws.com
smtp_port: 25
smtp_tls: Yes
smtp_from: sm@mail.conatel.cloud
smtp_from_display_name: Conatel Switch Manager
# AWX Vars
awx_version: 7.0.0
awx_database_name: # awx db name
awx_admin_username: # awx db user name
awx_admin_password: # awx db password
postgres_data_dir: '{{ docker_compose_dir }}/pgdocker' # se puede usar por defecto
secret_key: # TODO: verificar para que se usa
# PostgreSQL
pg_admin_username: #db admin user
pg_admin_password: #db password user
pg_port: 5432 # db port
pg_hostname: # db host
# Keycloak
keycloak_version: 7.0.0
keycloak_admin_username: # keycloak user name
keycloak_admin_password: # keycloak password
keycloak_database_name: # keycloak db name
keycloak_realm: sm # keycloak realm para la solución - se puede usar por defecto
# Cisco Prime
cisco_prime_user: # usuario del cisco prime
cisco_prime_password: # password del cisco prime
# API
api_admin_username: # nombre del usuario administrador de la api
api_admin_password: # password para el usuario administrador de la api
api_database_name: # nombre de la db para la api

# Ubicación de python en tu entono de desarrollo
ansible_python_interpreter: /usr/bin/env python3.7
#ldap
ldap_user: # nombre de usuario para el ldap
ldap_password: # password para el ldap
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
