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
