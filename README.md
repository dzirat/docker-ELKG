# Docker ELK stack + Grafana

Run the ELK stack (Elasticsearch, Logstash, Kibana) + Grafana with Docker,
updated to current stable releases.

The main purpose for this kind of distribution is educational.
You can play with the ELK stack and compare visual graphs provided by Kibana
and Grafana on top of the same Elasticsearch data.

Based on the official images:

- [elasticsearch](https://www.docker.elastic.co/r/elasticsearch/elasticsearch) — 9.5.1
- [logstash](https://www.docker.elastic.co/r/logstash/logstash) — 9.5.1
- [kibana](https://www.docker.elastic.co/r/kibana/kibana) — 9.5.1
- [grafana](https://hub.docker.com/r/grafana/grafana/) — 13.1.0

## Setup

1. Install [Docker](https://docs.docker.com/get-docker/) and the
   [Compose plugin](https://docs.docker.com/compose/install/) (bundled with
   Docker Desktop; `docker compose`, not the old standalone `docker-compose`).
2. Clone this repository.

## Usage

Start the ELK stack + Grafana (detached mode):

```
$ docker compose up -d
```

Give Elasticsearch a few seconds to report healthy — Logstash, Kibana, and
Grafana all wait on it via a healthcheck before starting.

Inject logs via TCP or UDP as JSON lines (the shipped Logstash pipeline
expects `json_lines`):

```
$ echo '{"message": "hello from bash"}' | nc localhost 5000
```

- Access Kibana at <http://localhost:5601>.
- Access Grafana at <http://localhost:3000> (default login `admin` / `admin`,
  you'll be prompted to change it on first login). An Elasticsearch
  datasource pointing at `logs_*` is provisioned automatically.

By default, the stack exposes the following ports:

- `5000`: Logstash TCP + UDP input 1 (indexed into `logs_tcp-5000-index-%{+YYYY.MM.dd}` / `logs_udp-5000-index-%{+YYYY.MM.dd}`)
- `6000`: Logstash TCP input 2 (indexed into `logs_tcp-6000-index-%{+YYYY.MM.dd}`)
- `9200`: Elasticsearch HTTP
- `9300`: Elasticsearch transport
- `5601`: Kibana
- `3000`: Grafana

Stop everything (and drop the named volumes) with:

```
$ docker compose down -v
```

## Notes on what changed vs. the old (2016-era) version

- Compose file uses the current `services:` top-level key and `docker compose`
  syntax (no `version:` key needed, `links:` replaced by a shared network).
- Kibana is pulled directly from Elastic's registry — no local `Dockerfile`
  build is needed anymore.
- Logstash pipeline config now lives in `logstash/pipeline/` (the modern
  convention), while `logstash/config/logstash.yml` holds settings.
- Elasticsearch/Kibana 9.x enable security (TLS + auth) by default; this repo
  disables `xpack.security.enabled` to keep the lab simple, matching the
  original's zero-auth setup. **Don't do this outside a local sandbox** —
  for anything shared, enable security and set real credentials instead.
- Elasticsearch and Grafana now persist data in named volumes
  (`esdata`, `grafanadata`) so indices and dashboards survive a restart.
- Healthchecks gate startup order instead of the old `links:` ordering.

This package uses compatible versions across all the products:

- elasticsearch 9.5.1
- logstash 9.5.1
- kibana 9.5.1
- grafana 13.1.0
