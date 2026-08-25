- Создадим [манифест](./distroless-container-pod.yaml) с distroless подом
- Запустим эфемерный контейнер для отладки
```shell
kubectl debug -it distroless-container-pod \
  --image=ubuntu:latest \
  --target=nginx-distroless
```
- Чтобы получить доступ к файловой системе контейнер, нужно обратиться через `/proc/1/root/`
```shell
/ # ls -a /proc/1/root/etc/nginx/
.  ..  conf.d  fastcgi_params  koi-utf  koi-win  mime.types  modules  nginx.conf  scgi_params uwsgi_params  win-utf
```
- Установим tcpdump
```shell
apt update
apt install tcpdump
```
- Запустим
```shell
tcpdump -nn -i any -e port 80
```
- Перенаправим порт пода на хост и сделаем пару вызовов к localhost
```shell
kubectl port-forward distroless-container-pod 80:80
```
- Вывод tcpdump
```shell
root@distroless-container-pod:/# tcpdump -nn -i any -e port 80
tcpdump: WARNING: any: That device doesnt support promiscuous mode
(Promiscuous mode not supported on the "any" device)
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on any, link-type LINUX_SLL2 (Linux cooked v2), snapshot length 262144 bytes
15:21:31.815923 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 80: 127.0.0.1.50564 > 127.0.0.1.80: Flags [S], seq 493761054, win 65495, options [mss 65495,sackOK,TS val 889681303 ecr 0,nop,wscale 7], length 0
15:21:31.815936 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 80: 127.0.0.1.80 > 127.0.0.1.50564: Flags [S.], seq 1286973110, ack 493761055, win 65483, options [mss 65495,sackOK,TS val 889681303 ecr 889681303,nop,wscale 7], length 0
15:21:31.815947 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.50564 > 127.0.0.1.80: Flags [.], ack 1, win 512, options [nop,nop,TS val 889681303 ecr 889681303], length 0
15:21:31.816027 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 855: 127.0.0.1.50564 > 127.0.0.1.80: Flags [P.], seq 1:784, ack 1, win 512, options [nop,nop,TS val 889681303 ecr 889681303], length 783: HTTP: GET / HTTP/1.1
15:21:31.816034 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.80 > 127.0.0.1.50564: Flags [.], ack 784, win 506, options [nop,nop,TS val 889681303 ecr 889681303], length 0
15:21:31.816188 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 252: 127.0.0.1.80 > 127.0.0.1.50564: Flags [P.], seq 1:181, ack 784, win 506, options [nop,nop,TS val 889681304 ecr 889681303], length 180: HTTP: HTTP/1.1 304 Not Modified
15:21:31.816201 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.50564 > 127.0.0.1.80: Flags [.], ack 181, win 511, options [nop,nop,TS val 889681304 ecr 889681304], length 0
15:21:31.825942 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 80: 127.0.0.1.50576 > 127.0.0.1.80: Flags [S], seq 2171521211, win 65495, options [mss 65495,sackOK,TS val 889681313 ecr 0,nop,wscale 7], length 0
15:21:31.825960 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 80: 127.0.0.1.80 > 127.0.0.1.50576: Flags [S.], seq 2076978820, ack 2171521212, win 65483, options [mss 65495,sackOK,TS val 889681313 ecr 889681313,nop,wscale 7], length 0
15:21:31.825977 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.50576 > 127.0.0.1.80: Flags [.], ack 1, win 512, options [nop,nop,TS val 889681313 ecr 889681313], length 0
15:21:32.543868 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 855: 127.0.0.1.50564 > 127.0.0.1.80: Flags [P.], seq 784:1567, ack 181, win 512, options [nop,nop,TS val 889682031 ecr 889681304], length 783: HTTP: GET / HTTP/1.1
15:21:32.544018 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 252: 127.0.0.1.80 > 127.0.0.1.50564: Flags [P.], seq 181:361, ack 1567, win 500, options [nop,nop,TS val 889682031 ecr 889682031], length 180: HTTP: HTTP/1.1 304 Not Modified
15:21:32.544030 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.50564 > 127.0.0.1.80: Flags [.], ack 361, win 511, options [nop,nop,TS val 889682031 ecr 889682031], length 0
```
- Развернем под для дебага ноды
```shell
kubectl debug node/kubernetes-debug -it --image=ubuntu:latest
```
- Файловая система ноды находится в `/host`
```shell
root@kubernetes-debug:~# ls /host
CHANGELOG  Release.key  bin  boot  data  dev  docker.key  etc  home  kic.txt  kind  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var  version.json
```
- Чтобы получить логи контейнера нужно перейти в `/host/var/log/pods`, название директории нужного 
пода будет похоже на `default_distroless-container-pod_5e4bccda-cda6-4bf3-b4b3-69656bb89adc`, 
заходим туда, далее в директорию с названием пода. Там будет ссылка на файл с логами, нужно 
выполнить `ls -l` и вызвать `cat` с путем к файлу, добавив `/host` в начало
```shell
root@kubernetes-debug:/host/var/log/pods/default_distroless-container-pod_5e4bccda-cda6-4bf3-b4b3-69656bb89adc/nginx-distroless# cat /host/var/lib/docker/containers/89dbb28a0db84b58738657377269f124eddd3a105f3c9f3ca625252144c36bc4/89dbb28a0db84b58738657377269f124eddd3a105f3c9f3ca625252144c36bc4-json.log
{"log":"127.0.0.1 - - [25/Aug/2026:23:18:33 +0800] \"GET / HTTP/1.1\" 200 612 \"-\" \"Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/149.0.0.0 Safari/537.36 Edg/149.0.0.0\" \"-\"\n","stream":"stdout","time":"2026-08-25T15:18:33.022178754Z"}
{"log":"2026/08/25 23:18:33 [error] 6#6: *1 open() \"/usr/share/nginx/html/favicon.ico\" failed (2: No such file or directory), client: 127.0.0.1, server: localhost, request: \"GET /favicon.ico HTTP/1.1\", host: \"localhost:8000\", referrer: \"http://localhost:8000/\"\n","stream":"stderr","time":"2026-08-25T15:18:33.12892607Z"}
{"log":"127.0.0.1 - - [25/Aug/2026:23:18:33 +0800] \"GET /favicon.ico HTTP/1.1\" 404 555 \"http://localhost:8000/\" \"Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/149.0.0.0 Safari/537.36 Edg/149.0.0.0\" \"-\"\n","stream":"stdout","time":"2026-08-25T15:18:33.128955334Z"}
{"log":"127.0.0.1 - - [25/Aug/2026:23:21:31 +0800] \"GET / HTTP/1.1\" 304 0 \"-\" \"Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/149.0.0.0 Safari/537.36 Edg/149.0.0.0\" \"-\"\n","stream":"stdout","time":"2026-08-25T15:21:31.816388342Z"}
{"log":"127.0.0.1 - - [25/Aug/2026:23:21:32 +0800] \"GET / HTTP/1.1\" 304 0 \"-\" \"Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/149.0.0.0 Safari/537.36 Edg/149.0.0.0\" \"-\"\n","stream":"stdout","time":"2026-08-25T15:21:32.54418019Z"}
```
