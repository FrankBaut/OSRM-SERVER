# OSRM-SERVER
Crear un servidor de OSRM usando docker, y lanzarlo localmente


Instaladocker siguiendo las instrucciones(Version de ubuntu)
- instrucciones en https://docs.docker.com/install/

Crear una carpeta de trabajo, en ella se guaradaran todos los archivos descargados
** mkdir -p /data/osrm/Mexico
** cd /data/osrm/Mexico

Descargamos los datos de mapas que provienene de openstreetmap, una de las bases de datos es geofabrik,
en terminal y en la ruta creada, ejecutamos:
** wget https://download.geofabrik.de/north-america/mexico-latest.osm.pbf

OSRM requiere un procesamiento previo de los datos extraidos de OpenStreetMap, construyendo un modelo optimizado de la grilla de calles. Para la optimización se debe elegir un modo de transporte. Las opciones disponibles por default son - car - foot - bicycle

Hacemos la extracción para el modo automóvil (car) con:

** docker run -t -v $(pwd):/data osrm/osrm-backend osrm-extract -p /opt/car.lua /data/argentina-latest.osm.pbf

Esto inicia una instancia de docker, corre el comando osrm-extract y luego desactiva la instacia.

La opción -v $(pwd):/data crea un directorio /data dentro del contenedor de docker, y muestra en el el contenido del directorio desde donde corrimos el comando; es decir, el directorio donde está descargada la data de Argentina. Así es como queda accesible para la instancia de docker como /data/argentina-latest.osm.pbf.

Cuando corremos el comando por primera vez tomará unos cuantos minutos, ya que docker necesita descargar antes el container con OSRM. En lo sucesivo no será necesario, ya que tras la descaga guarda una copia local del container.

Quedan dos operaciones pendientes que OSRM necesita antes de poder realizar ruteos.

Dividir el grafo en celdas, ejecutando:

** docker run -t -v $(pwd):/data osrm/osrm-backend osrm-partition /data/mexico-latest.osrm

Obsérvese que ahora usamos argentina-latest.osrm (ya no es “.pbf”), un archivo que quedó en el directorio de trabajo tras ejecutar el paso anterior.


… y por últumo asignar “peso” a cada celda del grafo con

** docker run -t -v $(pwd):/data osrm/osrm-backend osrm-customize /data/mexico-latest.osrm



Con los datos ya preparados, sólo resta iniciar el servicio de ruteo:

** docker run -t -i -p 5000:5000 -v $(pwd):/data osrm/osrm-backend osrm-routed --algorithm mld /data/argentina-latest.osrm

Con el comando anterior, estaremos lanzando OSRM localmente



Por ejemplo, para llamar al servidor desde R, hacemos
** library(tidyverse)
** library(osrm)

Por default, el puerto es:
** options(osrm.server = "http://127.0.0.1:5000/")



Para trabajar con OSRM, hay otros paquetes que puedes usar el contenedor de docker de OSRM como:

** stplanr
** osrm
** osrmr








