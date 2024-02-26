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
