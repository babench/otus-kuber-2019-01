# SergeSpinoza_platform
SergeSpinoza Platform repository

## Оглавление
1. [Домашнее задание №1, Minikube](#домашнее-задание-1)
2. [Домашнее задание №2, RBAC](#домашнее-задание-2)
3. [Домашнее задание №3, Сети](#домашнее-задание-3)

<br><br>
## Домашнее задание 1
### Выполнено
- Развернут minikube ( https://kubernetes.io/docs/tasks/tools/install-minikube/ );
- Добавлен Dashboard (команда `minikube addons enable dashboard`, зайти в панель - команда `minikube dashboard`);
- Установлен k9s ( https://k9ss.io );
- Проверено, что контейнеры восстанавливаются после удаления;
- Создан Dockerfile, который запускает http сервер на 8000 порту под пользователем с uid 1001 (сделал на python); 
- Написан манифест web-pod.yaml;
- Выполнен деплой пода web и init контейнера;
- Опробован в работе kube-forwarder (https://kube-forwarder.pixelpoint.io ).


### Ответы на вопросы
1. `kube-apiserver` запускается и рестартится через Systemd Service операционной системы, а `coredns` - контролируется самим k8s через ReplicaSet; 

### Полезное
Команды: 
- `minikube start` - старт minikube;
- `minikube ssh` - войти на виртуальную машину minikube по ssh;
- `kubectl cluster-info` - проверка подключения к кластеру;
- `kubectl get pods -n kube-system` - показать все pods в namespace `kube-system`;
- `kubectl get cs` - показать состояние кластера;
- `kubectl get ...` - показать ресурсы;
- `kubectl describe ...` - показать детальную информацию о конктретном ресурсе;
- `kubectl logs ...` - показать логи контейнера в поде;
- `kubectl exec ...` - выполнить команду в контейнере в поде;
- `kubectl apply -f file.yaml` - применить манифест;  
- `kubectl get pod web -o yaml` - получить манифест уже запущенного pod;
- `kubectl port-forward --address 0.0.0.0 pod/web 8000:8000` - редирект порта (в примере 8000);


<br><br>
## Домашнее задание 2
### Выполнено
- TASK 01:
  - Создать Service Account bob, дать ему роль admin в рамках всего кластера;
  - Создать Service Account dave без доступа к кластеру;
- TASK 02:
  - Создать Namespace prometheus;
  - Создать Service Account carol в этом Namespace;
  - Дать всем Service Account в Namespace prometheus возможность делать get, list, watch в отношении Pods всего кластера;
- TASK 03:
  - Создать Namespace dev;
  - Создать Service Account jane в Namespace dev;
  - Дать jane роль admin в рамках Namespace dev;
  - Создать Service Account ken в Namespace dev;
  - Дать ken роль view в рамках Namespace dev.


### Полезное
Команды: 
- `kubectl auth can-i get/create ...` - проверка прав/возможности создания (пример `kubectl auth can-i get deployments --as system:serviceaccount:dev:ken -n dev`, `kubectl auth can-i create deployments --as system:serviceaccount:dev:ken -n dev`).


<br><br>
## Домашнее задание 3
### Выполнено
- Добавили к ранее созданному манифесту web-pod.yaml readinessProbe и livenessProbe;
- Создали манифест deployment с указанием стратегии обновления;
- Создали манифест сервиса с ClusterIP web-svc-cip;
- Переключили kube-proxy в режим `ipvs`; 
- Очистили правила iptables в minikube от мусора; 
- Настроили работу с MetalLB (к сожалению в macos доступ через браузер получить не удалось даже с пропианным маршрутом и драйвером hyperkit);
- Установили и настроили работу ingress-nginx; 

### Задание со * 
- Настроен LoadBalancer для CoreDNS (kube-dns) для 53 TCP и UDP портов с одинаковым внешним ip. Для проверки необходимо применить манифест `kubectl apply -f coredns-svc-lb.yaml` и проверить, выполнив команду из ssh minikube, например (c выводом):

```
# nslookup web-svc.default.svc.cluster.local 172.17.255.3
Server:    172.17.255.3
Address 1: 172.17.255.3 coredns-svc-lb-tcp.kube-system.svc.cluster.local

Name:      web-svc.default.svc.cluster.local
Address 1: 172.17.0.7 172-17-0-7.web-svc.default.svc.cluster.local
Address 2: 172.17.0.9 172-17-0-9.web-svc.default.svc.cluster.local
Address 3: 172.17.0.8 172-17-0-8.web-svc.default.svc.cluster.local
```
- Добавлен доступ к kubernetes-dashboard через настроенный ingress-proxy (манифес добавлен в файл `web-ingress.yaml`);
- Реализовано канареечное развертывание, манифесты `canary-namespaces.yaml` и `canary-ingress.yaml`. 

### Развертывание и проверка канареечного развертывания: 
- Применить манифест `kubectl apply -f canary-namespaces.yaml`, который создаст необходимые namespace;
- Развернуть приложение production `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/docs/examples/http-svc.yaml -n echo-production`; 
- Развернуть "тестовое" приложение `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/docs/examples/http-svc.yaml -n echo-canary`;
- Применить манифест `kubectl apply -f canary-ingress.yaml` для создания ingress для канареечного развертывания;
- Узнать ip minikube, выполнив команду `minikube ip`;
- Прописать в /etc/hosts следующую строчку: `<minikube_ip> echo.com`
- Проверить работу следующим образом: 
  - Через браузер зайти на сайт http://echo.com, обратить внимание, что строка `pod namespace:` имеет значение: `echo-production`; 
  - С помощью плагина ModHeader для браузера googleChome (ссылка https://chrome.google.com/webstore/detail/modheader/idgpnmonknjnojddfkpgkljpfnnfcklj?hl=ru) добавить заголовок `CanaryByHeader` со значением `DoCanary` и обновить страницу;
  - После этого строка `pod namespace:`должна изменить значение на: `echo-canary`.


### Ответы на вопросы
1. Почему следующая конфигурация валидна, но не имеет смысла?

```
livenessProbe:
  exec:
   command:
     - 'sh'
     - '-c'
     - 'ps aux | grep my_web_server_process'
```

**Ответ:** потому что данная проверка всегда будет отдавать процесс самой команды grep. Чтобы эта проверка работала - надо исключить из результатов grep - например так:

```
livenessProbe:
  exec:
   command:
     - 'sh'
     - '-c'
     - 'ps aux | grep -v grep | grep my_web_server_process'
```

2. Бывают ли ситуации, когда она все-таки имеет смысл?
**Ответ:** если исключать в выводе процесс самой команды grep - то это ситуации когда процесс только делает что-то внутри контейнера (допустим пишет что-то в файле на persistent volume) и не слушает никакой tcp/udp порт (т.е. проверить этот сервис по сети не возможно). 

3. Попробуйте разные варианты деплоя с крайними значениями maxSurge и maxUnavailable (оба 0, оба 100%, 0 и 100%)
**Ответ:** если maxSurge и maxUnavailable выставить в 0 - то получаем ошибку невозможности выполнения деплоя, вида 
```
The Deployment "web" is invalid: spec.strategy.rollingUpdate.maxUnavailable: Invalid value: intstr.IntOrString{Type:0, IntVal:0, StrVal:""}: may not be 0 when `maxSurge` is 0
```
При других вариантах все работает корректно, т.к. maxUnavailable задает максимальное количество недоступных подов при обновлении, а maxSurge - превышение количества подов при обновлении от заданного в `replicas`;


### Полезное
Команды: 
- `kubectl apply -f web-pod.yaml --force` - принудительное обновление;
- `kubectl --namespace kube-system edit configmap/kube-proxy` - редактирование конфига kube-proxy (для включения ipvs - выставить `mode` в `ipvs`); 
- `minikube ssh` - зайти в ssh minikube;
- `iptables --list -nv -t nat` - показать nat правила iptables;
- Внутри ssh minikube команда `toolbox` - запустить и зайти в контейнер с Fedora;
  - `dnf install -y ipvsadm && dnf clean all` - установка ipvsadm внутри контейнера Fedora;
  - `ipvsadm --list -n` - список правил ipvs;
- `kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.8.0/manifests/metallb.yaml` - установка MetalLb;
- `kubectl --namespace metallb-system logs controller-xxxxxxxxxxx-xxxxx` - логи контроллера MetalLB
- `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml` или `minikube addons enable ingress` - установка ingress-nginx;

