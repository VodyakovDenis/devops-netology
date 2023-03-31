# Домашнее задание к занятию "Введение в Terraform"

### Цель задания

1. Установить и настроить Terrafrom.
2. Научиться использовать готовый код.

------

### Чеклист готовности к домашнему заданию

1. Скачайте и установите актуальную версию **terraform**(не менее 1.3.7). Приложите скриншот вывода команды ```terraform --version```
2. Скачайте на свой ПК данный git репозиторий. Исходный код для выполнения задания расположен в директории **01/src**.
3. Убедитесь, что в вашей ОС установлен docker

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. Установка и настройка Terraform  [ссылка](https://cloud.yandex.ru/docs/tutorials/infrastructure-management/terraform-quickstart#from-yc-mirror)
2. Зеркало документации Terraform  [ссылка](https://registry.tfpla.net/browse/providers) 
3. Установка docker [ссылка](https://docs.docker.com/engine/install/ubuntu/) 
------

### Задание 1

1. Перейдите в каталог [**src**](https://github.com/netology-code/ter-homeworks/tree/main/01/src). Скачайте все необходимые зависимости, использованные в проекте. 
2. Изучите файл **.gitignore**. В каком terraform файле допустимо сохранить личную, секретную информацию?
3. Выполните код проекта. Найдите  в State-файле секретное содержимое созданного ресурса **random_password**. Пришлите его в качестве ответа.
4. Раскомментируйте блок кода, примерно расположенный на строчках 29-42 файла **main.tf**.
Выполните команду ```terraform validate```. Объясните в чем заключаются намеренно допущенные ошибки? Исправьте их.
5. Выполните код. В качестве ответа приложите вывод команды ```docker ps```
6. Замените имя docker-контейнера в блоке кода на ```hello_world```, выполните команду ```terraform apply -auto-approve```.
Объясните своими словами, в чем может быть опасность применения ключа  ```-auto-approve``` ? 
8. Уничтожьте созданные ресурсы с помощью **terraform**. Убедитесь, что все ресурсы удалены. Приложите содержимое файла **terraform.tfstate**. 
9. Объясните, почему при этом не был удален docker образ **nginx:latest** ?(Ответ найдите в коде проекта или документации)


------

### Решение

```
denis@denis-lin(0):~/netology/devops-netology/homework$ terraform --version
Terraform v1.4.2
on linux_amd64
```

1. код:
```
denis@denis-lin(0):~/netology/ter-homeworks/01/src$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding kreuzwerker/docker versions matching "~> 3.0.1"...
- Finding latest version of hashicorp/random...
- Installing kreuzwerker/docker v3.0.2...
- Installed kreuzwerker/docker v3.0.2 (self-signed, key ID BD080C4571C6104C)
- Installing hashicorp/random v3.4.3...
- Installed hashicorp/random v3.4.3 (signed by HashiCorp)
...
Terraform has been successfully initialized!
...
```

2. personal.auto.tfvars - судя по описанию

3. строчка result

``` 
denis@denis-lin(0):~/netology/ter-homeworks/01/src$ cat terraform.tfstate
{
  "version": 4,
  "terraform_version": "1.4.2",
  "serial": 1,
  "lineage": "caad6397-5077-ba2c-2086-0c3e40b1b878",
  "outputs": {},
  "resources": [
    {
      "mode": "managed",
      "type": "random_password",
      "name": "random_string",
      "provider": "provider[\"registry.terraform.io/hashicorp/random\"]",
      "instances": [
        {
          "schema_version": 3,
          "attributes": {
            "bcrypt_hash": "$2a$10$irdT9le89K.i2MNCw2ndJeNheyxIrtL3qw8.jIgnt0zHlwZ8MRTiC",
            "id": "none",
            "keepers": null,
            "length": 16,
            "lower": true,
            "min_lower": 1,
            "min_numeric": 1,
            "min_special": 0,
            "min_upper": 1,
            "number": true,
            "numeric": true,
            "override_special": null,
            "result": "pK4Yq0QcBrHuHhvm",
            "special": false,
            "upper": true
          },
          "sensitive_attributes": []
        }
      ]
    }
  ],
  "check_results": null
}

```

4. код:

```
denis@denis-lin(0):~/netology/ter-homeworks/01/src$ terraform validate
╷
│ Error: Missing name for resource
│
│   on main.tf line 24, in resource "docker_image":
│   24: resource "docker_image" {
│
│ All resource blocks must have 2 labels (type, name).
╵
╷
│ Error: Invalid resource name
│
│   on main.tf line 29, in resource "docker_container" "1nginx":
│   29: resource "docker_container" "1nginx" {
│
│ A name must start with a letter or underscore and may contain only letters, digits, underscores, and dashes.
╵
```

Ошибки в имени ресурса исправленный конфиг ниже, было остаил за #:

```
denis@denis-lin(0):~/netology/ter-homeworks/01/src$ terraform validate
Success! The configuration is valid.

denis@denis-lin(0):~/netology/ter-homeworks/01/src$ cat main.tf
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.0.1"
    }
  }
  required_version = ">=0.13" /*Многострочный комментарий.
 Требуемая версия terraform */
}
provider "docker" {}

#однострочный комментарий

resource "random_password" "random_string" {
  length      = 16
  special     = false
  min_upper   = 1
  min_lower   = 1
  min_numeric = 1
}


#resource "docker_image" {
#  name         = "nginx:latest"
#  keep_locally = true
#}

#resource "docker_container" "1nginx" {
#  image = docker_image.nginx.image_id
#  name  = "example_${random_password.random_string.result}"

#  ports {
#    internal = 80
#    external = 8000
#  }
#}

resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = true
}

resource "docker_container" "nginx" {
  name    = "example_${random_password.random_string.result}"
  image   = docker_image.nginx.image_id

  ports {
    external = 8000
    internal = 80
  }
}
```

5. код:

```
denis@denis-lin(0):~/netology/ter-homeworks/01/src$ sudo docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS         PORTS                  NAMES
f2eb0d53a4cb   a99a39d070bf   "/docker-entrypoint.…"   11 seconds ago   Up 9 seconds   0.0.0.0:8000->80/tcp   example_pK4Yq0QcBrHuHhvm
```

6. код:

```
denis@denis-lin(0):~/netology/ter-homeworks/01/src$ sudo docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                  NAMES
cebcb27477cf   a99a39d070bf   "/docker-entrypoint.…"   30 seconds ago   Up 28 seconds   0.0.0.0:8000->80/tcp   hello_world_pK4Yq0QcBrHuHhvm
```

Судя по документации и на примере: 
	-auto-approve - Skips interactive approval of plan before applying. This option is ignored when you pass a previously-saved plan file, because Terraform considers you passing the plan file as the approval and so will never prompt in that case.   

terraform не запрашивает подтверждения перед применением, я уже после того как он выполнил команду смотрел что изменится, соответсвенно может быть ситуация когда что-то сломается, и сразу будет не ясно что вприницпе изменилось, скажем случайно гдето добавлен символ который был упущен из виду, или еще какая то делать которая может повлиять на конечный результат.   

7. код:

```
denis@denis-lin(0):~/netology/ter-homeworks/01/src$ sudo docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
denis@denis-lin(0):~/netology/ter-homeworks/01/src$ cat terraform.tfstate
{
  "version": 4,
  "terraform_version": "1.4.2",
  "serial": 11,
  "lineage": "caad6397-5077-ba2c-2086-0c3e40b1b878",
  "outputs": {},
  "resources": [],
  "check_results": null
}
```

8. Насколько я понял все дело в keep_locally = false / true   
Как вариант при значении false можем получить ошибку если например один и тот же образ используется другим работающим в данный момент контейнером.
