Примечания 

    Убедитесь, что ваш пользователь имеет права sudo. 

    Все компоненты устанавливаются локально на один сервер. 

    Фильтрация сообщений реализуется через mutate.gsub. 

    Для тестирования вы можете вызвать событие аудита, например:
    ausearch -k test
     
     
**1**

    apt install apt-transport-https
**2**

    wget -qO /tmp/elastic.gpg https://artifacts.elastic.co/GPG-KEY-elasticsearch
**3**

    sudo mv /tmp/elastic.gpg /etc/apt/trusted.gpg.d/elastic.gpg
**4**

    scp C:\pitonprojekt\hw_elk\ansible\roles\elasticsearch\files\elasticsearch-7.12.0-amd64.deb ubuntu@51.250.95.145:/home/ubuntu/
**5**

    sudo dpkg -i elasticsearch-7.12.0-amd64.deb
**6**

    systemctl daemon-reload
    systemctl enable elasticsearch.service
    systemctl start elasticsearch.service
**7**

    systemctl status elasticsearch.service
**8**

    scp C:\pitonprojekt\hw_elk\ansible\roles\elasticsearch\files\kibana-7.12.0-amd64.deb ubuntu@51.250.95.145:/home/ubuntu/
**9**

    dpkg -i kibana-7.12.0-amd64.deb
**10**

    sudo nano /etc/kibana/kibana.yml
    Поправим в /etc/kibana/kibana.yml: 
    ‘server.host: "localhost”’ на ‘server.host: "0.0.0.0"’,
    чтобы она была доступна не только на локальной машине
    А также раскомментируем поле: elasticsearch.hosts: ["http://localhost:9200"]
**11**

    systemctl restart kibana
    apt-get install -y auditd
    systemctl restart auditd
**12**

    scp C:\pitonprojekt\hw_elk\ansible\roles\elasticsearch\files\filebeat-7.12.0-amd64.deb ubuntu@51.250.95.145:/home/ubuntu/
**13**

    dpkg -i filebeat-7.12.0-amd64.deb
**14**

    sudo nano /etc/filebeat/filebeat.yml
    Раскомментируем поля в /etc/filebeat/filebeat.yml:
    setup.kibana:
      host: "localhost:5601"
**15**

    filebeat modules enable auditd
    filebeat setup -e
    systemctl restart filebeat
**16**

    wget -qO /tmp/elastic.gpg https://artifacts.elastic.co/GPG-KEY-elasticsearch 
**17**

    sudo mv /tmp/elastic.gpg /etc/apt/trusted.gpg.d/elastic.gpg
**18**

    scp C:\pitonprojekt\hw_elk\ansible\roles\elasticsearch\files\logstash-7.12.0-amd64.deb ubuntu@51.250.95.145:/home/ubuntu/
**19**
    dpkg -i logstash-7.12.0-amd64.deb
**20**

    systemctl restart logstash
**21**

    sudo nano /etc/filebeat/filebeat.yml
    Теперь поправим путь наших логов: настроим Filebeat на отправку логов не напрямую в Elasticsearch, 
    а в Logstash, и настроим Logstash на прием логов из Filebeat и отправку в Elasticsearch (пока без фильтрации).
    В /etc/filebeat/filebeat.yml закомментируем строки, диктующие направлять логи в Elastic 
    и раскомментируем те, что предписывают направлять логи в Logstash:
    #output.elasticsearch
    output.logstash
**21**

    systemctl restart filebeat
**22**

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
**23**

    systemctl restart logstash

**24**

    Добавим в наш конфиг-файл, находящийся в /etc/logstash/conf.d/, следующие строки:
    filter {
      mutate {
        gsub => ["message", "hostname=.+ addr=\d+\.\d+\.\d+\.\d+", "hostname=XXX addr=xx.xx.xx.xx"]
      }
    }
**25**

    systemctl restart logstash
**26**

    systemctl status logstash

**https://habr.com/ru/articles/590529/**

# добавить в гит большие файлы
**Установите Git LFS**

    git lfs install
**Отслеживайте большие файлы через Git LFS**

    git lfs track "*.deb"
**Добавьте файлы в репозиторий**

    git add .gitattributes
    git add *.deb
**Сделайте коммит и отправьте на сервер**

    git commit -m "Add large files via Git LFS"
    git push origin main
**Как проверить, что файлы добавлены правильно**

    git lfs ls-files



