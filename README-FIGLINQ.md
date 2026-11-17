# Figlinq image exporter

Image tag: `figlinq/image-exporter:v5` (Streambed 9.1.0). Previous: `v4` (2025-11-21).

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
plotly.js's webpack config. From a Streambed checkout of the release branch or tag (Node 20,
`yarn install --frozen-lockfile` done; the entry path must be absolute, or webpack takes it for
a module name):

```
node shelly/plotlyjs/bin/bundle.js "$PWD/shelly/plotlyjs/static/plotlyjs/src/plotly.js" \
    /tmp/plotlyjs-bundle-unmin.js /tmp/plotlyjs-bundle.js
cp /tmp/plotlyjs-bundle.js <image-exporter>/custom/plotlyjs-bundle.js
```

Record in `custom/plotlyjs-bundle.SOURCE`: the Streambed release branch (or tag) and commit,
the sha256 of Streambed's `yarn.lock` (the webpack output is reproducible only with the same
`node_modules`), the Node, plotly.js and webpack versions, and the bundle's sha256. Cite the
release branch or tag, not a feature branch. v5: integration-9.1.1 at `6dcf29b5c`.

## Build and check

```
docker build -f deployment/Dockerfile -t figlinq/image-exporter:v5 .
docker run --rm --network none --entrypoint sh figlinq/image-exporter:v5 -c "fc-list : family | sort -u"
```

Publishing the image and moving the tag in Streambed (`deploy/compose/compose.yml`,
`docker-compose/dev/docker-compose.yml`, `deploy/testing/manager.py`) are release steps that
need the owner's go.

## Pinning by digest

`v5` is a tag, and a tag can be pushed again. The published image is built from the merged
`main` commit of this repository (never from a pull request branch), from a clean checkout of
that commit. After the push, record the digest the registry returned and pin installations to
it:

```
docker push figlinq/image-exporter:v5
docker inspect --format '{{index .RepoDigests 0}}' figlinq/image-exporter:v5
# -> figlinq/image-exporter@sha256:<digest>
```

- Write the digest under "Published digests" below, next to the `main` commit it was built
  from and the date: `v5`: `sha256:<digest>`, built from `main` at `<commit>`, `<date>`.
- In Streambed, reference the image as `figlinq/image-exporter:v5@sha256:<digest>` (Compose and
  Podman pull the digest and ignore the tag, which stays for readers). An instance can check
  what it runs with `docker image inspect --format '{{.RepoDigests}}' figlinq/image-exporter:v5`.
- Never push a different build under an existing tag: make a new tag (`v5.1`, `v6`).

### Published digests

- `v5`: not published yet (needs the owner's go).

## Development notes

Note: three runtimes. `bin/orca.js` (the launcher) and `npm install` run on the image's Node
8.17 (`NODE_VERSION` in `deployment/Dockerfile`), so that code must stay Node 8 compatible. The
Electron app it starts (`src/app`) runs in Electron 6.1 (Node 12). Code in the rendered page (the
plotly.js bundle, `src/component`) runs in Chromium 76 and is shipped as ES5 (see "Updating the
plotly.js bundle").

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
