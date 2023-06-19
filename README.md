# Домашнее задание к занятию "Кластеризация и балансировка нагрузки" - Лебедев Алексей, fops-10



---

### Задание 1   


- Запустите два simple python сервера на своей виртуальной машине на разных портах
- Установите и настройте HAProxy, воспользуйтесь материалами к лекции по ссылке
- Настройте балансировку Round-robin на 4 уровне.
- На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy.

### Решение:  

  - Создадим index.html файлы, запустим два simple python сервера в фоне и проверим:

    ```
    root@haproxy:/home/vagrant# mkdir hw
    root@haproxy:/home/vagrant# cd hw
    root@haproxy:/home/vagrant/hw# mkdir -p html{1,2}
    root@haproxy:/home/vagrant/hw# ls -lai
    total 16
    3801096 drwxr-xr-x 4 root    root    4096 Jun 19 09:58 .
    3801090 drwxr-xr-x 5 vagrant vagrant 4096 Jun 19 09:54 ..
    3801100 drwxr-xr-x 2 root    root    4096 Jun 19 09:58 html1
    3801101 drwxr-xr-x 2 root    root    4096 Jun 19 09:58 html2
    root@haproxy:/home/vagrant/hw# echo "Server 1 Port 8888" > html1
    bash: html1: Is a directory
    root@haproxy:/home/vagrant/hw# echo "Server 1 Port 8888" > html1/index.html
    root@haproxy:/home/vagrant/hw# echo "Server 1 Port 9999" > html2/index.html
    root@haproxy:/home/vagrant/hw# cat html1/index.html
    Server 1 Port 8888
    root@haproxy:/home/vagrant/hw# cd html1
    root@haproxy:/home/vagrant/hw/html1# python3 -m http.server 8888 --bind 0.0.0.0 &
    [1] 1230
    root@haproxy:/home/vagrant/hw/html1# Serving HTTP on 0.0.0.0 port 8888 (http://0.0.0.0:8888/) ...

    root@haproxy:/home/vagrant/hw/html1# cd ..
    root@haproxy:/home/vagrant/hw# cd html2
    root@haproxy:/home/vagrant/hw/html2# python3 -m http.server 9999 --bind 0.0.0.0 &
    [2] 1232
    root@haproxy:/home/vagrant/hw/html2# Serving HTTP on 0.0.0.0 port 9999 (http://0.0.0.0:9999/) ...

    root@haproxy:/home/vagrant/hw/html2# curl http://localhost:8888
    127.0.0.1 - - [19/Jun/2023 10:14:05] "GET / HTTP/1.1" 200 -
    Server 1 Port 8888
    root@haproxy:/home/vagrant/hw/html2# curl http://localhost:9999
    127.0.0.1 - - [19/Jun/2023 10:14:10] "GET / HTTP/1.1" 200 -
    Server 1 Port 9999
    root@haproxy:/home/vagrant/hw/html2#

    ```

  - Установим HAproxy и пропишем следующий конфиг:

```
global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
        stats timeout 30s
        user haproxy
        group haproxy
        daemon

        # Default SSL material locations
        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private

        # Default ciphers to use on SSL-enabled listening sockets.
        # For more information, see ciphers(1SSL). This list is from:
        #  https://hynek.me/articles/hardening-your-web-servers-ssl-ciphers/
        # An alternative list with additional directives can be obtained from
        #  https://mozilla.github.io/server-side-tls/ssl-config-generator/?server=haproxy
        ssl-default-bind-ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:RSA+AESGCM:RSA+AES:!aNULL:!MD5:!DSS
        ssl-default-bind-options no-sslv3

defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http


listen stats  # веб-страница со статистикой
        bind                    :888
        mode                    http
        stats                   enable
        stats uri               /stats
        stats refresh           5s
        stats realm             Haproxy\ Statistics

frontend example  # секция фронтенд
        mode http
        bind :8088
        default_backend web_servers
#       acl ACL_example.com hdr(host) -i example.com
#       use_backend web_servers if ACL_example.com

backend web_servers    # секция бэкенд
        mode http
        balance roundrobin
#        option httpchk
        http-check expect status 200
        server s1 127.0.0.1:8888 check
        server s2 127.0.0.1:9999 check
```

Хочу отметить, что с конфигурацией, которая была по ссылке, сервис у меня не запустился, пришлось чуть подправить.  


- HAproxy статистика, где видно количество обращений:

      
