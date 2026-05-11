# In-browser rendering

> Render multi-terabyte datasets in the browser. Stream Zarr chunks straight to the GPU. No server, no pre-rendered tiles.

## What it does

Zarrita.js reads Zarr chunks directly from the browser using `fetch`. The `@developmentseed/deck.gl-zarr` `ZarrLayer` pairs the array's native chunk grid with deck.gl's `TileLayer`, reads the GeoZarr `multiscales` and `geo-proj` conventions to drive level selection, and reprojects each chunk into Web Mercator on the GPU. The user pans, zooms, and recolors a multi-terabyte dataset interactively, with no server-side rendering pipeline and no pre-rendered tile cache.

## Why it matters

- **No tiling pipeline:** skip the offline pre-render, the tile cache, and the tile server
- **Cloud-optimized:** the data lives in Zarr on object storage, the renderer reads it directly
- **Laptop-grade performance:** the GPU does the heavy lifting

## Live demos

### ECMWF 15-day ensemble forecast

A 15-day ECMWF ensemble forecast from [Dynamical.org](https://dynamical.org/), stored as a single Zarr v3 dataset with consolidated metadata, rendered chunk-by-chunk in the browser. Pan, zoom, scrub the lead time, swap colormaps, and adjust the value range, all client-side.

[![ECMWF 15-day forecast rendered with deck.gl-zarr](/figures/deck-gl-ecmwf.png)](https://developmentseed.org/deck.gl-raster/examples/dynamical-zarr-ecmwf/)

[See the deck.gl-zarr demo ↗](https://developmentseed.org/deck.gl-raster/examples/dynamical-zarr-ecmwf/)

### AlphaEarth Foundations mosaic

A continental GeoZarr mosaic of [AlphaEarth Foundations](https://source.coop/repositories/tge-labs/aef-mosaic/) satellite embeddings, served from `s3://us-west-2.opendata.source.coop/tge-labs/aef-mosaic/`. Nine annual snapshots of 64-dimensional embeddings at ~10 m resolution, rendered as a configurable RGB composite. All 64 bands of the selected year are uploaded per tile as a single GPU texture array, so swapping bands is a uniform change with no refetch, and dequantization happens in the shader.

[![AlphaEarth Foundations mosaic rendered with deck.gl-zarr](/figures/deck-gl-aef.png)](https://developmentseed.org/deck.gl-raster/examples/aef-mosaic/)

[See the deck.gl-zarr demo ↗](https://developmentseed.org/deck.gl-raster/examples/aef-mosaic/)

## How to use it

```ts
import { ZarrLayer } from '@developmentseed/deck.gl-zarr'
import * as zarr from 'zarrita'

const store = new zarr.FetchStore('https://example.com/dataset.zarr')
const node = await zarr.open.v3(store, { kind: 'group' })

const layer = new ZarrLayer({
  id: 'zarr-layer',
  node,
  // One entry per non-spatial dim. `null` = default slice,
  // a number pins an index, a `zarr.Slice` selects a range.
  selection: { time: 0, band: null },
  getTileData,   // (arr, { sliceSpec }) => zarr.get(arr, sliceSpec)
  renderTile,    // (data) => ImageData | RenderTileResult
})
```

`ZarrLayer` reads the GeoZarr conventions to derive the level pyramid, per-level affine transforms, and CRS, then leaves the data-to-pixel step to your `renderTile` callback so you can pick any colormap or composite.

## Learn more

- [`@developmentseed/deck.gl-zarr` docs](https://developmentseed.org/deck.gl-raster/api/deck-gl-zarr/)
- [`@developmentseed/deck.gl-raster` docs](https://developmentseed.org/deck.gl-raster/api/deck-gl-raster/)
- [GeoZarr spec](https://geozarr.org/)
- [Zarrita.js](https://zarrita.dev/)
- [deck.gl-raster examples](https://github.com/developmentseed/deck.gl-raster/tree/main/examples)
