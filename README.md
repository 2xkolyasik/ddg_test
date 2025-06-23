# ddg_test
Этот проект Ansible автоматизирует развертывание стека мониторинга (Prometheus, Grafana, Node Exporter) и базы данных MariaDB с MySQLd Exporter на системах на базе Debian. Проект включает два плейбука: site.yml для основной настройки и bootstrap.yml для установки Python3 на чистых системах.

Плейбуки:

bootstrap.yml
Устанавливает Python3 на чистых системах для работы модулей Ansible.

ansible-playbook -i inventory/hosts.ini bootstrap.yml --ask-become-pass


site.yml
Основной плейбук для настройки:

Мониторинг (roles/monitoring_host): Устанавливает Docker, Docker Compose, Prometheus, Grafana, копирует конфигурации и дашборды (mariadb_metrics.json, os_metrics.json) для мониторинга MariaDB и системных метрик.
База данных (roles/database_host): Устанавливает MariaDB, Node Exporter, MySQLd Exporter, настраивает пользователя MySQL exporter и systemd-сервисы.

ansible-playbook -i inventory/hosts.ini site.yml --ask-become-pass


Требования:

- Хосты на базе Debian.
- Доступ по SSH с пользователем (kls) и привилегиями sudo.
- Установленный Ansible на управляющем узле.
- Файл инвентаризации: inventory/hosts.ini с прописанными хостами.
- Определены переменные mysqld_exporter_password, node_exporter_version, mysqld_exporter_version в group_vars/all.yml

Примечания

Убедитесь, что /opt/monitoring/prometheus.yml включает в себя правильные адреса хостов.

scrape_configs:
  - job_name: 'mysql'
    static_configs:
      - targets: ['<database_host_ip>:9104']
  - job_name: 'node'
    static_configs:
      - targets: ['<database_host_ip>:9100']


Проверка

Логи Grafana:

ssh user@<metrics_host_ip> 'docker logs grafana'

Метрики Node Exporter:

ssh user@<database_host_ip> 'curl http://127.0.0.1:9100/metrics'

Метрики MySQLd Exporter:

ssh user@<database_host_ip> 'curl http://127.0.0.1:9104/metrics'
