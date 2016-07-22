# Tangram Direct OSM Data Source

Tangram currently supports TopoJSON, MVT, and GeoJSON data sources. I'd like to see native OSM data formats supported as well: OSM XML and OSM PBF. Tangram is a great candidate library for creating an OpenStreetMap editor, however, to make this possible, Tangram needs to have first-class support of OSM data. In particular, we must be able to fetch and render OSM XML received from the OSM 0.6 Editing API.

Because Tangram handles tiled data sources well, I think the initial approach should involve fetching OSM XML from the OSM Editing API on a tile-by-tile basis. OSM editors fetch their data from the editing api for general editing from the CGImap `/map` endpoint. This REST endpoint is bounding box based, but that isn't really a problem, because we can easily get the bounding box of a tile we want data for. We can change that request to stub in the `bbox`parameters instead of `z,x,y`.

There are a couple of challenges with handling direct OSM data. In particular, current Mapzen data is built around the concept of layers. Different elements are grouped into layers such as _roads_ or _buildings_. OSM data is flat and instead is given context by tags.
