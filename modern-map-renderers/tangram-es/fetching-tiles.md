# Fetching Tiles

## [View](https://github.com/tangrams/tangram-es/blob/f1ef7d16ee07b58b696bdbf6b47e941889b32961/core/src/view/view.h)

The `View` is an object that represents the state of the viewport of the map. The view contains the set of visible tiles, and we call [`View::getVisibleTiles`](https://github.com/tangrams/tangram-es/blob/f1ef7d16ee07b58b696bdbf6b47e941889b32961/core/src/view/view.h#L157) to get this information.

## [tangram.h](https://github.com/tangrams/tangram-es/blob/f1ef7d16ee07b58b696bdbf6b47e941889b32961/core/src/tangram.h)

[tangram.cpp](https://github.com/tangrams/tangram-es/blob/f1ef7d16ee07b58b696bdbf6b47e941889b32961/core/src/tangram.cpp) is the primary API, and this is where `View::getVisibleTiles` is called. There is a `bool update(float _dt)` function that is called in a tight loop by the main function of the application. In this function, we call:

```cpp
m_tileManager->updateTileSets(viewState, m_view->getVisibleTiles());
```

In the OS X GLFW application, `bool update` is called within

```cpp
while (keepRunning && !glfwWindowShouldClose(main_window)) {
    ...
    Tangram::update(delta);
    Tangram::render();
    ...
}
```

It is exposed to Android in the JNI.

```cpp
JNIEXPORT bool JNICALL Java_com_mapzen_tangram_MapController_nativeUpdate(JNIEnv* jniEnv, jobject obj, jfloat dt) {
    return Tangram::update(dt);
}
```


## [TileManager](https://github.com/tangrams/tangram-es/blob/f1ef7d16ee07b58b696bdbf6b47e941889b32961/core/src/tile/tileManager.h)

>Singleton container of TileSets. TileManager is a singleton that maintains a set of Tiles based on the current view into the map.

The `TileManager` is instantiated one time in the setup of the app in [tangram.cpp](https://github.com/tangrams/tangram-es/blob/f1ef7d16ee07b58b696bdbf6b47e941889b32961/core/src/tangram.cpp#L83)

```cpp
m_tileManager = std::make_unique<TileManager>(*m_tileWorker);
```

### updateTileSets

Since [`TileManager::updateTileSets`](https://github.com/tangrams/tangram-es/blob/bda96bbe33146974be96785b0fd39182451e463a/core/src/tile/tileManager.cpp#L118-L138) is called repeatedly by `Tangram::update`, this is a good first member function to look at.

First, it loops through all of the tile sets and updates them. [`TileManager::updateTileSet`](https://github.com/tangrams/tangram-es/blob/bda96bbe33146974be96785b0fd39182451e463a/core/src/tile/tileManager.cpp#L140-L340) is a long and involved function. Several activities include: unsetting proxies, making sure all visible tiles are in the `TileSet`, prioritizing the tile to be loaded based on the distance from the center of the map, and making sure the download requests does not exceed the limit of downloads.

Then, [`TileManager::loadTiles`](https://github.com/tangrams/tangram-es/blob/bda96bbe33146974be96785b0fd39182451e463a/core/src/tile/tileManager.cpp#L403-L439) is called. Here, we create the tasks, and we call

```cpp
if (tileSet.source->loadTileData(std::move(task), m_dataCallback)) {
    ...
}
```

[`DataSource::loadTileData`](https://github.com/hallahan/tangram-es/blob/OSM_XML/core/src/data/dataSource.cpp#L171-L174) is the actual place where the HTTP request is made. `startUrlRequest` is a platform wrapper function that has your specific platform make the HTTP request.

```cpp
bool DataSource::loadTileData(std::shared_ptr<TileTask>&& _task, TileTaskCb _cb) {

    std::string url(constructURL(_task->tileId()));

    // lambda captured parameters are const by default, we want "task" (moved) to be non-const,
    // hence "mutable"
    // Refer: http://en.cppreference.com/w/cpp/language/lambda
    return startUrlRequest(url,
            [this, _cb, task = std::move(_task)](std::vector<char>&& rawData) mutable {
                this->onTileLoaded(std::move(rawData), std::move(task), _cb);
            });

}
```

## TileWorker

[`TileWorker`](https://github.com/tangrams/tangram-es/blob/ce2ff32f4265c88e1bd4360ea34b15634e8b2309/core/src/tile/tileWorker.cpp][`TileWorker::run`](https://github.com/tangrams/tangram-es/blob/ce2ff32f4265c88e1bd4360ea34b15634e8b2309/core/src/tile/tileWorker.cpp) is the wrapper for a thread that does the work of fetching a tile. The worker is constructed with a `TileBuilder` and a `std::thread`. This is done once during the setup of the app in [`Tangram::initialize`](https://github.com/tangrams/tangram-es/blob/f1ef7d16ee07b58b696bdbf6b47e941889b32961/core/src/tangram.cpp#L80). 

```cpp
m_tileWorker = std::make_unique<TileWorker>(MAX_WORKERS);
```

A lot happens in [`TileWorker::run`](https://github.com/tangrams/tangram-es/blob/ce2ff32f4265c88e1bd4360ea34b15634e8b2309/core/src/tile/tileWorker.cpp#L46-L119). It takes the member `m_mutex` and creates a lock. It handles all of the tasks and removes them from the queue when necessary. There is a priority mechanism for removing the correct tile from the queue. Finally, it calls `TileTask::process`.

>TODO: Figure out where `TileWorker::run` is called.

## TileTask

[TileTask::process](https://github.com/tangrams/tangram-es/blob/ce2ff32f4265c88e1bd4360ea34b15634e8b2309/core/src/tile/tileTask.cpp#L17-L26) takes `TileBuilder` and calls `GeoJsonSource::parse`.

## GeoJsonSource

`GeoJsonSource` is constructed in [`SceneLoader::loadSource`](https://github.com/tangrams/tangram-es/blob/ce2ff32f4265c88e1bd4360ea34b15634e8b2309/core/src/scene/sceneLoader.cpp#L864). This happens once.

[`GeoJsonSource::parse`](https://github.com/tangrams/tangram-es/blob/ce2ff32f4265c88e1bd4360ea34b15634e8b2309/core/src/data/geoJsonSource.cpp#L17-L64) is the final stage where we create `TileData` from a `TileTask`. This is done for each and every tile.