# Field Survey

A lightweight, offline-capable web app (PWA) for collecting roadside ground-truth
points (crop type, side L/R/B, note, photo) and a GPS track, then exporting them
for GIS or Google Earth. No app store, no server — your data stays on your device.

Quick Start: Try the app in action by visiting our live demo here: https://yailymov.github.io/field-collector/

**If you use or adapt this, please credit the author (see LICENSE).**

## Files
`index.html` · `sw.js` · `manifest.webmanifest` · `icon-192.png` · `icon-512.png` · `LICENSE` · `README.md`
Keep all of them in the **repository root**.

## Hosting (required — GPS/camera only work over https)
A local file cannot use GPS or the camera; the page must be served over https.
- **Public GitHub Pages:** push the files, then Settings → Pages → branch `main`, folder `/(root)`.
  URL: `https://<user>.github.io/field-collector/`. Only the *code* is public — your collected
  data is never uploaded and stays on your phone.
- **Keep the repo private?** Use a free host instead: drag the folder onto `app.netlify.com/drop`,
  or connect the private repo to **Cloudflare Pages** / **Vercel**.

## On the phone
1. Open the https URL **with internet** once (caches the map + app).
2. Browser menu → **Add to Home Screen** (on iPhone use **Safari**).
3. Allow location access. Works offline afterwards (for map areas you have viewed).
   On iPhone, installing to the Home Screen is what keeps stored data from being evicted.

## Test locally on a PC
Double-clicking `index.html` (file://) shows the map but **GPS, camera, the service worker and the
GeoTIFF reader will not work** (browsers block them outside a secure context). Serve the folder over
localhost instead — from the folder run:
```
python -m http.server 8000
```
then open `http://localhost:8000`. localhost counts as secure, so everything works, fully offline.

## Offline
All code and libraries (Leaflet, proj4, GeoTIFF reader) live in the repo — nothing is fetched from
the internet. Once the page has loaded once over localhost/https, the app, your data, your loaded
GeoTIFF/vector layers and the map tiles you have already viewed all work with no connection. The only
thing that needs internet is downloading map tiles for areas you have **never** opened before.

## Map layers (🗺 + layers control)
Top-right has a **layers control** (the ▣ icon): switch the base map between **OSM** and
**Satellite**, and toggle any layers you add. The 🗺 button adds layers from a file:
- **GeoTIFF** (your classified map). Colors show up only if they are **baked into the file** —
  export from QGIS as **RGB or paletted GeoTIFF, ideally a COG**. A single-band raster styled only
  via a `.qml` will render gray. Projected rasters are reprojected automatically (proj4 is loaded).
- **Vector routes / boundaries**: **KML, KMZ, GPX, GeoJSON** — e.g. the route you plan to drive,
  so you can see where to go. Each layer gets its own color and appears in the layers control.

You can pull these files from **Google Drive or Dropbox** through the file picker (if those apps are
installed). Rendering libraries load once from the internet, then are cached for offline use.
Note: very large GeoTIFFs can be heavy on a phone — prefer a COG or a smaller area.

## Collecting
- Tap the crop button → pick from the list → it closes immediately (no confirm). The list is
  ordered by how common the crop is in Ukraine; **Other…** is pinned on top for a custom name.
- Pick the side L / R / B. Crop + side stay "armed", so the big **POINT** button drops a labelled
  point at your GPS position in one tap. Each point shows its `L · Crop` label on the map.
- Long-press the map = manual point. Tap a point = edit crop/side/note, add a photo, snap to GPS,
  or drag the marker. Edits save automatically.
- **Start track** records the line with timestamps and keeps the screen awake.

## Export / share
**Export** offers:
- **KMZ** — opens in Google Earth with points, photos (in the popups) and the track in one file.
- **Points (GeoJSON)**, **Track (GPX)**, **Everything (GeoJSON)** for GIS.
Files open in the system Share sheet (messenger, email, Drive, Dropbox). Export never erases data.

## Deleting
Data menu (⋯): **Clear track only**, or **Delete ALL data** — which makes you type the word
`DELETE` first, so nothing is wiped by accident.

## Large rasters & limits
For big classified maps, export a **Cloud-Optimized GeoTIFF (COG)** (`gdal_translate -of COG in.tif out.tif`)
— the reader then pulls only the pixels for the current view, so large files open without choking.
Plain (striped) GeoTIFFs are read whole into memory, which is heavy on phones. The GeoTIFF reader uses
web workers, so the page must be served over localhost/https (not file://).

## Large rasters (PMTiles)
Loading large `.tif` files directly into the browser can crash the tab on mobile devices due to memory limits. To solve this, convert your big rasters to **PMTiles** format. PMTiles is a single-file archive that allows the app to extract only the currently visible map tiles directly from your device's storage, instantly and completely offline.

**How to convert a GeoTIFF to PMTiles:**
You will need GDAL and the `pmtiles` python package (`pip install pmtiles`). Run these commands on your PC:

1. Convert TIF to MBTiles:
   ```bash
   gdal_translate -of mbtiles -co TILE_FORMAT=PNG -co ZOOM_LEVEL_STRATEGY=AUTO in.tif temp.mbtiles
	```
2. Build overviews (pyramids) for fast zooming on lower scales:
   ```bash
	gdaladdo -r average temp.mbtiles
	```
3. Convert to PMTiles:
   ```bash
	pmtiles convert temp.mbtiles out.pmtiles  
	```
Now just select out.pmtiles inside the app. The app still supports basic .tif for small files, but PMTiles is strongly recommended for anything larger than a few megabytes.   
Recording a track with the screen **fully off** is throttled by browsers (a Wake Lock is requested, so
keep the screen on). A pre-downloaded whole-region OSM/satellite basemap and true background tracking
are the only things that would need a native Android build.
