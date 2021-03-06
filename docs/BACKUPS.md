# Crear Respaldos <!-- omit in toc -->

Creación de los respaldos necesarios para preparar el sistema para disaster recovery a ese punto

- [Respaldar project_secret y variables de entorno](#respaldar-project_secret-y-variables-de-entorno)
- [Respaldar archivos de configuración](#respaldar-archivos-de-configuración)
- [Respaldar bases de datos](#respaldar-bases-de-datos)

## Respaldar project_secret y variables de entorno

Si se hizo un deploy estandard de la solución, las variables de entorno se encuentran en el archivo `sm-infrastructure/secret.yml` y para poder abrir dicho archivo es necesario contar con el `sm-infrastructure/password` el cual también debe respaldarse.

## Respaldar archivos de configuración

Todos los archivos de configuracion que y los certificados (asumiendo que la variables de entorno que configuran rutas relativas se usaron por defecto y que ademas se configuro correctamente la variable `project_folder`), son colocados en el directorio `sm-infrastructure/lib` por tanto esta es la carpeta que debería respaldarse.

## Respaldar bases de datos

Para respaldar las bases de datos hay que conectarse al servidor y parado en `sm-infrastructure` ejecutar

```bash
make db_backup
```

una vez ejecutado, dentro de ese directorio se debe de haber creado un archivo con el nombre `db_dump_sw_manager_<datetime_de_generado>.sql` este es el archivo a respaldar para poder reconstruir la base.

[<img src="images/backToHome.png" width="150px" height="150px"/>](../README.md)
