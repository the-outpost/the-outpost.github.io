# Tangram ES

<!-- Place this tag where you want the button to render. -->
<a class="github-button" href="https://github.com/tangrams/tangram-es" data-style="mega" data-count-href="/tangrams/tangram-es/stargazers" data-count-api="/repos/tangrams/tangram-es#stargazers_count" data-count-aria-label="# stargazers on GitHub" aria-label="Star tangrams/tangram-es on GitHub">Star</a> Company: [Mapzen, Samsung Research America, Inc.](https://mapzen.com/)

## Tips and Tricks

### Switching between MVT and GeoJSON tile sources

Press `G`.

#### [main.cpp](https://github.com/tangrams/tangram-es/blob/fd48ed73de4147620c2d811c6fbc3084f2cbb7d8/osx/src/main.cpp#L227-L240)
```cpp
case GLFW_KEY_G:
    static bool geoJSON = false;
    if (!geoJSON) {
        LOGS("Switching to GeoJSON data source");
        Tangram::queueSceneUpdate("sources.osm.type", "GeoJSON");
        Tangram::queueSceneUpdate("sources.osm.url", "https://vector.mapzen.com/osm/all/{z}/{x}/{y}.json");
    } else {
        LOGS("Switching to MVT data source");
        Tangram::queueSceneUpdate("sources.osm.type", "MVT");
        Tangram::queueSceneUpdate("sources.osm.url", "https://vector.mapzen.com/osm/all/{z}/{x}/{y}.mvt");
    }
    geoJSON = !geoJSON;
    Tangram::applySceneUpdates();
    break;
```

## data

Let's get started looking at the `src/data` directory. As you'd imagine, this is where we handle data--data for a tile, data sources, and the overall structure in which the data is organized.

[tileData.h](https://github.com/tangrams/tangram-es/blob/master/core/src/data/tileData.h) is the container for tile data. [GeoJsonSource](https://github.com/tangrams/tangram-es/blob/master/core/src/data/geoJsonSource.cpp), [MVTSource](https://github.com/tangrams/tangram-es/blob/master/core/src/data/mvtSource.cpp), and [TopoJsonSource](https://github.com/tangrams/tangram-es/blob/master/core/src/data/topoJsonSource.cpp). 

`tileData.h` is a collection of 3 structs. 

```cpp
struct Feature {
    Feature() {}
    Feature(int32_t _sourceId) { props.sourceId = _sourceId; }

    GeometryType geometryType = GeometryType::polygons;

    std::vector<Point> points;
    std::vector<Line> lines;
    std::vector<Polygon> polygons;

    Properties props;
};

struct Layer {

    Layer(const std::string& _name) : name(_name) {}

    std::string name;

    std::vector<Feature> features;

};

struct TileData {

    std::vector<Layer> layers;

};
```

Inside all of these sources, we add to the list of layers in the `tileData`. For example, in [geoJsonSource.cpp](https://github.com/tangrams/tangram-es/blob/master/core/src/data/geoJsonSource.cpp#L53-L54) we do this:

```cpp
for (auto layer = document.MemberBegin(); layer != document.MemberEnd(); ++layer) {
    if (GeoJson::isFeatureCollection(layer->value)) {
        tileData->layers.push_back(GeoJson::getLayer(layer->value, projFn, m_id));
        tileData->layers.back().name = layer->name.GetString();
    }
}
```

Looking at [topoJsonSource.cpp](https://github.com/tangrams/tangram-es/blob/b11adf00bb2b47921982c4e7932b2bbc27a0f2ed/core/src/data/topoJsonSource.cpp#L55-L57), we also do basically the same thing:

```cpp
for (auto layer = objects.MemberBegin(); layer != objects.MemberEnd(); ++layer) {
    tileData->layers.push_back(TopoJson::getLayer(layer, topology, m_id));
}
```

Since `tileData` is really just struct containers, the creation of actual tileData is found in utility classes, as you see above. 

## Creating a Layer

`GeoJson::getLayer` has a callback function fed to it called `projFn`. This function projects the LatLngs to points within the coordinate space of the tile. The exact same function is also used for `TopoJson::getLayer`.

```cpp
BoundingBox tileBounds(_projection.TileBounds(task.tileId()));
glm::dvec2 tileOrigin = {tileBounds.min.x, tileBounds.max.y*-1.0};
double tileInverseScale = 1.0 / tileBounds.width();

const auto projFn = [&](glm::dvec2 _lonLat){
    glm::dvec2 tmp = _projection.LonLatToMeters(_lonLat);
    return Point {
        (tmp.x - tileOrigin.x) * tileInverseScale,
        (tmp.y - tileOrigin.y) * tileInverseScale,
         0
    };
};
```

This [coordinate space](https://github.com/tangrams/tangram-es/blob/b11adf00bb2b47921982c4e7932b2bbc27a0f2ed/core/src/data/tileData.h#L17-L22), as you can see, is simple:

```
  (0.0, 1.0) ---------- (1.0, 1.0)
            |          |             N
            ^ +y       |          W <|> E
            |          |             S
            |    +x    |
  (0.0, 0.0) ----->---- (1.0, 0.0)
```

`TopoJson::getLayer` gets a topology object which has already had [points within the topology projected](https://github.com/tangrams/tangram-es/blob/b11adf00bb2b47921982c4e7932b2bbc27a0f2ed/core/src/data/topoJsonSource.cpp#L49).

## Layers and Styles

Being that we have layers in tile data, we need to study how they are styled and rendered as specified in `scene.yml`.

[TileData is created](https://github.com/tangrams/tangram-es/blob/b11adf00bb2b47921982c4e7932b2bbc27a0f2ed/core/src/tile/tileTask.cpp#L19) by TileTask in `TileTask::process`.

```cpp
void TileTask::process(TileBuilder& _tileBuilder) {

    auto tileData = m_source->parse(*this, *_tileBuilder.scene().mapProjection());

    if (tileData) {
        m_tile = _tileBuilder.build(m_tileId, *tileData, *m_source);
    } else {
        cancel();
    }
}
```

### TileBuilder

[TileBuilder.h](https://github.com/tangrams/tangram-es/blob/b11adf00bb2b47921982c4e7932b2bbc27a0f2ed/core/src/tile/tileBuilder.h) is the class where we build the actual tile from the tile data with the `StyleBuilder`.

`TileBuilder::build` is the primary member function we care about that returns a `Tile`.

```cpp
std::shared_ptr<Tile> TileBuilder::build(TileID _tileID, const TileData& _tileData, const DataSource& _source) {

    auto tile = std::make_shared<Tile>(_tileID, *m_scene->mapProjection(), &_source);

    tile->initGeometry(m_scene->styles().size());

    m_styleContext.setKeywordZoom(_tileID.s);

    for (auto& builder : m_styleBuilder) {
        if (builder.second)
            builder.second->setup(*tile);
    }

    for (const auto& datalayer : m_scene->layers()) {

        if (datalayer.source() != _source.name()) { continue; }

        for (const auto& collection : _tileData.layers) {

            if (!collection.name.empty()) {
                const auto& dlc = datalayer.collections();
                bool layerContainsCollection =
                    std::find(dlc.begin(), dlc.end(), collection.name) != dlc.end();

                if (!layerContainsCollection) { continue; }
            }

            for (const auto& feat : collection.features) {
                m_ruleSet.apply(feat, datalayer, m_styleContext, *this);
            }
        }
    }

    for (auto& builder : m_styleBuilder) {
        tile->setMesh(builder.second->style(), builder.second->build());
    }

    return tile;
}
```

This build is done during [TileTask::process](https://github.com/tangrams/tangram-es/blob/b11adf00bb2b47921982c4e7932b2bbc27a0f2ed/core/src/tile/tileTask.cpp#L22).

Note that `builder.second` is getting the value (as opposed to the key) of the `m_styleBuilder`, which is a hash map implemented via a class called [`fastmap`](https://github.com/tangrams/tangram-es/blob/b11adf00bb2b47921982c4e7932b2bbc27a0f2ed/core/src/util/fastmap.h).


### StyleBuilder



## scene.yaml

The most general-purpose, and useful (in my opinion) style file we have is the Cinnabar style. The [scene.yaml for Cinnabar](https://tangrams.github.io/carousel/styles/cinnabar-style.yaml) is pretty long and extensive, and this is my main reference for seeing how to do in-depth styling.

Mapzen has good documentation explaining [how the scene file works](https://mapzen.com/documentation/tangram/Scene-file/).

### Loading scene.yml

[scene.yaml](https://github.com/tangrams/tangram-es/blob/b11adf00bb2b47921982c4e7932b2bbc27a0f2ed/scenes/scene.yaml) is loaded by `void loadScene(const char* _scenePath)`. This is called explicity as well as in `void initialize(const char* _scenePath)`.

```cpp
void loadScene(const char* _scenePath) {
    LOG("Loading scene file: %s", _scenePath);

    auto sceneString = stringFromFile(setResourceRoot(_scenePath).c_str(), PathType::resource);

    // Copy old scene
    auto scene = std::make_shared<Scene>(*m_scene);

    if (SceneLoader::loadScene(sceneString, *scene)) {
        setScene(scene);
    }
}
```

This non-class function in turn calls [`SceneLoader::loadScene`](https://github.com/tangrams/tangram-es/blob/b11adf00bb2b47921982c4e7932b2bbc27a0f2ed/core/src/scene/sceneLoader.cpp#L48-L57), and then the resultant scene object is set. The scene.yaml is loaded in [parse.cpp](https://github.com/tangrams/yaml-cpp/blob/master/src/parse.cpp) within the [yaml-cpp submodule](https://github.com/tangrams/tangram-es/blob/a7d7494aa8ee464872629b352f112782e25bf49e/.gitmodules#L22-L24). The [yaml-cpp](https://github.com/tangrams/yaml-cpp) project is maintained by Mapzen tangrams. The actual loading is making a new root node object of the YAML document. It appears that yaml-cpp is a DOM style parser.

### Applying scene.yaml

Once we have loaded `scene.yaml`, we need to apply it.

```cpp
bool SceneLoader::loadScene(const std::string& _sceneString, Scene& _scene) {

    Node& root = _scene.config();

    if (loadConfig(_sceneString, root)) {
        applyConfig(root, _scene);
        return true;
    }
    return false;
}
```

[`SceneLoader::applyConfig`](https://github.com/tangrams/tangram-es/blob/a7d7494aa8ee464872629b352f112782e25bf49e/core/src/scene/sceneLoader.cpp#L152-L270) is a long member function that does all of the work of setting up the [Scene](https://github.com/tangrams/tangram-es/blob/a7d7494aa8ee464872629b352f112782e25bf49e/core/src/scene/scene.h) object. This function is paricularly relevent, because you can see how the yaml file is read and used to setup the scene object. It goes through all of the top-level scene elements, and these elements are specified in the [Scene File documentation](https://mapzen.com/documentation/tangram/Scene-file/#top-level-elements).

### Loading a Source

[`SceneLoader::loadSource`](https://github.com/tangrams/tangram-es/blob/a7d7494aa8ee464872629b352f112782e25bf49e/core/src/scene/sceneLoader.cpp#L705-L784) is where we setup our data sources. The documentation states that data sources can be tiled or non-tiled. In specfic, we can see that this support only applies currently for GeoJSON. If we are to make an OSM_XML source type, we'll want to follow the same mechanism for accepting tiled and non-tiled OSM sources.

## Label Placement

https://mapzen.com/documentation/tangram/Filters-Overview/#label_placement

<script async defer src="https://buttons.github.io/buttons.js"></script>



