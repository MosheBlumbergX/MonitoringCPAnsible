# How to install Grafana and Prometheus on RHEL8

## Prometheus 

```
sudo groupadd --system prometheus
sudo useradd -s /sbin/nologin --system -g prometheus prometheus
sudo mkdir /var/lib/prometheus

for i in rules rules.d files_sd; do
 sudo mkdir -p /etc/prometheus/${i};
done

curl -s https://api.github.com/repos/prometheus/prometheus/releases/latest \
| grep browser_download_url \
| grep linux-amd64 \
| cut -d '"' -f 4 \
| wget -qi -


tar xvf prometheus-*.tar.gz
cd prometheus-*/
sudo cp prometheus promtool /usr/local/bin/

sudo cp -r prometheus.yml consoles/ console_libraries/ /etc/prometheus/ 
```

Create a Prometheus configuration file.

Prometheus configuration file will be located under `/etc/prometheus/prometheus.yml`, in the targets, add the CP components, for example (8079 is the zookeeper service, 8080 is the broker service):

```
      - targets: ['localhost:9090','ip-192-168-4-72.eu-west-2.compute.internal:8079' ,'ip-192-168-5-25.eu-west-2.compute.internal:8080' ,'ip-192-168-6-107.eu-west-2.compute.internal:8079' ,'ip-192-168-7-245.eu-west-2.compute.internal:8079' ,'ip-192-168-8-96.eu-west-2.compute.internal:8080','ip-192-168-11-244.eu-west-2.compute.internal:8080']
```

```
# Global config
global: 
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.  
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.  
  scrape_timeout: 15s  # scrape_timeout is set to the global default (10s).

# A scrape configuration containing exactly one endpoint to scrape:# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['localhost:9090','ip-192-168-4-72.eu-west-2.compute.internal:8079' ,'ip-192-168-5-25.eu-west-2.compute.internal:8080' ,'ip-192-168-6-107.eu-west-2.compute.internal:8079' ,'ip-192-168-7-245.eu-west-2.compute.internal:8079' ,'ip-192-168-8-96.eu-west-2.compute.internal:8080','ip-192-168-11-244.eu-west-2.compute.internal:8080']
```

```
sudo vi /etc/systemd/system/prometheus.service 
```
Copy paste: 

```
[Unit]
Description=Prometheus
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus
ExecReload=/bin/kill -HUP $MAINPID
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.external-url=

SyslogIdentifier=prometheus
Restart=always

[Install]
WantedBy=multi-user.target
```

```
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chmod -R 775 /etc/prometheus/
sudo mkdir /var/lib/prometheus/
sudo chown -R prometheus:prometheus /var/lib/prometheus/
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
```

Check server IP on port 9090 , and targets, like this: `http://ec2-18-169-53-36.eu-west-2.compute.amazonaws.com:9090/targets?search=`


## Grafana 

```
sudo yum update
sudo vi /etc/yum.repos.d/grafana.repo
```
Copy paste: 

```
[grafana]
name=grafana
baseurl=https://rpm.grafana.com
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://rpm.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
```

```
sudo yum install grafana
sudo systemctl daemon-reload
sudo systemctl start grafana-server
sudo systemctl status grafana-server
sudo systemctl enable grafana-server
```

Grafana should now serve on port 3000, for example:  `http://ec2-18-169-53-36.eu-west-2.compute.amazonaws.com:3000`


## Add dashboards, or create 

Add data source in Grafana UI: Prometheus  
Add following HTTP URL and make sure to save and test (replace dns with yours): `http://ec2-18-169-53-36.eu-west-2.compute.amazonaws.com:9090`  
The new page will have a URL with a UID in it: `http://ec2-18-169-53-36.eu-west-2.compute.amazonaws.com:3000/datasources/edit/RxmrY5HVz` copy the UID (for example `RxmrY5HVz`)   
Change the datasource UID in the provided file cp-grafana.json  
Replace `"uid": "RxmrY5HVz"` with your own, the new UID from before  
Deploy CP grafana dashboard  
    Import dashboard for CP: example-confluent-platform.json


## Reference 

[Prometheus](https://computingforgeeks.com/how-to-install-prometheus-on-rhel-8/)
[Grafana](https://grafana.com/docs/grafana/latest/setup-grafana/installation/rpm/)