A continuación presentamos el proceso para lanzar un servidor OSRM(Open Source
Routig Machine) mediante un contenedor de Docker. La forma de construirlo se
detalla como sigue.

Instala Docker siguiendo las instrucciones(versión de Ubuntu)

instrucciones en https://docs.docker.com/install/

Crear una carpeta de trabajo, en ella se guardaran todos los archivos
descargados:

- mkdir -p /data/osrm/Mexico
- cd /data/osrm/Mexico

Descargamos los datos de las carretas de Mexico en Openstreetmap. Un
repositorio de las bases de datos es geofabrik, en terminal y en la ruta
anteriormente creada, ejecutamos:

- wget https://download.geofabrik.de/north-america/mexico-latest.osm.pbf

OSRM necesita un pre-procesamiento de los datos extraídos de Openstreetmap,
construyendo un modelo optimizado de la red de calles. Para la optimización se
debe elegir un medio de transporte. Las opciones disponibles por default son: car,
foot, bicycle.

Hacemos la extracción para el modo automóvil(car) con:

- docker run -t -v $(pwd):/data osrm/osrm-backend osrm-extract -p
/opt/car.lua /data/mexico-latest.osm.pbf

Esto inicia una instancia de Docker, corre el comando osrm-extract y luego
desactiva la instancia.

La opción -v $(pwd):/data crea un directorio /data dentro del contenedor de
Docker, y muestra el contenido del directorio desde donde corrimos el comando;
es decir, el directorio donde están descargados los datos de México. Así es como
queda accesible para la instancia de Docker:
- /data/mexico-latest.osm.pbf.

Cuando corremos el comando por primera vez tomará unos cuantos minutos, ya
que Docker necesita descargar antes el contenedor con OSRM. Posteriormente no
será necesario, ya que tras la descarga guarda una copia local del contenedor.
Quedan dos operaciones pendientes que OSRM necesita antes de poder realizar
ruteos. Dividir el grafo en celdas, ejecutando: 
- docker run -t -v $(pwd):/data
osrm/osrm-backend osrm-partition /data/mexico-latest.osrm
Observe que ahora usamos mexico-latest.osrm (ya no es ".pbf"), un archivo que
quedó en el directorio de trabajo tras ejecutar el paso anterior y por último:
Asignar ’peso’ a cada celda del grafo con:
- docker run -t -v $(pwd):/data osrm/ osrm-backend osrm-customize /data/mexico-latest.osrm

Con los datos ya preparados, sólo resta iniciar el servicio de ruteo:

- docker run -t -i -p 5000:5000 -v $(pwd):/data osrm/osrm-backend osrm-routed –algorithm mld /data/mexico-latest.osrm

Con el comando anterior, estaremos lanzando OSRM localmente para realizar
consultas en él.
