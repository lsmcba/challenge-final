# Bootcamp Final Challenge

Soluciones:

`hello-world-golang` - Golang REST endpoints

Se creo un Dockerfile para poder levantar la aplicacion, especificando el puerto 3002 para que corra la aplicacion.

`hello-world-nodejs` - Nodejs REST endpoint

Se modifico el archivo Dockerfile, agregando la linea EXPOSE y modificando la instalacion de las librerias unicamente de typescript, para reducir el tiempo de 
instalacion de las librerias necesarias para la aplicacion. 

`hello-world-nginx` - Nginx reverse proxy

Se modifico el Dockerfile, agregando la linea EXPOSE faltante. Tambien se modifico el archivo "default.conf", ya que le faltaba las lineas de upstream para 
cada servicio, y las redirecciones para la ejecucion de get-scores e inc-scores.

## Time

`Tiempo de correccion de Golang: 30 Minutos.`

`Tiempo de correccion de nodeJS: 1 Hora.`

`Tiempo de correccion de NGINX: 1 Hora.`

`Tiempo de implementacion de GitHub Action: 1 hora y 30 minutos`
