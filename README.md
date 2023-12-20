# weather_exporter
Part of the 'On Rocky' series. This is a simple implmentation of Openweather in Grafana.

## Prepare
This implmentation is layered on top of Prometheus. Some pre-flight steps are required. 

```
docker exec -it PROMETHEUS bash
yum install cronie jq bc
/usr/sbin/crond start
```
Now create a file called weather_exporter at the root of PROMETHEUS node.
```
#!/bin/sh
crl=`curl -s "https://api.openweathermap.org/data/2.5/weather?q=$1&appid=<Your Key>&units=metric" | jq '.main.temp'`
temp=$(echo "scale=2;((9/5) * $crl) + 32" |bc)
city=$(echo $1 | sed 's/ /\\ /g')
curl -XPOST http://INFLUXDB:8086/write?db=qe1 --data-binary "weather,city=$city temperature=$temp $2"
echo $temp
```
Finally add weather_exporter to a scheduler like crontabs. Type crontab -e and enter.
```
0 * * * * /weather_exporter Seattle
*/1 * * * * /weather_exporter Tukwila
```
## Grafana Dashboard
To enable a Grafana Dashboard for weather_exporter we need to add the datasource. In Grafana, Connections --> Add a new connection. Search for InfluxDB. Give the datasource a unique name. Enter http://INFLUXDB:8086 for url. Http method is GET. Click Save and Test.

