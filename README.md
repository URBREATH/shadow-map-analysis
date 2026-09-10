# Shadowmap

Shadowmap is a VC Map plugin for analyzing the temporal shading of a selected area in a Cesium 3D scene. For a selected date, it samples the scene between calculated sunrise and sunset and reports the proportion of shaded and unshaded pixels within a polygon.

The plugin is intended for site, block, and neighborhood-scale analysis in support of urban planning, landscape architecture, climate adaptation, and GIS workflows.

> **Maturity:** Beta

## Features

- Analyze shadows for a selected calendar date from sunrise to sunset.
- Sample the scene every 15, 30, or 60 minutes.
- Select the calculation area by drawing a rectangle or polygon on the map.
- Enter a reusable polygon as WGS84 coordinate pairs.
- Calculate shadow percentages, pixel counts, and area values for each time point.
- Display time series, charts, summary values, and result tables.
- Classify the selected area from 09:00 to 15:00 UTC as:
  - always shaded;
  - partially shaded; or
  - always sunny.
- Display the accumulated classification as a georeferenced map overlay.
- Export analysis reports as PDF and CSV.
- Optionally export RGB GeoTIFF results in EPSG:3857, including world files and an SLD style where applicable.
- Optionally upload reports, parameters, geometries, and raster results to MinIO.
- Optionally register uploaded files as an IDRA dataset with DCAT-AP distributions.

## Requirements

Shadowmap runs inside VC Map UI 6.x and requires:

- an active Cesium/WebGL map;
- visible terrain or 3D objects that can cast shadows;
- `@vcmap/core` and `@vcmap-cesium/engine` supplied by the host application;
- `@vcmap/ui` for the plugin interface.

MinIO uploads and IDRA catalogue registration additionally require reachable and correctly configured network services. These features do not work offline.

## Using the plugin

1. Open a VC Map application with an active Cesium 3D scene.
2. Open **Shadowmap analysis** from the VC Map toolbox.
3. Select the analysis date.
4. Select a sampling interval: 15, 30, or 60 minutes. The default is 30 minutes.
5. Define the calculation area by drawing a bounding box or polygon. Alternatively, enter polygon coordinates in WGS84, for example:

   ```json
   [
     [13.375916137434332, 52.50958736153632],
     [13.376031120070795, 52.50959004427611],
     [13.375970378742716, 52.50978645439449]
   ]
   ```

6. Start the analysis and wait for the result window to open.
7. Review the charts, time series, area values, and accumulated 09:00-15:00 UTC classification.
8. Export the results locally or use the optional MinIO upload workflow.

The entered polygon must contain WGS84 longitude/latitude pairs in the form `[longitude, latitude]`.

## Configuration

The plugin accepts the following configuration options:

| Option                   | Type    | Default                                                        | Description                                                    |
| ------------------------ | ------- | -------------------------------------------------------------- | -------------------------------------------------------------- |
| `allowCatalogueRegistry` | boolean | `false`                                                        | Enable registration of uploaded results in the IDRA catalogue. |
| `catalogueEndpoint`      | string  | `https://urbreath.virtualcitymap.de/xxx/api/` | IDRA dataset API endpoint.                                     |
| `allowMinioUpload`       | boolean | `false`                                                        | Enable the MinIO upload action in the result window.           |
| `allowGeotiffExport`     | boolean | `false`                                                        | Enable GeoTIFF export in the result window.                    |
| `minioEndpoint`          | string  | `https://urbreath.virtualcitymap.de/xxx`                | MinIO proxy endpoint.                                          |
| `minioBucketName`        | string  | `vcs-analysis`                                                 | Bucket used for uploaded analysis files.                       |

Example configuration:

```json
{
  "name": "shadowmap",
  "allowCatalogueRegistry": false,
  "allowGeotiffExport": true,
  "catalogueEndpoint": "https://urbreath.virtualcitymap.de/xxxx/api/",
  "allowMinioUpload": false,
  "minioEndpoint": "https://urbreath.virtualcitymap.de/xxxx",
  "minioBucketName": "vcs-analysis"
}
```

The host application must provide the required service endpoints and permissions when MinIO or catalogue integration is enabled.

## Outputs

Depending on the selected export and upload options, Shadowmap can produce:

- a PDF report containing maps, charts, tables, and optional scene images;
- CSV files containing time-based results and summary values;
- a JSON summary of the analysis;
- a JSON file containing the analysis parameters;
- the selected calculation area as GeoJSON;
- RGB GeoTIFF files for individual analysis times;
- an accumulated sunny-area GeoTIFF with accompanying world file and optional SLD style;
- an optional IDRA dataset referencing uploaded distributions.

## Method

Sunrise and sunset are estimated mathematically from the selected date and the map location. The calculation uses solar declination, solar noon, and the solar hour angle:

- Solar declination: `23.45 * sin(360 * (284 + dayOfYear) / 365)`
- Hour angle: `acos(-tan(latitude) * tan(declination))`
- Solar noon: `12 - longitude / 15` hours UTC
- Sunrise: solar noon minus the hour angle
- Sunset: solar noon plus the hour angle

The shadow ratio is derived by comparing rendered WebGL pixels with and without the Cesium shadow map for the selected area. This makes the result dependent on the current rendering state of the scene.

## Limitations

- Results are based on rendered pixels rather than an independent geometric or physical shadow simulation.
- Shadow detection uses heuristic pixel-difference thresholds and has not been validated as a legal, normative, or physically exact calculation.
- Results depend on camera position, canvas resolution, visible layers, scene visibility, and render state.
- The accumulated sunny-area classification uses a fixed 09:00-15:00 UTC window; it is not automatically converted to local time.
- Time-zone handling recognizes only selected hard-coded European coordinate ranges and falls back to `Europe/Brussels` elsewhere.
- Weather, clouds, atmospheric effects, diffuse light, vegetation transmission, reflections, and thermal comfort effects are not modeled.
- The analysis requires a visible Cesium 3D scene and relevant shadow-casting geometry.
- Runtime and memory use increase with canvas size, polygon size, and shorter sampling intervals.
- Large city- or region-wide analyses are outside the intended scope of a single run.
- Internal Cesium and WebGL properties are used for pixel analysis and may change between platform versions.
- GeoTIFF files represent rendered RGB pixels in the polygon extent; they are not independent geometric shadow simulations.

Shadowmap should therefore be treated as a planning and visualization aid. Legal or regulatory daylight and shading assessments require independent professional validation.

## Development

Install dependencies and run the available checks with:

```bash
npm install
npm run type-check
npm run lint
npm test
```

Build the plugin with:

```bash
npm run build
```

For local development, start the VC Map plugin development server:

```bash
npm run start
```

Other useful commands include `npm run bundle`, `npm run preview`, and `npm run coverage`.

## Project Links

- [VC Map project](https://github.com/virtualcitySYSTEMS/map-ui)

## License

This project is licensed under the MIT License. See [LICENSE.md](LICENSE.md).
