# Primer deploy del proyecto

En este punto entendemos se cuenta con todo el proyecto completo, la estructura de directorios esta formada y cuenta con los repos necesarios para inicializar el proyecto, explicado en el [documento principal](../README.md)

## Infraestructura lógica del proyecto

El proyecto cuenta inicialmente con tres estructuras principales,

- La red de docker sobre la cual se montan y se exponen todos los servicios
- La base de datos, es un servicio independiente al resto que se encuentra colgado en la red docker

**NOTA:** si se baja la red docker y no la db, y al volver a levantar la docker-net es necesario recrear la base para que se reconecte

- El resto de los servicios (contenedores) que conforman la solución (api, dashboard y servicios auxiliares como keycloak, awx, etc)

### Pasos para levantar ambientes

Una vez que tenemos la estructura de repositorios clonada es necesario obtener o crear los siguientes archivos:

Dentro de la carpeta `sm-infrastructure` hay que crear un llave privada RSA256 con nombre `password`, en adelante esta llave sera utilizada para cifrar las variables de entorno por lo tanto no se encuentra en ningun otro lado que el servidor hasta que no se respalde.
Esta llave nos permite crear por medio de `ansible-vault` un archivo con las variables secretas que necesitamos y lo cifra.
Una vez que tenemos el archivo y estando parado dentro de `sm-infrastructure`, ejecutamos

```bash
make create_secrets
```

para crear ese archivo secret.yml. Este comando ademas de crear, abre el archivo para colocar las variables customizadas para este deploy del proyecto, las variables que se deben configurar para este proyecto son las siguientes:

```yml
# Main vars
project_name: sm
project_folder: '/route/to/your/local/project' # debe configurarse para la instalacion particular y dbe coincidir con la ruta a la carpeta raiz del proyecto
build_folder: '{{ project_folder }}/sm-infrastructure/lib' # se recomienda utilizar el por defecto
output_folder: '{{ build_folder }}/ouputs' # se recomienda utilizar el por defecto
certificates_folder: '{{ build_folder }}/certificates' # se recomienda utilizar el por defecto
docker_compose_dir: '{{ build_folder }}/docker' # se recomienda utilizar el por defecto
project_data_dir: '{{ project_folder }}/sm-playbooks' # se recomienda utilizar el por defecto
api_folder: '{{ project_folder  }}/sm-api' # se recomienda utilizar el por defecto
dashboard_folder: '{{ project_folder }}/sm-dashboard' # se recomienda utilizar el por defecto
public_domain: public.domain
aws_access_key: #solo para dev, cuando se usaba route53
aws_secret_key: #solo para dev, cuando se usaba route53
aws_region: us-east-1
key_name: '{{ project_name }}_key'
email_address: scattonec@saltogrande.org
smtp_username: smtp_user
smtp_password: smtp_pass
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
secret_key: clave_secreta_para_awx
# PostgreSQL
pg_admin_username: your_user # usuario por defecto para administración de la base de datos
pg_admin_password: your_pass # password por defecto para administración de la base de datos
pg_port: 5432 # puerdo de la base de datos
pg_hostname: postgres # nombre del host (contenedor) donde se encuentra la base de datos
# Keycloak
keycloak_version: 7.0.0 # version de keycloak utilizada en el proyecto
keycloak_admin_username: keycloak # usuario por defecto para la db y administracion de keycloak
keycloak_admin_password: keycloakpassword # password por defecto para la db y administracion de keycloak
keycloak_database_name: keycloak # nombre para la base de datos de keycloak
keycloak_realm: sm # nombre del realm donde se van a alojar los usuarios en keycloak
# API
api_admin_username: your_user # usuario para la base de datos de la api
api_admin_password: your_pass # password para la base de datos de la api
api_database_name: your_db # nombre para la base de datos de la api
ansible_python_interpreter: /your/python/with/andisble #ejemplo /usr/bin/python3 o /usr/bin/python direccion al binario python donde se encuentra ansible instalado
# Cisco Prime
cisco_prime_is_desable: True # Habilita/deshabilita cisco prime por si solo se trabaja con sw agregados de forma manual
cisco_prime_base_url: your_prime_url # https://123.123.123.1
cisco_prime_user: your_prime_user
cisco_prime_password: your_prime_pass
prime_switches_ssh_user: your_sw_user
prime_switches_ssh_pass: your_sw_pass
#ldap
ldap_user: your_ldap_user
ldap_password: your_ldap_pass
# Dockerhub credentials
dockerhub_user: docker_user # Solo para en fase de desarrollo subir las imagenes
dockerhub_password: docker_pass # Solo para en fase desarrollo subir las imagenes
#switch publico
sw_public_ip: 123.123.123.123 # SW agregados solo para test, si no se configuran no afecta
sw_public_os: your_so # SW agregados solo para test, si no se configuran no afecta
sw_public_ssh_user: your_user # SW agregados solo para test, si no se configuran no afecta
sw_public_ssh_pass: your_pass # SW agregados solo para test, si no se configuran no afecta
sw_public_ssh_port: 2222 # SW agregados solo para test, si no se configuran no afecta
#switch privado
sw_private_ip: 123.123.123.123
sw_private_os: ios
sw_private_ssh_user: example_ssh_user
sw_private_ssh_pass: "exmaple_ssh_pass"
sw_private_ssh_port: 2222
```

Si quisiéramos volver a editar los secretos, estando parado en `sm-infraestructura` ejecutar `make secret`

Una vez que configuramos todas las variables necesarias para el proyecto, estamos en condiciones
de levantar los servicios para esto ejecutamos los siguientes commandos

```bash
make setup #crea la estructura de directorios y la red de docker
make certs # solo si estamos en producción, crea los certificados para https
make db_up # levanta la base de datos
make dev # si estamos en dev - levanta todos los servicios en modo desarrollo para poder
# debugear y modificar sin necesidad de recargar manual api ni dashboard
make prod # si estamos en prod - previamente se deb<e haber compilado la version
# de api y dashboard que se quiere iniciar.
# <<<Proceso de compilación mas adelante>>>, necesario haber creado los certs
make build-tower-cli # crea la imagen de el contenedor utilizado para enviar las configuraciones al awx
make awx-send # envía las configuraciones al awx
```

### DEPLOY DEV ENVIRONMENT

**DEV-SHORTCUT:** `make setup && make db_up && sleep 30 && make dev && make build-tower-cli && make awx-send-dev`

### DEPLOY PROD ENVIRONMENT

Para levantar producción es necesario ejecutar un comando de forma independiente ya que hay que el cliente debe agregar sus certificados para https.
**PROD-SHORTCUT-PART-1:** `make setup`

Luego debe pegar sus certificados crt y key en los directorios que especifico en la variable `certificates_folder` y con el nombre que especifico en `public_domain` mas la extencion del archivo dependiendo si es el `.crt` o `.key`, una vez prontos los certificados se procede a levantar el resto de los servicios

**PROD-SHORTCUT-PART-2:** `make db_up && sleep 30 && make prod && make build-tower-cli && make awx-send-prod`

**NOTA:** es importante respetar el orden en que se ejecutan los comandos porque cada uno depende de los anteriores.
Por ejemplo la db se debe conectar de la docker network que se crea en setup.

### Bajar un ambiente completo

Para bajar un ambiente completo es necesario ejecutar:

```bash
make _db_down # baja la base de datos y borra el volumen # ¡¡¡DANGER!!! Borra todos los datos de la BDD.
make tear_down # baja el resto de los servicios, borra los templates, borra la red docker
```

[<img src="images/backToHome.png" width="150px" height="150px"/>](../README.md)
