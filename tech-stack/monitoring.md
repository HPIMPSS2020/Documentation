# Monitoring System

Monitoring a system is a process within a \(distributed\) system for collecting and storing state data. Multiple tools were used for collecting, monitoring, and analyzing the data.  The monitoring system consists of 3 main elements: Prometheus, Grafana, and Visualization.

## Prometheus

[Prometheus](https://prometheus.io/) records real-time metrics in a time series database used for event monitoring and alerting. Prometheus is built using an HTTP pull model for scraping the metrics, with flexible queries and a real-time alerting system. Using the Prometheus client makes it possible to define a multi-dimensional [data model](https://prometheus.io/docs/concepts/data_model/) that is identified by metric name and key/value pairs. The gathered metrics are yield through an endpoint, which Prometheus can use to scrape the data. As for a distributed system, Prometheus discovers its targets using service discovery or static configuration. It stores all scraped samples locally and runs rules over this data to either aggregate and record new time series from existing data or generate alerts. [Grafana](https://grafana.com/) or other API consumers can be used to visualize the collected data.

## Grafana

[Grafana](https://grafana.com/) is multi-platform open-source analytics and interactive visualization web application. It provides charts, graphs, and alerts for the web when connected to supported data sources. End users can create complex monitoring dashboards using interactive query builders. Grafana includes built-in support of Prometheus. This built-in support makes it possible to scrape the data and monitor it in real-time.

## Visualization

The visualization demonstrates the tangle creation and tip selection in the web browser. The project is a fork of the [IOTA visualization](https://gitlab.hpi.de/osm/tangle-learning/iotavisualization) repository. It is written using React and D3.js and uses a Flask backend to perform various operations like service discovery, fetching peer transactions, and fetching peer information.