Import the following dashboard as an example. 
```
{
  "annotations": {
    "list": [
      {
        "builtIn": 1,
        "datasource": {
          "type": "grafana",
          "uid": "-- Grafana --"
        },
        "enable": true,
        "hide": true,
        "iconColor": "rgba(0, 211, 255, 1)",
        "name": "Annotations & Alerts",
        "type": "dashboard"
      }
    ]
  },
  "description": "an example of a simple home made exporter.",
  "editable": true,
  "fiscalYearStartMonth": 0,
  "graphTooltip": 0,
  "id": 17,
  "links": [],
  "liveNow": false,
  "panels": [
    {
      "datasource": {
        "type": "influxdb",
        "uid": "b0d3503d-247b-4547-ae03-ad9bfb0599dc"
      },
      "description": "",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "thresholds"
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "red",
                "value": null
              },
              {
                "color": "#EAB839",
                "value": 50
              },
              {
                "color": "green",
                "value": 60
              }
            ]
          },
          "unit": "short"
        },
        "overrides": [
          {
            "matcher": {
              "id": "byName",
              "options": "weather.temperature {city: Seattle}"
            },
            "properties": [
              {
                "id": "displayName",
                "value": "Seattle"
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "weather.temperature {city: Tukwila}"
            },
            "properties": [
              {
                "id": "displayName",
                "value": "Tukwila"
              }
            ]
          }
        ]
      },
      "gridPos": {
        "h": 7,
        "w": 3,
        "x": 0,
        "y": 0
      },
      "id": 1,
      "options": {
        "colorMode": "background",
        "graphMode": "area",
        "justifyMode": "auto",
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "textMode": "value_and_name",
        "wideLayout": false
      },
      "pluginVersion": "10.2.2",
      "repeat": "City",
      "repeatDirection": "h",
      "targets": [
        {
          "datasource": {
            "type": "influxdb",
            "uid": "b0d3503d-247b-4547-ae03-ad9bfb0599dc"
          },
          "groupBy": [
            {
              "params": [
                "city::tag"
              ],
              "type": "tag"
            }
          ],
          "hide": false,
          "measurement": "weather",
          "orderByTime": "ASC",
          "policy": "default",
          "query": "select * from weather where city = '$city' GROUP BY \"city\"::tag",
          "rawQuery": true,
          "refId": "B",
          "resultFormat": "time_series",
          "select": [
            [
              {
                "params": [
                  "temperature"
                ],
                "type": "field"
              }
            ]
          ],
          "tags": []
        }
      ],
      "title": "Current Temperature",
      "type": "stat"
    },
    {
      "datasource": {
        "type": "influxdb",
        "uid": "b0d3503d-247b-4547-ae03-ad9bfb0599dc"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisBorderShow": false,
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 0,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "insertNulls": false,
            "lineInterpolation": "linear",
            "lineWidth": 1,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "auto",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 7,
        "w": 12,
        "x": 3,
        "y": 0
      },
      "id": 3,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "mode": "single",
          "sort": "none"
        }
      },
      "targets": [
        {
          "datasource": {
            "type": "influxdb",
            "uid": "b0d3503d-247b-4547-ae03-ad9bfb0599dc"
          },
          "groupBy": [
            {
              "params": [
                "$__interval"
              ],
              "type": "time"
            },
            {
              "params": [
                "null"
              ],
              "type": "fill"
            }
          ],
          "measurement": "weather",
          "orderByTime": "ASC",
          "policy": "default",
          "query": "SELECT temperature FROM weather where city = 'Seattle'",
          "rawQuery": false,
          "refId": "A",
          "resultFormat": "time_series",
          "select": [
            [
              {
                "params": [
                  "temperature"
                ],
                "type": "field"
              },
              {
                "params": [],
                "type": "mean"
              }
            ]
          ],
          "tags": [
            {
              "key": "city::tag",
              "operator": "=~",
              "value": "/^$city$/"
            }
          ]
        }
      ],
      "title": "History",
      "type": "timeseries"
    }
  ],
  "refresh": "",
  "schemaVersion": 38,
  "tags": [],
  "templating": {
    "list": [
      {
        "current": {
          "selected": false,
          "text": "Tukwila",
          "value": "Tukwila"
        },
        "datasource": {
          "type": "influxdb",
          "uid": "b0d3503d-247b-4547-ae03-ad9bfb0599dc"
        },
        "definition": "select * from weather",
        "hide": 0,
        "includeAll": false,
        "label": "City",
        "multi": false,
        "name": "city",
        "options": [],
        "query": "select * from weather",
        "refresh": 1,
        "regex": "",
        "skipUrlSync": false,
        "sort": 1,
        "type": "query"
      },
      {
        "auto": true,
        "auto_count": 30,
        "auto_min": "10s",
        "current": {
          "selected": false,
          "text": "1m",
          "value": "1m"
        },
        "hide": 0,
        "name": "Interval",
        "options": [
          {
            "selected": false,
            "text": "auto",
            "value": "$__auto_interval_Interval"
          },
          {
            "selected": true,
            "text": "1m",
            "value": "1m"
          },
          {
            "selected": false,
            "text": "10m",
            "value": "10m"
          },
          {
            "selected": false,
            "text": "30m",
            "value": "30m"
          },
          {
            "selected": false,
            "text": "1h",
            "value": "1h"
          },
          {
            "selected": false,
            "text": "6h",
            "value": "6h"
          },
          {
            "selected": false,
            "text": "12h",
            "value": "12h"
          },
          {
            "selected": false,
            "text": "1d",
            "value": "1d"
          },
          {
            "selected": false,
            "text": "7d",
            "value": "7d"
          },
          {
            "selected": false,
            "text": "14d",
            "value": "14d"
          },
          {
            "selected": false,
            "text": "30d",
            "value": "30d"
          }
        ],
        "query": "1m,10m,30m,1h,6h,12h,1d,7d,14d,30d",
        "queryValue": "",
        "refresh": 2,
        "skipUrlSync": false,
        "type": "interval"
      }
    ]
  },
  "time": {
    "from": "now-6h",
    "to": "now"
  },
  "timepicker": {
    "hidden": true,
    "refresh_intervals": [
      "1h",
      "2h",
      "1d",
      "7d",
      "30d",
      "90d"
    ]
  },
  "timezone": "browser",
  "title": "Current Weather (weather_exporter)",
  "uid": "e5cae55b-b603-489d-8eba-c660434e7a3c",
  "version": 42,
  "weekStart": "monday"
}
```
