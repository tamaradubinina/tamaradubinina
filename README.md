Docker

★ Для docker compose

    git clone https://github.com/skl256/grafana_stack_for_docker.git  \ нажимаем y
    cd grafana_stack_for_docker  \
    sudo mkdir -p /mnt/common_volume/swarm/grafana/config  \
    sudo mkdir -p /mnt/common_volume/grafana/{grafana-config,grafana-data,prometheus-data,loki-data,promtail-data}  \
    sudo chown -R $(id -u):$(id -g) {/mnt/common_volume/swarm/grafana/config,/mnt/common_volume/grafana}  \
    touch /mnt/common_volume/grafana/grafana-config/grafana.ini  \
    cp config/* /mnt/common_volume/swarm/grafana/config/  \
    mv grafana.yaml docker-compose.yaml  \
    
![image](https://github.com/user-attachments/assets/610227ad-b66b-4a51-bb1d-cd5bb1b6fb07)

★ Установка последней версии и утилитю docker-compose

    sudo yum install curl  \
    COMVER=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep 'tag_name' | cut -d" -f4)  \
    sudo curl -L "https://github.com/docker/compose/releases/download/$COMVER/docker-compose-$(uname -s)-$(uname -m)" -o 
    /usr/bin/docker-compose  \
    sudo chmod +x /usr/bin/docker-compose  \
    docker-compose --version  \

![image](https://github.com/user-attachments/assets/a8b74838-10d2-4d88-93a4-39e935943c95)

★ Установка и настройка Docker

    sudo wget -P /etc/yum.repos.d/ https://download.docker.com/linux/centos/docker-ce.repo
    sudo yum install docker-ce docker-ce
    sudo systemctl enable docker --now
    docker compose up -d

![image](https://github.com/user-attachments/assets/9d4552df-abf6-48aa-a7df-523c610e9328)

![image](https://github.com/user-attachments/assets/898539ba-c100-4730-85cf-aba3984a2bfc)

![image](https://github.com/user-attachments/assets/52771674-c60c-4962-8677-d30eaf4f967b)

Конфигурация отдельных сервисов

Prometheus

★ После установки докера пишем sudo vi docker-compose.yaml (Обязательно что бы был расширения.yaml)

  ➤ Переходим в текстовый редактор
  
  ➤ Что-бы изменить в тесковом редакторе надо нажать insert
  
  ➤ Что бы сохранить в этом документе нажимаем Esc пишем :wq! В текставом редакторе нужно поставить node-exporter после services
  
      node-exporter:
        image: prom/node-exporter
        volumes:
          - /proc:/host/proc:ro
          - /sys:/host/sys:ro
          - /:/rootfs:ro
        container_name: exporter < запомнить название name
        hostname: exporter
        command:
          - --path.procfs=/host/proc
          - --path.sysfs=/host/sys
          - --collector.filesystem.ignored-mount-points
          - ^/(sys|proc|dev|host|etc|rootfs/var/lib/docker/containers|rootfs/var/lib/docker/overlay2|rootfs/run/docker/netns|rootfs/var/lib/docker/aufs)($$|/)
        ports:
          - 9100:9100
        restart: unless-stopped
        environment:
            TZ: "Europe/Moscow"
        networks:
          - default

![image](https://github.com/user-attachments/assets/9f44f6e0-5492-467e-b824-c36f4f132787)

Далее:

    cd
    cd /mnt/common_volume/swarm/grafana/config 
    sudo vi prometheus.yaml 

![image](https://github.com/user-attachments/assets/b70be1f5-194d-4234-94c5-0e55be0aa3d9)

★ В этом файле нужно изменить первый ip Адрес на тот, который запоминали

  ➤ Вставляем в первый targets
  ➤ После двоеточие цифры оставляем

Grafana

★ переходим на сайт localhost:3000

★ User & Password GRAFANA: admin

  ➤ Код графаны: 3000
  ➤ Код прометеуса: http://prometheus:9090

★ в меню выбираем вкладку Dashboards и создаем Dashboard

  ➤ ждем кнопку +Add visualization, а после "Configure a new data source"
  ➤ выбираем Prometheus

★ Connection

  ➤ http://prometheus:9090

★ Authentication

  ➤ Basic authentication
    
    ➢ User: admin
    ➢ Password: admin
  
  ➤ Нажимаем на Save & test и должно показывать зелёную галочку

★ в меню выбираем вкладку Dashboards и создаем Dashboard

  ➤ ждем кнопку "Import dashboard"
  ➤ Find and import dashboards for common applications at grafana.com/dashboards: 1860 //нажать на кнопку Load
  ➤ Select Prometheus нажать на кнопку "Import"
  
![image](https://github.com/user-attachments/assets/8272c7c6-beeb-472b-afe2-d80274f9e20f)

Если не работает, то переходим в:

    cd grafana_stack_for_docker
    sudo docker compose stop
    sudo docker compose up -d

![image](https://github.com/user-attachments/assets/80892aaf-2126-4e10-9cc6-9926d9bba6ca)

![image](https://github.com/user-attachments/assets/7d061740-ff41-45f8-9443-28c5e42521a9)

Перезапускаем докер

VicroriaMetrics

★ Изменяем docker-compose.yaml:

    cd grafana_stack_for_docker
    sudo vi docker-compose.yaml

![image](https://github.com/user-attachments/assets/aac337b9-aa56-4a3e-a158-9ab9cc6a84a5)

После в самом текстовом редакторе после prometheus вставляем:

        vmagent:
        container_name: vmagent
        image: victoriametrics/vmagent:v1.105.0
        depends_on:
          - "victoriametrics"
        ports:
          - 8429:8429
        volumes:
          - vmagentdata:/vmagentdata
          - ./prometheus.yml:/etc/prometheus/prometheus.yml
        command:
          - "--promscrape.config=/etc/prometheus/prometheus.yml"
          - "--remoteWrite.url=http://victoriametrics:8428/api/v1/write"
        restart: always
      # VictoriaMetrics instance, a single process responsible for
      # storing metrics and serve read requests.
      victoriametrics:
        container_name: victoriametrics
        image: victoriametrics/victoria-metrics:v1.105.0
        ports:
          - 8428:8428
          - 8089:8089
          - 8089:8089/udp
          - 2003:2003
          - 2003:2003/udp
          - 4242:4242
        volumes:
          - vmdata:/storage
        command:
          - "--storageDataPath=/storage"
          - "--graphiteListenAddr=:2003"
          - "--opentsdbListenAddr=:4242"
          - "--httpListenAddr=:8428"
          - "--influxListenAddr=:8089"
          - "--vmalert.proxyURL=http://vmalert:8880"
        restart: always

Сохранить и выйти

★ Захом в connection
   
   ➢ там где мы писали http:prometheus:9090 пишем http:victoriametrics:9090 И заменяем имя из "Prometheus-2" в "Vika"
   ➢ нажимаем на dashboards add visualition выбираем "Vika"
   ➢ снизу меняем на "cod"
   ➢ Переходим в терминал и пишем

    echo -e "# TYPE OILCOINT_metric1 gauge\nOILCOINT_metric1 0" | curl --data-binary @- 
    http://localhost:8428/api/v1/import/prometheus  
    curl -G 'http://localhost:8428/api/v1/query' --data-urlencode 'query=OILCOINT_metric1'

![image](https://github.com/user-attachments/assets/0c676249-b14f-44b8-ae58-1a80c20caf87)

★ Копируем переменную OILCOINT_metric1 и вставляем в cod

★ Нажимаем run image

★ Должно получится вот так
