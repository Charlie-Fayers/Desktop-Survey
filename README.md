Ecological Screening Tool — Setup Guide
What it does
User clicks Select boundary and then clicks a red line boundary feature on the map
A geodesic buffer is drawn at the chosen distance (default 2 km)
The tool queries every configured ecological layer in parallel using `queryFeatures()`
A plain-English paragraph is generated summarising all intersecting features, ready to paste into a report
---
Prerequisites
ArcGIS Experience Builder 1.11 or later (Developer Edition or Online)
A web map containing:
One or more red line boundary layers (the tool auto-detects layers whose title matches `red line`, `site boundary`, or `application boundary`)
The national open access layers listed below — add only the ones you have access to
---
Installation (Experience Builder Developer Edition)
Copy the `ecological-screening-tool/` folder into:
```
   client/your-extensions/widgets/
   ```
Restart the Experience Builder dev server
The widget will appear in the widget panel under Custom Widgets
Add it to your experience and connect it to your map widget via Select map
---
Layer name matching
The tool matches layers by their title in the web map. Open `widget.js` and edit the
`ECOLOGICAL_LAYERS` array at the top of the file to match your exact layer titles:
```js
const ECOLOGICAL_LAYERS = [
  {
    title: "Sites of Special Scientific Interest (SSSI)",  // ← must match your layer title exactly
    shortName: "Sites of Special Scientific Interest (SSSIs)",
    nameField: "SSSI_NAME",   // ← attribute field containing the designation name
    type: "statutory",
  },
  // ... add or remove layers as needed
];
```
Field names for common national datasets
Dataset	Typical title in ArcGIS Online	Name field
SSSI (England)	Sites of Special Scientific Interest (SSSI)	`SSSI_NAME`
SAC (GB)	Special Areas of Conservation (SAC)	`SAC_NAME`
SPA (GB)	Special Protection Areas (SPA)	`SPA_NAME`
Ramsar (GB)	Ramsar Sites	`RAMSAR_NAME`
LNR (England)	Local Nature Reserves (LNR)	`LNR_NAME`
Ancient Woodland (England)	Ancient Woodland Inventory	`NAME`
Priority Habitat Inventory	Priority Habitat Inventory (PHI)	`Main_Habit`
iNaturalist	iNaturalist Research Grade Observations	`species_name`
Set `groupByField: true` for the Priority Habitat layer so the report lists habitat types rather than individual polygons.
---
Buffer behaviour
Buffer is applied using `geometryEngine.geodesicBuffer()` so it is accurate for large areas
The boundary geometry is projected to British National Grid (WKID 27700) before buffering to ensure metric accuracy
The buffer displays on the map as a dashed green outline; it is removed on Reset
---
Report output format
The generated paragraph follows this structure:
> A 2 km buffer was applied to the red line boundary of the application site and interrogated against national open access ecological datasets on [date].
> The following statutory nature conservation designations were recorded within the 2 km buffer: [count] Sites of Special Scientific Interest ('Name A', 'Name B') and [count] Special Areas of Conservation ('Name C').
> In terms of designated habitats, [count] Priority Habitat Inventory parcels encompassing 'Lowland meadows', 'Lowland calcareous grassland' and 3 others were identified; [count] parcels of ancient woodland were recorded ('Name D', 'Name E').
> A total of [count] iNaturalist biodiversity records were retrieved within the buffer area, including records of 'Species A', 'Species B' and 3 others.
> No Ramsar wetland sites, Local Nature Reserves (LNRs) or Special Protection Areas (SPAs) were identified within the search area.
---
Extending the tool
Adding more layers
Add entries to the `ECOLOGICAL_LAYERS` array. Supported `type` values control which
paragraph section the layer is reported under:
`"statutory"` — statutory designations paragraph
`"habitat"` — habitats paragraph
`"records"` — biodiversity records paragraph
Adding distance to nearest feature
To report the distance from the boundary to each designation (rather than just intersection),
change the query's `spatialRelationship` to `"intersects"` and post-process the results
using `geometryEngine.distance()` on each returned feature.
Exporting to Word / PDF
Connect the `reportText` state to a download function using the
FileSaver.js library (already available in ExB)
to export as `.txt`, or use the `docx` npm package to produce a formatted Word paragraph.
---
Web AppBuilder adaptation
If you are using Web AppBuilder (Classic) rather than Experience Builder, the same
logic applies but the component structure changes:
Create a WAB custom widget folder with `Widget.js`, `Widget.html`, `manifest.json`
Replace the React JSX with Dijit/AMD equivalents
Use `esri/tasks/QueryTask` and `esri/tasks/support/Query` directly (already AMD-compatible)
The `buildReportParagraph()` function is framework-agnostic and can be copied as-is
---
Known limitations
The tool queries feature services only; map image layers (dynamic) are not supported.
Convert any relevant dynamic layers to hosted feature layers in your portal first.
Maximum of 200 features returned per layer query. For very large datasets (e.g. PHI),
increase `num` in the Query or implement pagination.
Projection to BNG requires the geometry service to be configured in your ArcGIS portal.
