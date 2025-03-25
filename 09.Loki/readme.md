# Setting Up Loki with Grafana

## Prerequisite

Ensure you have Helm installed on your system. If not, you can install it from [here](https://helm.sh/docs/intro/install/).

## 1. Add and Update Helm Repository

First, add the Grafana Helm repository and update it:

```sh
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

## 2. Install Loki Stack

Install the Loki stack using Helm:

```sh
helm install loki grafana/loki-stack --create-namespace loki-grafana --values=values.yaml 
```

## 3. Import Grafana Dashboard

Use the following JSON to import and create a dashboard in Grafana:

```json
{
    "__inputs": [
        {
            "name": "DS_LOKI",
            "label": "Loki",
            "description": "",
            "type": "datasource",
            "pluginId": "loki",
            "pluginName": "Loki"
        },
        {
            "name": "DS_PROMETHEUS",
            "label": "Prometheus",
            "description": "",
            "type": "datasource",
            "pluginId": "prometheus",
            "pluginName": "Prometheus"
        }
    ],
    "__elements": {},
    "__requires": [
        {
            "type": "grafana",
            "id": "grafana",
            "name": "Grafana",
            "version": "11.3.0"
        },
        {
            "type": "panel",
            "id": "logs",
            "name": "Logs",
            "version": ""
        },
        {
            "type": "datasource",
            "id": "loki",
            "name": "Loki",
            "version": "1.0.0"
        },
        {
            "type": "datasource",
            "id": "prometheus",
            "name": "Prometheus",
            "version": "1.0.0"
        },
        {
            "type": "panel",
            "id": "text",
            "name": "Text",
            "version": ""
        },
        {
            "type": "panel",
            "id": "timeseries",
            "name": "Time series",
            "version": ""
        }
    ],
    "annotations": {
        "list": [
            {
                "$$hashKey": "object:75",
                "builtIn": 1,
                "datasource": {
                    "type": "datasource",
                    "uid": "grafana"
                },
                "enable": true,
                "hide": true,
                "iconColor": "rgba(0, 211, 255, 1)",
                "name": "Annotations & Alerts",
                "type": "dashboard"
            }
        ]
    },
    "description": "Loki logs panel with prometheus variables",
    "editable": true,
    "fiscalYearStartMonth": 0,
    "graphTooltip": 0,
    "id": null,
    "links": [],
    "panels": [
        {
            "datasource": {
                "type": "loki",
                "uid": "${DS_LOKI}"
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
                        "axisPlacement": "hidden",
                        "barAlignment": 0,
                        "barWidthFactor": 0.6,
                        "drawStyle": "bars",
                        "fillOpacity": 100,
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
                        "showPoints": "never",
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
                    },
                    "unit": "short"
                },
                "overrides": []
            },
            "gridPos": {
                "h": 3,
                "w": 24,
                "x": 0,
                "y": 0
            },
            "id": 6,
            "options": {
                "dataLinks": [],
                "legend": {
                    "calcs": [],
                    "displayMode": "list",
                    "placement": "bottom",
                    "showLegend": false
                },
                "tooltip": {
                    "mode": "multi",
                    "sort": "none"
                }
            },
            "pluginVersion": "11.3.0",
            "targets": [
                {
                    "datasource": {
                        "type": "loki",
                        "uid": "${DS_LOKI}"
                    },
                    "editorMode": "code",
                    "expr": "sum(count_over_time({namespace=\"$namespace\", pod=~\"$pod\"} |~ \"$search\"[$__interval]))",
                    "queryType": "range",
                    "refId": "A"
                }
            ],
            "type": "timeseries"
        },
        {
            "datasource": {
                "type": "loki",
                "uid": "${DS_LOKI}"
            },
            "gridPos": {
                "h": 25,
                "w": 24,
                "x": 0,
                "y": 3
            },
            "id": 2,
            "maxDataPoints": "",
            "options": {
                "dedupStrategy": "none",
                "enableLogDetails": true,
                "prettifyLogMessage": false,
                "showCommonLabels": false,
                "showLabels": false,
                "showTime": true,
                "sortOrder": "Descending",
                "wrapLogMessage": true
            },
            "pluginVersion": "11.3.0",
            "targets": [
                {
                    "datasource": {
                        "type": "loki",
                        "uid": "${DS_LOKI}"
                    },
                    "editorMode": "code",
                    "expr": "{namespace=\"$namespace\", pod=~\"$pod\"} |~ \"$search\"",
                    "queryType": "range",
                    "refId": "A"
                }
            ],
            "title": "Logs Panel",
            "type": "logs"
        },
        {
            "gridPos": {
                "h": 3,
                "w": 24,
                "x": 0,
                "y": 28
            },
            "id": 4,
            "transparent": true,
            "type": "text"
        }
    ],
    "schemaVersion": 40,
    "tags": [],
    "templating": {
        "list": [
            {
                "current": {},
                "datasource": {
                    "type": "prometheus",
                    "uid": "${DS_PROMETHEUS}"
                },
                "definition": "label_values(kube_pod_info, namespace)",
                "includeAll": false,
                "name": "namespace",
                "options": [],
                "query": "label_values(kube_pod_info, namespace)",
                "refresh": 1,
                "regex": "",
                "type": "query"
            },
            {
                "allValue": ".*",
                "current": {},
                "datasource": {
                    "type": "prometheus",
                    "uid": "${DS_PROMETHEUS}"
                },
                "definition": "label_values(container_network_receive_bytes_total{namespace=~\"$namespace\"},pod)",
                "includeAll": true,
                "multi": true,
                "name": "pod",
                "options": [],
                "query": "label_values(container_network_receive_bytes_total{namespace=~\"$namespace\"},pod)",
                "refresh": 1,
                "regex": "",
                "type": "query"
            },
            {
                "current": {
                    "text": "",
                    "value": ""
                },
                "name": "search",
                "options": [
                    {
                        "selected": true,
                        "text": "",
                        "value": ""
                    }
                ],
                "query": "",
                "type": "textbox"
            }
        ]
    },
    "time": {
        "from": "now-30m",
        "to": "now"
    },
    "timepicker": {},
    "timezone": "",
    "title": "Loki Quick Search",
    "uid": "de27tglj6zzeoe",
    "version": 1,
    "weekStart": ""
}
```

To import this JSON, follow these steps:

1. Open your Grafana instance.
2. Navigate to the Dashboard section.
3. Click on the `+` icon to create a new dashboard.
4. Select `Import`.
5. Paste the JSON content and click `Load`.

Your Loki Quick Search dashboard should now be set up and ready to use.

[Back to Main README](../README.md)