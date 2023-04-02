# Download OSM extract

https://download.geofabrik.de/north-america/us/virginia.html

# Download Population Grid

https://ghsl.jrc.ec.europa.eu/download.php?ds=pop

Select GHS-POP 2020 100m Mollweide

# Set up

## PostGIS

Start up PostGIS

```
cd osm2pgsql
docker compose up -d
```

### osm2pgsql

Install the osm2pgsql tool:

```
apt install osm2pgsql
```

Import the data. This will take a few minutes or many hours depending on the size and your hardware.

```
# replace /path/to/extract.osm.pbf with the actual file path, e.g. ~/Downloads/virginia-latest.osm.pbf
PGPASSWORD=password osm2pgsql --create --verbose -P 15432 -U postgres -H localhost -S osm2pgsql.style /path/to/extract.osm.pbf
```

## Valhalla

Copy osm extract to `valhalla_data/`

```
cd ..
cp /path/to/extract.osm.pbf valhalla_data/
```

Run valhalla with:

```
docker run -dt -v $PWD/valhalla_data:/custom_files -p 8002:8002 --name valhalla gisops/valhalla:latest
```

Then run
```
docker logs -f valhalla
```

until you see the line `INFO: Found config file. Starting valhalla service!`

# Find the hospitals and all isochrone data for them

Retrieves isochrones within distance of all hospitals.

```
python3 query_hospitals.py
```

## Convert isochrone polygons to mbtiles

### Install tippecanoe

Follow the instructions here: https://github.com/felt/tippecanoe

Convert the geoJson polygons to mbtiles. `--simplification 6` is needed for displaying
large isochrone polygons in mapLibreGL.
Otherwise we hit the maximum vertices available in WebGL (65535) at low zoom levels.
This also helps reduce the size of the tiles.

```
tippecanoe --simplification 6 -f polygon_0.json -f polygon_1.json -f polygon_2.json -f polygon_3.json -o virginia_iso.mbtiles
```

### Install go-pmtiles

https://github.com/protomaps/go-pmtiles

### Convert mbtiles to PMTiles format

```
git clone https://github.com/protomaps/go-pmtiles
cd go-pmtiles/
go install
go run main.go convert ../virginia_iso.mbtiles ../virginia_iso.pmtiles
cd ..
```

# Generating a base map with planetiler

```
cp /path/to/extract.osm.pbf planetiler_data/

docker run -e JAVA_TOOL_OPTIONS="-Xmx4g" -v "$(pwd)/planetiler_data":/data ghcr.io/onthegomap/planetiler:latest --download --osm-path=/data/virginia-latest.osm.pbf --output /data/extract.pmtiles --maxzoom=8
```

This will automatically create a tiles file `planetiler_data/extract.pmtiles` in the PMTiles format.
No conversion is necessary.

# Copy these files to a public directory or upload them to a static hosting service.

```

```

# population.py

Examines population data within a polygon.


