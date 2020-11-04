# OSRM-Docker Container

Here is the process to launch an OSRM (Open Source Routig Machine) server using a Docker container. 
The way to build it is detailed as follows.

Install Docker following the instructions (Ubuntu version)

instructions in https://docs.docker.com/install/

Create a work folder(Mexico in this case), in it all files will be saved
downloaded:

- mkdir -p /data/osrm/Mexico
- cd /data/osrm/Mexico

We downloaded the data of the roads of Mexico in Openstreetmap.
A repository of the databases is geofabrik, in terminal and in the previously created path, we execute:

- wget https://download.geofabrik.de/north-america/mexico-latest.osm.pbf

OSRM needs a pre-processing of the data extracted from Openstreetmap, 
building an optimized model of the street network. For optimization, a means of transport must be chosen. 
The options available by default are: car, foot, bicycle.

We do the extraction for the automobile mode (car) with:

- docker run -t -v $(pwd):/data osrm/osrm-backend osrm-extract -p
/opt/car.lua /data/mexico-latest.osm.pbf

This starts a Docker instance, runs the osrm-extract command, and then deactivates the instance.

The -v $ (pwd):/data option creates a /data directory inside the Docker container,
and displays the contents of the directory from where we ran the command; that is,
the directory where the data for Mexico is downloaded. This is how it is accessible to the Docker instance:

- /data/mexico-latest.osm.pbf.

When we run the command for the first time it will take a few minutes, since Docker needs to download the container with OSRM first. Later it will not be necessary, since after the download it saves a local copy of the container. There are two pending operations that OSRM needs before it can perform routes. Divide the graph into cells, executing:

- docker run -t -v $(pwd):/data osrm/osrm-backend osrm-partition /data/mexico-latest.osrm

Notice that we now use mexico-latest.osrm (it is no longer ".pbf"), a file that remained in the working directory after executing the previous step and finally: Assign ’weight’ to each cell of the graph with:

- docker run -t -v $(pwd):/data osrm/ osrm-backend osrm-customize /data/mexico-latest.osrm

With the data already prepared, it only remains to start the routing service:

- docker run -t -i -p 5000:5000 -v $(pwd):/data osrm/osrm-backend osrm-routed –algorithm mld /data/mexico-latest.osrm

With the above command, we will be launching OSRM locally to query it.

For example, to call the server from R
- library(tidyverse)
- library(osrm)

By default, the port is:

- options(osrm.server = "http://127.0.0.1:5000/")

To work with OSRM, there are other packages that you can use the OSRM docker container like:

- stplanr
- osrm
- osrmr
