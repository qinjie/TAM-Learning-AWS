# Setup ElasticSearch on EC2


## 1. Install ElasticSearch & Kibana

1. Provision an EC2 using Amazon Linux Image v2 and SSH into this EC2 instance.

2. Import ElasticSearch PGP Key

```
sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```

3. Configure RPM Repository for ElasticSearch.

```
sudo vim /etc/yum.repos.d/elasticsearch.repo
```

   * Paste in following configuration. 

```
[elasticsearch]
name=Elasticsearch repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=0
autorefresh=1
type=rpm-md
```

4. Install Elasticsearch from RPM Repository

```sh
yum install --enablerepo=elasticsearch elasticsearch
```

5. Configure RPM Repository for Kabana.

```sh
sudo vim /etc/yum.repos.d/kibana.repo
```

   * Paste in following configuration.

```
[kibana-7.x]
name=Kibana repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

6. Install Kibana from RPM Repository.

```sh
sudo yum install kibana
```

7. Enable auto running of Elasticsearch service.

```sh
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch.service
sudo systemctl start elasticsearch.service
```

8. Enable auto running of Kibana service.

```sh
sudo systemctl daemon-reload
sudo systemctl enable kibana.service
sudo systemctl start kibana.service
```

### [Optional] JVM Not Found

If elasticsearch.service cannot be started due to JVM not found, perform following to set JAVA_HOME.

1. Check where is java.

```sh
which java
```

2. Find the original location of the output, which shows the path of JDK folder, e.g. `/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.342.b07-1.amzn2.0.1.x86_64`

```sh
ls -la <OUTPUT>
```

3. Append following to `/etc/profile`.

```
sudo vim /etc/profile
```

   * Append following, update the JAVA_HOME value accordingly.

```sh
export JAVA_HOME="/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.342.b07-1.amzn2.0.1.x86_64"
export PATH=$JAVA_HOME/bin:$PATH
```

   

## 2. Configure ElasticSearch and Kibana

1. Before you are able to access ElasticSearch API, you need to obtain an API key.

   * discovery.type: single-node setting is needed to stop ES accepting other nodes.
   * network.host: 0.0.0.0 will make it listen on all IPs, internal and external.

```
echo "xpack.security.enabled: true" | sudo tee -a /etc/elasticsearch/elasticsearch.yml
echo "discovery.type: single-node" | sudo tee -a /etc/elasticsearch/elasticsearch.yml
echo "network.host: 0.0.0.0" | sudo tee -a /etc/elasticsearch/elasticsearch.yml
```

2. Restart elasticsearch service.

```
sudo systemctl restart elasticsearch.service
```

3. Obtain an API key.

```
sudo /usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto
```

4. Configure Kibana. Paste in the `kibana_system` password when prompted.

```
echo 'elasticsearch.username: "kibana_system"' | sudo tee -a /etc/kibana/kibana.yml
echo "server.host: 0.0.0.0" | sudo tee -a /etc/kibana/kibana.yml
sudo /usr/share/kibana/bin/kibana-keystore add elasticsearch.password
```



5. Since we have enabled `xpack.security` plugin in elasticsearch.yml, we need to configure Kibana with username and password.

```bash
sudo vim /etc/kibana/kibana.yml
```

   * Enable and update following values:

```ini
elasticsearch.username: kibana
elasticsearch.password: <KIBANA_PASSWORD>
```

6. Restart Kibana service.

```
sudo systemctl restart elasticsearch.service
```

## 3. Test

1. Test ElasticSearch API from local machine.

```sh
curl localhost:9200 -u elastic -p <ELASTIC_PASSWORD>
```

2. Test from another machine.

```
curl 13.214.191.3:9200 -u elastic -p <ELASTIC_PASSWORD>
```

3. Test Kibana dashboard.

```
curl localhost:5601 -u elastic -p <ELASTIC_PASSWORD>
```


## Reference

* https://rharshad.com/setup-elasticsearch-cluster-aws-ec2/
* https://github.com/RedisLabsModules/automata/issues/8
