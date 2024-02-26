```
 - sudo useradd -m -s /bin/false prometheus
                - sudo mkdir -p /var/lib/prometheus
                - sudo chown prometheus /var/lib/prometheus/
                - sudo wget {{PackageUrl}} -P /tmp
                - sudo tar -zxpvf /tmp/prometheus-2.50.0.linux-amd64.tar.gz -C /tmp
                - sudo rm -rvf /tmp/prometheus-2.50.0.linux-amd64.tar.gz
                - sudo cp /tmp/prometheus-2.50.0.linux-amd64/prometheus /usr/local/bin
                - sudo cp /tmp/prometheus-2.50.0.linux-amd64/promtool /usr/local/bin
                - sudo chown prometheus /usr/local/bin/prometheus
                - sudo chown prometheus /usr/local/bin/promtool
                - sudo mv /tmp/prometheus-2.50.0.linux-amd64 /etc/prometheus
                - sudo chown -R prometheus /etc/prometheus/
                - sudo echo "{{resolve:ssm:PrometheusScrapeConfig}}" > /etc/prometheus/prometheus.yml
                - sudo echo "{{resolve:ssm:PrometheusServiceConfig}}" > /etc/systemd/system/prometheus.service
                - sudo systemctl daemon-reload
                - sudo systemctl start prometheus
```
```

/etc/prometheus/prometheus.yml
scrape_configs:
  - job_name: 'node'
    static_configs:
     - targets: ['localhost:9100']
remote_write:
   - url: https://aps-workspaces.us-east-1.amazonaws.com/workspaces/ws-ae461c26-fbfa-42bf-b6a1-13c63c0d5ecc/api/v1/remote_write
      queue_config:
        max_samples_per_send: 1000
        max_shards: 200
        capacity: 2500
        sigv4:
          region: us-east-1

/etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
           --config.file /etc/prometheus/prometheus.yml \
            --storage.tsdb.path /var/lib/prometheus/ \
            --web.console.templates=/etc/prometheus/consoles \
            --web.console.libraries=/etc/prometheus/console_libraries
[Install]
WantedBy=multi-user.target
```
