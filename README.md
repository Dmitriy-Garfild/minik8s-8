# Домашнее задание к занятию «Управление доступом»



### Цель задания

В тестовой среде Kubernetes нужно предоставить ограниченный доступ пользователю.

------

### Чеклист готовности к домашнему заданию

1. Установлено k8s-решение, например MicroK8S.
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым github-репозиторием.

------

### Инструменты / дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) RBAC.
2. [Пользователи и авторизация RBAC в Kubernetes](https://habr.com/ru/company/flant/blog/470503/).
3. [RBAC with Kubernetes in Minikube](https://medium.com/@HoussemDellai/rbac-with-kubernetes-in-minikube-4deed658ea7b).

------

### Задание 1. Создайте конфигурацию для подключения пользователя

1. Создайте и подпишите SSL-сертификат для подключения к кластеру.
2. Настройте конфигурационный файл kubectl для подключения.
3. Создайте роли и все необходимые настройки для пользователя.
4. Предусмотрите права пользователя. Пользователь может просматривать логи подов и их конфигурацию (`kubectl logs pod <pod_id>`, `kubectl describe pod <pod_id>`).
5. Предоставьте манифесты и скриншоты и/или вывод необходимых команд.

**Ответ.**

1. 
- Создаем серт пользователя `user1` и группой `devs`
```sh
# Приватный ключ
openssl genrsa -out user1.key 2048
# Запрос на сертификат csr для user1 и группы devs
openssl req -new -key user1.key -out user1.csr -subj "/CN=user1/O=devs"
```


  - Выполняем выпуск сертификата, который будет подписан ключом и CA сертом кластера microk8s
  ```sh
  openssl x509 -req -in user1.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out user1.crt -days 3
  ```


  - Добавляем нового пользователя в конфиг **kubectl**
  ```sh
  kubectl config set-credentials user1 \
  --client-certificate user1.crt \
  --client-key user1.key \
  --embed-certs=true
  ```
    *Можно отредактировать конфиг **kubectl**, чтобы в нем не хранилось значение сертификата и ключа. Параметры `client-certificate-data` и `client-key-data` изменить на `client-certificate` и `client-key`, а значения заменить на полные пути к файлу сертификата и ключа*
    - Добавляем новый контекст в конфиг **kubectl**. Где указываем целевой кластер и пользователя
    ```sh
    kubectl config set-context user1_context \
    --cluster microk8s-cluster --user=user1
    ```
    - Проверяем, что контекст создался и переключаемся на новый контекст
    ```sh
    kubectl config get-contexts
    kubectl config use-context user1_context
    ```
    - [Скриншот](image_1.jpg)
    
2. 
  - Средствами **kubernetes**
    - Нам необходим запрос сертификата (`.csr`) в шифрованом виде `base64`
    ```sh
    cat user2.csr | base64
    ```
    - Создаем объект кубернетеса [csr-user2.yaml](csr-user2.yaml). 
    <details><summary>csr-user2.yaml</summary>

    ```yaml
    ---
    apiVersion: certificates.k8s.io/v1
    kind: CertificateSigningRequest
    metadata:
      name: csr-user2
      labels:
        tier: lesson24
        company: netology
        owner: infernofeniks
    spec:
      groups:
      - system:authenticated
      request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1pEQ0NBVXdDQVFBd0h6RU9NQXdHQTFVRUF3d0ZkWE5sY2pJeERUQUxCZ05WQkFvTUJHUmxkbk13Z2dFaQpNQTBHQ1NxR1NJYjNEUUVCQVFVQUE0SUJEd0F3Z2dFS0FvSUJUUNiYXJ0WkdwUGhoa0ZGTVRzQmJoN0pqT2kxCkRKZzZJYnBkOXFWMHVjSllHQm1USmJzend1ZmFydHlBUzlEaEprMFloR1pEaGlzNmx5NTQxbzRFWERVbDluVU4KOFd0ejJOeG1YOXpINGZPZnJReGNibWl3cUloSjBrdUE3Mzg0cHVQc1lWZndNFdSeGZ2dkpyOExxTjhIVXRxcwpVZ0tHY1RwK2FlUWxFaDZrdGN2bTc5SUxwaTFnOVVVWURXUjJyMS9LbjNxSjZjcUpGNlhMYXkvUTgzNWhFeGY3CjFJbU1UWVNrWWVtOW9BcGJGNVNZUnJRaFdSR1VSMXVCQ1BCVHFkdS9URVRJTHpsY2hydDFRWFVQ0FOa3lFZFgKeXhPa1BoUDNiK2ViQ2JSK0Nuay9CK0tUalpteldwblNoN01rWmsyclZmK2psT0IvOFc3N1EraS9xc0diQWdNQgpBQUdnQURBTkJna3Foa2lHOXcwQkFRc0ZBQU9DQVFFQVUwVmZ2YmdpRTRFazVoMzUxclkzdGlvemszRWFRFdICnM5YzdoWTE4aTZMSlUybUJYRDMxNEdIRkcyYWIvWlFJOGVSUElPWU4vMXdrNU1qTnJPeFZOWDR3V0JQLzhtRjMKeGxvdjNmczhkNjJsa1hZVzcxZDBEWEg2TnNEY0JZRi9ibFhMd3IyVzIwbURRalljSUZCSUh0NU9uVVF0c1M1Qgp3ajEZ3VmMHB1c0EyRWlHOEFZVlBzYjl4M0JrbXJYOU8rM2VBWjBzdmZxWUdmWFkzWE9tK0NhaFY1TWZLenFaCnJsQy9WSkRwS0ZmYU03L0JpN0cwOTlEOTZwT0JnNSttVkl2Y3dKcHVvUmluVTFwd0thLzI2bkpLVXNOZUxpUHgKbUpqNFdkQmdDODBTDhiRVZnb0JxNnQ0dVRya3NQU2xpdzhya25CYWJjTFhBOFdlYzgrVGdRPT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==
      expirationSeconds: 259200 # три дня
      usages:
    - client auth
      signerName: kubernetes.io/kube-apiserver-client
    ...
    ```
    </details>

    - Проверяем наличае запроса и подтверждаем его:
    ```
    kubectl get csr
    kubectl certificate approve csr-user2
    ```
    *Состояние изменится с `Pending` на `Approved,Issued`*
    - Смотрим полученный сертификат:
    ```sh
    kubectl get csr csr-user2 -o jsonpath={.status.certificate} | base64 --decode
    ```
    - Создаем файл сертификата `user2.crt` пользователя, который мы забрали в прошлом шаге.
    ```sh
    kubectl get csr csr-user2 -o jsonpath={.status.certificate} | base64 --decode > user2.crt
    ```
    - Добавляем нового пользователя в конфиг **kubectl**
    ```sh
    kubectl config set-credentials user2 --client-certificate user2.crt --client-key user2.key --embed-certs=true
    ```
    - Добавляем новый контекст в конфиг **kubectl**. Где указываем целевой кластер и пользователя
    ```sh
    kubectl config set-context user2_context --cluster microk8s-cluster --user=user2
    ```
    - Проверяем, что контекст создался и переключаемся на новый контекст
    ```sh
    kubectl config get-contexts
    kubectl config use-context user2_context
    ```
4. Пользователь может просматривать логи подов и их конфигурацию в определенном namespace.
    - [role.yaml](role.yaml)
    - [rolebinding.yaml](rolebinding.yaml)
    - [Скриншот работы RBAC](image_2.jpg)
5. Манифесты и скриншоты.
    - [Скриншот создания и подписания сертификата](image_1.jpg)
    - [role.yaml](role.yaml)
    - [rolebinding.yaml](rolebinding.yaml)
    - [Скриншот работы RBAC](image_2.JPG)
------

### Правила приёма работы

1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

------

