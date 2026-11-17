# Figlinq image exporter

Image tag: `figlinq/image-exporter:v5` (Streambed 9.1.1). Previous: `v4` (2025-11-21).

## What v5 contains

- `custom/plotlyjs-bundle.js`: the Streambed plotly.js bundle (plotly.js 2.29.1 with the
  Figlinq transforms: `summary`, `fit`, `moving-average`/`average`, `populate-slider`,
  breakpoints). Built from Streambed (see below), copied to
  `/var/www/image-exporter/build/plotly-bundle.js`. Without it, exports of a plot with a
  `summary` transform draw the raw rows (streambed#811).
- Open Sans 3.000 (SIL Open Font License 1.1, `custom/fonts/OpenSans/OFL.txt`), the same TTF
  files Streambed serves to the plot editor (`shelly/webapp/static/webapp/fonts/OpenSans`),
  installed in `/usr/share/fonts/truetype/open-sans` and checked against
  `custom/fonts/OpenSans/SHA256SUMS` at build time. Plot text is laid out in Open Sans, as in
  the editor (streambed#923). Liberation and DejaVu stay installed as fallbacks; Streambed's
  `PLOTLY_IMAGESERVER_FONT_FAMILIES` must list the families installed here.

## Updating the plotly.js bundle

All js must run in the orca Electron (6.1, Chromium 76): the bundle is transpiled to ES5 by
plotly.js's webpack config. From a Streambed checkout of the release (Node 20, `npm ci` done):

```
node shelly/plotlyjs/bin/bundle.js shelly/plotlyjs/static/plotlyjs/src/plotly.js \
    /tmp/plotlyjs-bundle-unmin.js /tmp/plotlyjs-bundle.js
cp /tmp/plotlyjs-bundle.js <image-exporter>/custom/plotlyjs-bundle.js
```

Record the Streambed commit and the bundle's sha256 in `custom/plotlyjs-bundle.SOURCE`.

## Build and check

```
docker build -f deployment/Dockerfile -t figlinq/image-exporter:v5 .
docker run --rm --network none --entrypoint sh figlinq/image-exporter:v5 -c "fc-list : family | sort -u"
```

Publishing the image and moving the tag in Streambed (`deploy/compose/compose.yml`,
`docker-compose/dev/docker-compose.yml`, `deploy/testing/manager.py`) are release steps that
need the owner's go.

## Development notes

Note: All js must be Node 8 compatible.

TODO:

1. Copy all files for creating annotations from streambed.
2. figure passed to addSignificanceAnnotation must have actual data (for rendering) AND \_src (for referencing columns).
   to start server on port 9096 with debug output:

```
xvfb-run --auto-servernum --server-args="-screen 0 1024x768x24" node bin/orca.js serve --port 9096 --debug
```

Then navigate to `http://localhost:9096/plotly-graph` to access the component.

curl -X POST http://localhost:9096/annotate -H "Content-Type: application/json" -d '{
"figure": {
"data": [{
"type": "bar",
"x": ["A", "B"],
"y": [10, 15]
}],
"layout": {}
},
"analysisResults": {
"result": {
"p_value": 0.03,
"significance": {
"significant": true,
"stars": "\*\*"
}
}
}
}'
