# Shadowmap

**Provided by:** VC Map Project (virtualcitySYSTEMS)

## Description

Shadowmap is a VC Map plugin for analyzing the temporal shading of a selected area in a Cesium 3D scene. For a selected date, it samples the scene between estimated sunrise and sunset and reports the proportion of shaded and unshaded pixels within a polygon.

It is intended for site-, block-, and neighbourhood-scale analysis supporting urban planning, landscape architecture, climate adaptation, and GIS workflows.

## Installation Prerequisites

- VC Map UI 6.x with an active Cesium/WebGL map.
- Visible terrain or 3D objects that can cast shadows.
- `@vcmap/core` and `@vcmap-cesium/engine`, supplied by the host application.
- `@vcmap/ui` for the plugin interface.
- **For optional MinIO uploads or IDRA catalogue registration:** reachable and correctly configured network services.

## Installation Instructions

The provided documentation does not include host-specific deployment steps.

For local setup and development:

1. Install dependencies with `npm install`.
2. Start the VC Map plugin development server with `npm run start`.
3. Build the plugin with `npm run build`. For additional development and validation commands, see [Development](#development).

## Built Image Registry

Not specified in the provided documentation.

## License

This project is licensed under the MIT License. See [LICENSE.md](LICENSE.md).

Copyright 2025 tadolphi tadolphi@vc.systems

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the “Software”), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

## External technical resources

- [VC Map project](https://github.com/virtualcitySYSTEMS/map-ui)

## User Guide References

No separate user guide or FAQ links were provided. Usage instructions are included under [Additional Information](#additional-information).

## Additional Information

### Maturity

**Beta**

### Features

- Analyze shadows for a selected calendar date from sunrise to sunset.
- Sample the scene every 15, 30, or 60 minutes.
- Select the calculation area by drawing a rectangle or polygon on the map, or enter a reusable polygon as WGS84 coordinate pairs.
- Calculate shadow percentages, pixel counts, and area values for each time point.
- Display time series, charts, summary values, and result tables.
- Classify the selected area from 09:00 to 15:00 UTC as always shaded, partially shaded, or always sunny.
- Display the accumulated classification as a georeferenced map overlay.
- Export PDF and CSV reports.
- Optionally export RGB GeoTIFF results in EPSG:3857, including world files and an SLD style where applicable.
- Optionally upload reports, parameters, geometries, and raster results to MinIO.
- Optionally register uploaded files as an IDRA dataset with DCAT-AP distributions.

### Using the plugin

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
7. Review the charts, time series, area values, and accumulated 09:00–15:00 UTC classification.
8. Export the results locally or use the optional MinIO upload workflow.

Entered polygon coordinates must be WGS84 longitude/latitude pairs in the form `[longitude, latitude]`.

### Configuration

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `allowCatalogueRegistry` | boolean | `false` | Enable registration of uploaded results in the IDRA catalogue. |
| `catalogueEndpoint` | string | `https://urbreath.virtualcitymap.de/xxx/api/` | IDRA dataset API endpoint. |
| `allowMinioUpload` | boolean | `false` | Enable the MinIO upload action in the result window. |
| `allowGeotiffExport` | boolean | `false` | Enable GeoTIFF export in the result window. |
| `minioEndpoint` | string | `https://urbreath.virtualcitymap.de/xxx` | MinIO proxy endpoint. |
| `minioBucketName` | string | `vcs-analysis` | Bucket used for uploaded analysis files. |

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

The `xxx` and `xxxx` values are placeholders. The host application must provide the correct service endpoints and permissions when MinIO or catalogue integration is enabled.

### Outputs

Depending on the selected export and upload options, Shadowmap can produce:

- A PDF report containing maps, charts, tables, and optional scene images.
- CSV files containing time-based results and summary values.
- A JSON summary of the analysis.
- A JSON file containing the analysis parameters.
- The selected calculation area as GeoJSON.
- RGB GeoTIFF files for individual analysis times.
- An accumulated sunny-area GeoTIFF with an accompanying world file and optional SLD style.
- An optional IDRA dataset referencing uploaded distributions.

### Method

Sunrise and sunset are estimated mathematically from the selected date and map location. The calculation uses solar declination, solar noon, and the solar hour angle:

```math
\text{Solar declination} = 23.45 \cdot \sin\left(\frac{360 \cdot (284 + \text{dayOfYear})}{365}\right)

\text{Hour angle} = \arccos\left(-\tan(\text{latitude}) \cdot \tan(\text{declination})\right)

\text{Solar noon} = 12 - \frac{\text{longitude}}{15} \text{ hours UTC}

\text{Sunrise} = \text{solar noon} - \text{hour angle}

\text{Sunset} = \text{solar noon} + \text{hour angle}
```

The shadow ratio is derived by comparing rendered WebGL pixels with and without the Cesium shadow map for the selected area. Results therefore depend on the current rendering state of the scene.

### Limitations

- Results are based on rendered pixels rather than an independent geometric or physical shadow simulation.
- Shadow detection uses heuristic pixel-difference thresholds and has not been validated as a legal, normative, or physically exact calculation.
- Results depend on camera position, canvas resolution, visible layers, scene visibility, and render state.
- The accumulated sunny-area classification uses a fixed 09:00–15:00 UTC window; it is not automatically converted to local time.
- Time-zone handling recognizes only selected hard-coded European coordinate ranges and falls back to `Europe/Brussels` elsewhere.
- Weather, clouds, atmospheric effects, diffuse light, vegetation transmission, reflections, and thermal comfort effects are not modeled.
- The analysis requires a visible Cesium 3D scene and relevant shadow-casting geometry.
- Runtime and memory use increase with canvas size, polygon size, and shorter sampling intervals.
- Large city- or region-wide analyses are outside the intended scope of a single run.
- Internal Cesium and WebGL properties are used for pixel analysis and may change between platform versions.
- GeoTIFF files represent rendered RGB pixels in the polygon extent; they are not independent geometric shadow simulations.

Shadowmap is a planning and visualization aid. Legal or regulatory daylight and shading assessments require independent professional validation.

### Development

Run the available checks with:

```bash
npm install
npm run type-check
npm run lint
npm test
```

Build the plugin:

```bash
npm run build
```

Start the local VC Map plugin development server:

```bash
npm run start
```

Other useful commands include:

```bash
npm run bundle
npm run preview
npm run coverage
```
