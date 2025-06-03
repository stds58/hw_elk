Примечания 

    Убедитесь, что ваш пользователь имеет права sudo. 

    Все компоненты устанавливаются локально на один сервер. 

    Фильтрация сообщений реализуется через mutate.gsub. 

    Для тестирования вы можете вызвать событие аудита, например:
    ausearch -k test
     
     
     
apt install apt-transport-https

wget -qO /tmp/elastic.gpg https://artifacts.elastic.co/GPG-KEY-elasticsearch

sudo mv /tmp/elastic.gpg /etc/apt/trusted.gpg.d/elastic.gpg

scp C:\pitonprojekt\hw_elk\ansible\roles\elasticsearch\files\elasticsearch-7.12.0-amd64.deb ubuntu@51.250.95.145:/home/ubuntu/

sudo dpkg -i elasticsearch-7.12.0-amd64.deb

systemctl daemon-reload
systemctl enable elasticsearch.service
systemctl start elasticsearch.service

systemctl status elasticsearch.service

scp C:\pitonprojekt\hw_elk\ansible\roles\elasticsearch\files\kibana-7.12.0-amd64.deb ubuntu@51.250.95.145:/home/ubuntu/

dpkg -i kibana-7.12.0-amd64.deb

sudo nano /etc/kibana/kibana.yml
Поправим в /etc/kibana/kibana.yml: 
‘server.host: "localhost”’ на ‘server.host: "0.0.0.0"’,
чтобы она была доступна не только на локальной машине
А также раскомментируем поле: elasticsearch.hosts: ["http://localhost:9200"]

systemctl restart kibana
apt-get install -y auditd
systemctl restart auditd

scp C:\pitonprojekt\hw_elk\ansible\roles\elasticsearch\files\filebeat-7.12.0-amd64.deb ubuntu@51.250.95.145:/home/ubuntu/

dpkg -i filebeat-7.12.0-amd64.deb
#dpkg -i /tmp/filebeat.deb
sudo nano /etc/filebeat/filebeat.yml
Раскомментируем поля в /etc/filebeat/filebeat.yml:
setup.kibana:
  host: "localhost:5601"

filebeat modules enable auditd
filebeat setup -e
systemctl restart filebeat

wget -qO /tmp/elastic.gpg https://artifacts.elastic.co/GPG-KEY-elasticsearch 

sudo mv /tmp/elastic.gpg /etc/apt/trusted.gpg.d/elastic.gpg

scp C:\pitonprojekt\hw_elk\ansible\roles\elasticsearch\files\logstash-7.12.0-amd64.deb ubuntu@51.250.95.145:/home/ubuntu/

dpkg -i logstash-7.12.0-amd64.deb

systemctl restart logstash

sudo nano /etc/filebeat/filebeat.yml
Теперь поправим путь наших логов: настроим Filebeat на отправку логов не напрямую в Elasticsearch, 
а в Logstash, и настроим Logstash на прием логов из Filebeat и отправку в Elasticsearch (пока без фильтрации).
В /etc/filebeat/filebeat.yml закомментируем строки, диктующие направлять логи в Elastic 
и раскомментируем те, что предписывают направлять логи в Logstash:
#output.elasticsearch
output.logstash

systemctl restart filebeat


sudo nano /etc/logstash/conf.d
Теперь настроим Logstash на прием сообщений из Filebeat и отправку их в Elastic.
Добавим новый файл в /etc/logstash/conf.d и заполним следующим содержимым:

input {
  beats {
    port => 5044
  }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
  }
}


systemctl restart logstash


Добавим в наш конфиг-файл, находящийся в /etc/logstash/conf.d/, следующие строки:
filter {
  mutate {
    gsub => ["message", "hostname=.+ addr=\d+\.\d+\.\d+\.\d+", "hostname=XXX addr=xx.xx.xx.xx"]
  }
}

systemctl restart logstash

systemctl status logstash

https://habr.com/ru/articles/590529/
