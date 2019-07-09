# SergeSpinoza_platform
SergeSpinoza Platform repository

## Оглавление
1. [Домашнее задание №1](#Домашнее-задание-№1)

<br><br>
## Домашнее задание №1
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

