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
