# Recuperación del proyecto tras perdida del servidor

Antes de comenzar con esta parte hay que asegurarse que el servidor cuenta con los requisitos y ademas el proyecto esta inicializado, tal como se describe en el [README](../README.md).

Una vez en este estado, lo siguiente es restaurar todos los archivos de configuración de variables de entorno, se debe colocar dentro de la carpeta `sm-infrastructure`
los archivos `password` y `secret.yml`

Luego se inicia el deploy ejecutando

```bash
make setup
```

Se debe remplazar la carpeta `lib` creada en setup, por la carpeta `lib` respaldada, asumiendo valores de variables de directorios por defecto y que la carpeta lib se respaldo con ese nombre, sino renombrarla al restaurarla.

Una vez hecho esto podemos levantar la db ejecutando

```bash
make db_up
```

Una vez que la db se levanto, debemos colocar el ultimo archivo de backup que tenemos en el servidor y con el comando

```bash
cat <nombre_de_su_archivo_backup>.sql | docker exec -i your-db-container psql -U postgres # nombre_de_su_archivo backup esta como variable pero por ejemplo deberia ser algo asi: db_dump_sw_manager_<datetime_del_backup>
```

Por ultimo se procede a levantar el resto de los servicios

```bash
make prod
```
