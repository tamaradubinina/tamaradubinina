Docker

Быстрый старт

★ Для docker compose

    git clone https://github.com/skl256/grafana_stack_for_docker.git  \ нажимаем y
    cd grafana_stack_for_docker  \
    sudo mkdir -p /mnt/common_volume/swarm/grafana/config  \
    sudo mkdir -p /mnt/common_volume/grafana/{grafana-config,grafana-data,prometheus-data,loki-data,promtail-data}  \
    sudo chown -R $(id -u):$(id -g) {/mnt/common_volume/swarm/grafana/config,/mnt/common_volume/grafana}  \
    touch /mnt/common_volume/grafana/grafana-config/grafana.ini  \
    cp config/* /mnt/common_volume/swarm/grafana/config/  \
    mv grafana.yaml docker-compose.yaml  \


![image](https://github.com/user-attachments/assets/aff91c2c-4315-471d-95b1-96fa0d849a2a)

![image](https://github.com/user-attachments/assets/3a16ba8e-5bd8-480d-b8c8-a4fd0dcec481)


★ Установка последней версии и утилитю docker-compose

    sudo yum install curl  \
    COMVER=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep 'tag_name' | cut -d" -f4)  \
    sudo curl -L "https://github.com/docker/compose/releases/download/$COMVER/docker-compose-$(uname -s)-$(uname -m)" -o 
    /usr/bin/docker-compose  \
    sudo chmod +x /usr/bin/docker-compose  \
    docker-compose --version  \

![image](https://github.com/user-attachments/assets/0fb3523f-df39-4f2e-89c8-534505438508)

![image](https://github.com/user-attachments/assets/b3d2d3fd-51a3-43aa-9a75-ba4d8c5d7b2b)


★ Установка и настройка Docker

    sudo wget -P /etc/yum.repos.d/ https://download.docker.com/linux/centos/docker-ce.repo
    sudo yum install docker-ce docker-ce
    sudo systemctl enable docker --now
    docker compose up -d

![image](https://github.com/user-attachments/assets/8b036e9f-80fd-4d1d-bb29-3a31f3f4797f)

![image](https://github.com/user-attachments/assets/82d965b9-5341-4eff-9ab7-f205c92743d7)



![image](https://github.com/user-attachments/assets/01c60bc8-3113-45b0-9ea6-b95c3d91c306)


![image](https://github.com/user-attachments/assets/5c45d5f2-350b-4f5d-a4be-e1bc4b506ae3)


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



Note

Всё должно выглядить как на скриншоте https://github.com/TymkaTy/Docker/blob/main/grafana_stack_for_docker/Screenshot_Prometheus.png


Если не работает, то переходим в:

    cd grafana_stack_for_docker
    sudo docker compose stop
    sudo docker compose up -d

![image](https://github.com/user-attachments/assets/af7795f9-1c64-4ad0-925f-a703efa07645)

![image](https://github.com/user-attachments/assets/7d061740-ff41-45f8-9443-28c5e42521a9)


Т.е. перезапускаем докер

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

(Значение 0 меняем на любое другое)

★ Копируем переменную OILCOINT_metric1 и вставляем в cod

★ Нажимаем run image

★ Должно получится вот так
