# ELK + Filebeat (Nginx) — Mac (Docker) + Debian VM

## Topology
- Mac: Docker Compose stack for Elasticsearch, Logstash, Kibana
- VM Debian (ARM friendly): Nginx + Filebeat sending to Logstash (5044) on the Mac

## 1) Start ELK on the Mac
```bash
cd docker-elk
docker compose up -d
# Wait ~30s until Elasticsearch (9200) and Kibana (5601) are ready
```

Default credentials used here:
- `elastic / changeme` (from docker-compose env). **Do not use in prod.**

## 2) Configure Filebeat on the Debian VM
```bash
# Install
sudo apt update && sudo apt install -y filebeat

# Copy config from this repo to the VM (adapt the path as needed)
# Example if you've cloned this repo on the VM already:
sudo cp filebeat-debian/filebeat.yml /etc/filebeat/filebeat.yml
sudo mkdir -p /etc/filebeat/modules.d
sudo cp filebeat-debian/modules.d/nginx.yml /etc/filebeat/modules.d/nginx.yml

# Enable and start
sudo systemctl enable filebeat --now

# Check
sudo systemctl status filebeat
sudo journalctl -u filebeat -f
```

> Ensure your Mac's LAN IP in `filebeat.yml` is correct (default put as `192.168.1.2`).

## 3) Generate logs on the VM
```bash
# 200 OK
curl http://localhost

# 404
curl http://localhost/does-not-exist
```

## 4) Verify in Kibana
- Open http://localhost:5601 on the Mac
- Discover → set index pattern to `nginx-logs-*`
- You should see documents flowing in when you hit Nginx.

## Notes
- Logstash pipeline groks Nginx access logs via `%{COMBINEDAPACHELOG}` and sets `@timestamp` from the log timestamp.
- You can further customize filters (geoip, mutate, etc.) in `docker-elk/logstash/pipeline/logstash.conf`.
