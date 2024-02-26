```
---
AWSTemplateFormatVersion: 2010-09-09
Description: |
  its for ssm automation for prometheus intsallation
Parameters:
  PrometheusInstallUrl:
    Type: String
    Description: The URL of the linux installer for Prometheus
    Default: https://github.com/prometheus/prometheus/releases/download/v2.49.1/prometheus-2.49.1.linux-amd64.tar.gz
Resources:
  MyDocument:
    Type: AWS::SSM::Document
    Properties:
      Content:
        schemaVersion: "2.2"
        description: "Run a script on Linux instances."
        parameters:
          PackageUrl:
            type: String
            description: Prometheus package URL
            default: !Ref "PrometheusInstallUrl"
        mainSteps:
          - action: aws:runShellScript
            name: runCommands
            inputs:
              timeoutSeconds: "60"
              runCommand:
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
      DocumentFormat: YAML
      DocumentType: Command
      Name: !Sub "${AWS::StackName}"
Outputs:
  PrometheusScrapeConfig:
    Value:  sudo echo "{{resolve:ssm:PrometheusScrapeConfig}}" > /etc/systemd/system/prometheus.service
  PrometheusServiceConfig:
    Value:  sudo echo "{{resolve:ssm:PrometheusServiceConfig}}" > /etc/prometheus/prometheus.yml
    ```
