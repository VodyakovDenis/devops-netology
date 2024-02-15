# Дипломная работа

> ## Цели:
> 1. Подготовить облачную инфраструктуру на базе облачного провайдера Яндекс.Облако.
> 2. Запустить и сконфигурировать Kubernetes кластер.
> 3. Установить и настроить систему мониторинга.
> 4. Настроить и автоматизировать сборку тестового приложения с использованием Docker-контейнеров.
> 5. Настроить CI для автоматической сборки и тестирования.
> 6. Настроить CD для автоматического развёртывания приложения.

---
## Этапы выполнения:

### Создание облачной инфраструктуры

>**Ожидаемые результаты:**
> 1. Terraform сконфигурирован и создание инфраструктуры посредством Terraform возможно без дополнительных ручных действий.
> 3. Полученная конфигурация инфраструктуры является предварительной, поэтому в ходе дальнейшего выполнения задания возможны изменения.

Предварительная настройка включает в себя несколько шагов, необходимых для последующей работы с yandex.cloud через terraform.    
* Установить утилиту **yc** и подключится к облаку.   
* Создание сервисного аккаунта с ролью editor на дефолтной директории облака   
* Создание s3-bucket для хранения состояния terraform   
        Далее необходимо инициализировать terraform   

<details><summary>Пример</summary>

**Установка** [yc](https://cloud.yandex.ru/ru/docs/cli/quickstart) 

* Ответы на вопросы и порядок работы довольно не плохо описан в статье: [Начало работы с Terraform](https://cloud.yandex.ru/ru/docs/tutorials/infrastructure-management/terraform-quickstart)

```bash
curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh | bash
```

**Создание сервисного аккаунта с ролью editor на дефолтной директории облака**

```bash
yc iam service-account create --name terraform-acc
yc resource-manager folder add-access-binding --name default --role editor --subject "serviceAccount:<accId>"
```
* accId - это уникальный идентификатор нового сервисного аккаунта

**Получаем ключ доступа для данного сервисного аккаунта**

```bash
yc iam access-key create --service-account-name terraform-acc --format=json
```

**Создание s3-bucket для хранения состояния terraform**

```bash
yc storage bucket create --name=netology-state
```

**Инициализация terraform**

```bash
terraform init \
  -backend-config="bucket=netology-state" \
  -backend-config="access_key=<service_account_key_id>" \
  -backend-config="secret_key=<service_account_secret_key>"

```
* где **service_account_key_id** и **service_account_secret_key** данные полученные на шаге получения ключа доступа для сервисного аккаунта

**Создание и переключение на новый workspace**

```bash
terraform workspace new diplom
```

Данные шаги мной выполнялись в ручную. 
</details>
    
**Решение: Создание VPC и подсетей через terraform**

<details><summary>network.tf</summary>

```bash
resource "yandex_vpc_network" "diplom-network" {
  name = "diplom-network"
}

resource "yandex_vpc_subnet" "diplom-network-subnet-a" {
  name           = "diplom-network-subnet-a"
  zone           = "ru-central1-a"
  network_id     = yandex_vpc_network.diplom-network.id
  v4_cidr_blocks = ["192.168.10.0/24"]
}

resource "yandex_vpc_subnet" "diplom-network-subnet-b" {
  name           = "diplom-network-subnet-b"
  zone           = "ru-central1-b"
  network_id     = yandex_vpc_network.diplom-network.id
  v4_cidr_blocks = ["192.168.11.0/24"]
}
```

</details>

<details><summary>terraform plan</summary>

```bash
denis@denis-lin(0):~/netology/diplom$ terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # yandex_vpc_network.diplom-network will be created
  + resource "yandex_vpc_network" "diplom-network" {
      + created_at                = (known after apply)
      + default_security_group_id = (known after apply)
      + folder_id                 = (known after apply)
      + id                        = (known after apply)
      + labels                    = (known after apply)
      + name                      = "diplom-network"
      + subnet_ids                = (known after apply)
    }

  # yandex_vpc_subnet.diplom-network-subnet-a will be created
  + resource "yandex_vpc_subnet" "diplom-network-subnet-a" {
      + created_at     = (known after apply)
      + folder_id      = (known after apply)
      + id             = (known after apply)
      + labels         = (known after apply)
      + name           = "diplom-network-subnet-a"
      + network_id     = (known after apply)
      + v4_cidr_blocks = [
          + "192.168.10.0/24",
        ]
      + v6_cidr_blocks = (known after apply)
      + zone           = "ru-central1-a"
    }

  # yandex_vpc_subnet.diplom-network-subnet-b will be created
  + resource "yandex_vpc_subnet" "diplom-network-subnet-b" {
      + created_at     = (known after apply)
      + folder_id      = (known after apply)
      + id             = (known after apply)
      + labels         = (known after apply)
      + name           = "diplom-network-subnet-b"
      + network_id     = (known after apply)
      + v4_cidr_blocks = [
          + "192.168.11.0/24",
        ]
      + v6_cidr_blocks = (known after apply)
      + zone           = "ru-central1-b"
    }

Plan: 3 to add, 0 to change, 0 to destroy.

────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.

```

</details>

<details><summary>terraform apply</summary>

```bash
denis@denis-lin(0):~/netology/diplom$ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # yandex_vpc_network.diplom-network will be created
  + resource "yandex_vpc_network" "diplom-network" {
      + created_at                = (known after apply)
      + default_security_group_id = (known after apply)
      + folder_id                 = (known after apply)
      + id                        = (known after apply)
      + labels                    = (known after apply)
      + name                      = "diplom-network"
      + subnet_ids                = (known after apply)
    }

  # yandex_vpc_subnet.diplom-network-subnet-a will be created
  + resource "yandex_vpc_subnet" "diplom-network-subnet-a" {
      + created_at     = (known after apply)
      + folder_id      = (known after apply)
      + id             = (known after apply)
      + labels         = (known after apply)
      + name           = "diplom-network-subnet-a"
      + network_id     = (known after apply)
      + v4_cidr_blocks = [
          + "192.168.10.0/24",
        ]
      + v6_cidr_blocks = (known after apply)
      + zone           = "ru-central1-a"
    }

  # yandex_vpc_subnet.diplom-network-subnet-b will be created
  + resource "yandex_vpc_subnet" "diplom-network-subnet-b" {
      + created_at     = (known after apply)
      + folder_id      = (known after apply)
      + id             = (known after apply)
      + labels         = (known after apply)
      + name           = "diplom-network-subnet-b"
      + network_id     = (known after apply)
      + v4_cidr_blocks = [
          + "192.168.11.0/24",
        ]
      + v6_cidr_blocks = (known after apply)
      + zone           = "ru-central1-b"
    }

Plan: 3 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

yandex_vpc_network.diplom-network: Creating...
yandex_vpc_network.diplom-network: Creation complete after 2s [id=enpvbm2m8t8b8bp5omk6]
yandex_vpc_subnet.diplom-network-subnet-a: Creating...
yandex_vpc_subnet.diplom-network-subnet-b: Creating...
yandex_vpc_subnet.diplom-network-subnet-a: Creation complete after 1s [id=e9bnvfvdpli6jqlsgopc]
yandex_vpc_subnet.diplom-network-subnet-b: Creation complete after 2s [id=e2lsblaev3lcj8hcotmd]

Apply complete! Resources: 3 added, 0 changed, 0 destroyed.

```


![img1](img/img1.png)

![img2](img/img2.png)

</details>

<details><summary>terraform destroy</summary>

```bash
denis@denis-lin(0):~/netology/diplom$ terraform destroy
yandex_vpc_network.diplom-network: Refreshing state... [id=enpvbm2m8t8b8bp5omk6]
yandex_vpc_subnet.diplom-network-subnet-b: Refreshing state... [id=e2lsblaev3lcj8hcotmd]
yandex_vpc_subnet.diplom-network-subnet-a: Refreshing state... [id=e9bnvfvdpli6jqlsgopc]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # yandex_vpc_network.diplom-network will be destroyed
  - resource "yandex_vpc_network" "diplom-network" {
      - created_at                = "2024-02-03T06:34:41Z" -> null
      - default_security_group_id = "enp94bj339ltkrvgsjnv" -> null
      - folder_id                 = "b1gfkh66dgk5tpn7ajko" -> null
      - id                        = "enpvbm2m8t8b8bp5omk6" -> null
      - labels                    = {} -> null
      - name                      = "diplom-network" -> null
      - subnet_ids                = [
          - "e2lsblaev3lcj8hcotmd",
          - "e9bnvfvdpli6jqlsgopc",
        ] -> null
    }

  # yandex_vpc_subnet.diplom-network-subnet-a will be destroyed
  - resource "yandex_vpc_subnet" "diplom-network-subnet-a" {
      - created_at     = "2024-02-03T06:34:43Z" -> null
      - folder_id      = "b1gfkh66dgk5tpn7ajko" -> null
      - id             = "e9bnvfvdpli6jqlsgopc" -> null
      - labels         = {} -> null
      - name           = "diplom-network-subnet-a" -> null
      - network_id     = "enpvbm2m8t8b8bp5omk6" -> null
      - v4_cidr_blocks = [
          - "192.168.10.0/24",
        ] -> null
      - v6_cidr_blocks = [] -> null
      - zone           = "ru-central1-a" -> null
    }

  # yandex_vpc_subnet.diplom-network-subnet-b will be destroyed
  - resource "yandex_vpc_subnet" "diplom-network-subnet-b" {
      - created_at     = "2024-02-03T06:34:44Z" -> null
      - folder_id      = "b1gfkh66dgk5tpn7ajko" -> null
      - id             = "e2lsblaev3lcj8hcotmd" -> null
      - labels         = {} -> null
      - name           = "diplom-network-subnet-b" -> null
      - network_id     = "enpvbm2m8t8b8bp5omk6" -> null
      - v4_cidr_blocks = [
          - "192.168.11.0/24",
        ] -> null
      - v6_cidr_blocks = [] -> null
      - zone           = "ru-central1-b" -> null
    }

Plan: 0 to add, 0 to change, 3 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

yandex_vpc_subnet.diplom-network-subnet-b: Destroying... [id=e2lsblaev3lcj8hcotmd]
yandex_vpc_subnet.diplom-network-subnet-a: Destroying... [id=e9bnvfvdpli6jqlsgopc]
yandex_vpc_subnet.diplom-network-subnet-b: Destruction complete after 1s
yandex_vpc_subnet.diplom-network-subnet-a: Destruction complete after 1s
yandex_vpc_network.diplom-network: Destroying... [id=enpvbm2m8t8b8bp5omk6]
yandex_vpc_network.diplom-network: Destruction complete after 1s

Destroy complete! Resources: 3 destroyed.

```

</details>

---

## Создание Kubernetes кластера

> **Ожидаемый результат:**
> 1. Работоспособный Kubernetes кластер.
> 2. В файле ~/.kube/config находятся данные для доступа к кластеру.
> 3. Команда kubectl get pods --all-namespaces отрабатывает без ошибок.



За основу возьмем минимальную конфигурацию.

<details><summary>instances.tf</summary>

```bash
resource "random_shuffle" "diplom-network-subnet-random" {
  input        = [yandex_vpc_subnet.diplom-network-subnet-a.id, yandex_vpc_subnet.diplom-network-subnet-b.id]
  result_count = 1
}

resource "yandex_compute_instance" "k8s-cluster" {
  for_each = toset(["k8s-node1", "k8s-node2", "k8s-node3"])

  name = each.key
  hostname = "${each.key}.diplom.local"

  resources {
    cores  = 2
    memory = 2
  }

  boot_disk {
    initialize_params {
      image_id = "fd8kdq6d0p8sij7h5qe3" # ubuntu-20-04-lts-v20220822
      size = "20"
    }
  }

  network_interface {
    subnet_id = random_shuffle.diplom-network-subnet-random.result[0]
    nat       = true
  }

  metadata = {
    serial-port-enable = 1
    ssh-keys = "ubuntu:${file("~/.ssh/id_rsa.pub")}"
  }
}

output "cluster_ips" {
  value = {
    internal = values(yandex_compute_instance.k8s-cluster)[*].network_interface.0.ip_address
    external = values(yandex_compute_instance.k8s-cluster)[*].network_interface.0.nat_ip_address
  }
}
```

</details>

Для распределения по зонам доступности использован ***random_shuffle***   

<details><summary>terraform plan</summary>

```bash
denis@denis-lin(0):~/netology/diplom$ terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # random_shuffle.diplom-network-subnet-random will be created
  + resource "random_shuffle" "diplom-network-subnet-random" {
      + id           = (known after apply)
      + input        = [
          + (known after apply),
          + (known after apply),
        ]
      + result       = (known after apply)
      + result_count = 1
    }

  # yandex_compute_instance.k8s-cluster["k8s-node1"] will be created
  + resource "yandex_compute_instance" "k8s-cluster" {
      + created_at                = (known after apply)
      + folder_id                 = (known after apply)
      + fqdn                      = (known after apply)
      + gpu_cluster_id            = (known after apply)
      + hostname                  = "k8s-node1.diplom.local"
      + id                        = (known after apply)
      + maintenance_grace_period  = (known after apply)
      + maintenance_policy        = (known after apply)
      + metadata                  = {
          + "serial-port-enable" = "1"
          + "ssh-keys"           = <<-EOT
                ubuntu:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC7KoRsmTb6EC67YSxCrvoqTLhZcBgEd7lM4LebHU5p7qBtvyxA+oUxkH1Z4HjR2zbKEzDqtFfgREN5gaMFzHG/pa7RgMPR1c5lBynlsCzHDQqKt9417IoXfT71bFo2Mh+wXHo9gcJw9tU+4pab93zMBN7a40XtKWYQFsSQ0TffpprCcBkkqBeuBS9BFh3oaL7hRIOWPmUwpXmhY+Te7oaluj8fPxq/2ahPF1kthLY1NTLQvRGbrxENxsysi0jZwfDkYzWdWqhI5BA8fyG8e1wcpTM+9U8F4sAGMBxI5vEcC1J/Eb940tnEfp+oeZzcrSH9IRVCOgyGnEBh/qloxP7HVlErpX7KGf8h6s18h4d4mi5ylgBBgYfn/Wszej+Yotdgc+TJYihe/dDRulsxEcU/UQjIheqIoIIZApnajk0GdEyOtZpU/cY+CrtBH2czMLlNpn4lEOkcaCQu2XeqKn2yQNiOqz8jsDrVrTRrJHTEZ7+kkUROqmuNtcJdWl+4YH0= denis@denis-lin
            EOT
        }
      + name                      = "k8s-node1"
      + network_acceleration_type = "standard"
      + platform_id               = "standard-v1"
      + service_account_id        = (known after apply)
      + status                    = (known after apply)
      + zone                      = (known after apply)

      + boot_disk {
          + auto_delete = true
          + device_name = (known after apply)
          + disk_id     = (known after apply)
          + mode        = (known after apply)

          + initialize_params {
              + block_size  = (known after apply)
              + description = (known after apply)
              + image_id    = "fd8kdq6d0p8sij7h5qe3"
              + name        = (known after apply)
              + size        = 20
              + snapshot_id = (known after apply)
              + type        = "network-hdd"
            }
        }

      + network_interface {
          + index              = (known after apply)
          + ip_address         = (known after apply)
          + ipv4               = true
          + ipv6               = (known after apply)
          + ipv6_address       = (known after apply)
          + mac_address        = (known after apply)
          + nat                = true
          + nat_ip_address     = (known after apply)
          + nat_ip_version     = (known after apply)
          + security_group_ids = (known after apply)
          + subnet_id          = (known after apply)
        }

      + resources {
          + core_fraction = 100
          + cores         = 2
          + memory        = 2
        }
    }

  # yandex_compute_instance.k8s-cluster["k8s-node2"] will be created
  + resource "yandex_compute_instance" "k8s-cluster" {
      + created_at                = (known after apply)
      + folder_id                 = (known after apply)
      + fqdn                      = (known after apply)
      + gpu_cluster_id            = (known after apply)
      + hostname                  = "k8s-node2.diplom.local"
      + id                        = (known after apply)
      + maintenance_grace_period  = (known after apply)
      + maintenance_policy        = (known after apply)
      + metadata                  = {
          + "serial-port-enable" = "1"
          + "ssh-keys"           = <<-EOT
                ubuntu:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC7KoRsmTb6EC67YSxCrvoqTLhZcBgEd7lM4LebHU5p7qBtvyxA+oUxkH1Z4HjR2zbKEzDqtFfgREN5gaMFzHG/pa7RgMPR1c5lBynlsCzHDQqKt9417IoXfT71bFo2Mh+wXHo9gcJw9tU+4pab93zMBN7a40XtKWYQFsSQ0TffpprCcBkkqBeuBS9BFh3oaL7hRIOWPmUwpXmhY+Te7oaluj8fPxq/2ahPF1kthLY1NTLQvRGbrxENxsysi0jZwfDkYzWdWqhI5BA8fyG8e1wcpTM+9U8F4sAGMBxI5vEcC1J/Eb940tnEfp+oeZzcrSH9IRVCOgyGnEBh/qloxP7HVlErpX7KGf8h6s18h4d4mi5ylgBBgYfn/Wszej+Yotdgc+TJYihe/dDRulsxEcU/UQjIheqIoIIZApnajk0GdEyOtZpU/cY+CrtBH2czMLlNpn4lEOkcaCQu2XeqKn2yQNiOqz8jsDrVrTRrJHTEZ7+kkUROqmuNtcJdWl+4YH0= denis@denis-lin
            EOT
        }
      + name                      = "k8s-node2"
      + network_acceleration_type = "standard"
      + platform_id               = "standard-v1"
      + service_account_id        = (known after apply)
      + status                    = (known after apply)
      + zone                      = (known after apply)

      + boot_disk {
          + auto_delete = true
          + device_name = (known after apply)
          + disk_id     = (known after apply)
          + mode        = (known after apply)

          + initialize_params {
              + block_size  = (known after apply)
              + description = (known after apply)
              + image_id    = "fd8kdq6d0p8sij7h5qe3"
              + name        = (known after apply)
              + size        = 20
              + snapshot_id = (known after apply)
              + type        = "network-hdd"
            }
        }

      + network_interface {
          + index              = (known after apply)
          + ip_address         = (known after apply)
          + ipv4               = true
          + ipv6               = (known after apply)
          + ipv6_address       = (known after apply)
          + mac_address        = (known after apply)
          + nat                = true
          + nat_ip_address     = (known after apply)
          + nat_ip_version     = (known after apply)
          + security_group_ids = (known after apply)
          + subnet_id          = (known after apply)
        }

      + resources {
          + core_fraction = 100
          + cores         = 2
          + memory        = 2
        }
    }

  # yandex_compute_instance.k8s-cluster["k8s-node3"] will be created
  + resource "yandex_compute_instance" "k8s-cluster" {
      + created_at                = (known after apply)
      + folder_id                 = (known after apply)
      + fqdn                      = (known after apply)
      + gpu_cluster_id            = (known after apply)
      + hostname                  = "k8s-node3.diplom.local"
      + id                        = (known after apply)
      + maintenance_grace_period  = (known after apply)
      + maintenance_policy        = (known after apply)
      + metadata                  = {
          + "serial-port-enable" = "1"
          + "ssh-keys"           = <<-EOT
                ubuntu:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC7KoRsmTb6EC67YSxCrvoqTLhZcBgEd7lM4LebHU5p7qBtvyxA+oUxkH1Z4HjR2zbKEzDqtFfgREN5gaMFzHG/pa7RgMPR1c5lBynlsCzHDQqKt9417IoXfT71bFo2Mh+wXHo9gcJw9tU+4pab93zMBN7a40XtKWYQFsSQ0TffpprCcBkkqBeuBS9BFh3oaL7hRIOWPmUwpXmhY+Te7oaluj8fPxq/2ahPF1kthLY1NTLQvRGbrxENxsysi0jZwfDkYzWdWqhI5BA8fyG8e1wcpTM+9U8F4sAGMBxI5vEcC1J/Eb940tnEfp+oeZzcrSH9IRVCOgyGnEBh/qloxP7HVlErpX7KGf8h6s18h4d4mi5ylgBBgYfn/Wszej+Yotdgc+TJYihe/dDRulsxEcU/UQjIheqIoIIZApnajk0GdEyOtZpU/cY+CrtBH2czMLlNpn4lEOkcaCQu2XeqKn2yQNiOqz8jsDrVrTRrJHTEZ7+kkUROqmuNtcJdWl+4YH0= denis@denis-lin
            EOT
        }
      + name                      = "k8s-node3"
      + network_acceleration_type = "standard"
      + platform_id               = "standard-v1"
      + service_account_id        = (known after apply)
      + status                    = (known after apply)
      + zone                      = (known after apply)

      + boot_disk {
          + auto_delete = true
          + device_name = (known after apply)
          + disk_id     = (known after apply)
          + mode        = (known after apply)

          + initialize_params {
              + block_size  = (known after apply)
              + description = (known after apply)
              + image_id    = "fd8kdq6d0p8sij7h5qe3"
              + name        = (known after apply)
              + size        = 20
              + snapshot_id = (known after apply)
              + type        = "network-hdd"
            }
        }

      + network_interface {
          + index              = (known after apply)
          + ip_address         = (known after apply)
          + ipv4               = true
          + ipv6               = (known after apply)
          + ipv6_address       = (known after apply)
          + mac_address        = (known after apply)
          + nat                = true
          + nat_ip_address     = (known after apply)
          + nat_ip_version     = (known after apply)
          + security_group_ids = (known after apply)
          + subnet_id          = (known after apply)
        }

      + resources {
          + core_fraction = 100
          + cores         = 2
          + memory        = 2
        }
    }

  # yandex_vpc_network.diplom-network will be created
  + resource "yandex_vpc_network" "diplom-network" {
      + created_at                = (known after apply)
      + default_security_group_id = (known after apply)
      + folder_id                 = (known after apply)
      + id                        = (known after apply)
      + labels                    = (known after apply)
      + name                      = "diplom-network"
      + subnet_ids                = (known after apply)
    }

  # yandex_vpc_subnet.diplom-network-subnet-a will be created
  + resource "yandex_vpc_subnet" "diplom-network-subnet-a" {
      + created_at     = (known after apply)
      + folder_id      = (known after apply)
      + id             = (known after apply)
      + labels         = (known after apply)
      + name           = "diplom-network-subnet-a"
      + network_id     = (known after apply)
      + v4_cidr_blocks = [
          + "192.168.10.0/24",
        ]
      + v6_cidr_blocks = (known after apply)
      + zone           = "ru-central1-a"
    }

  # yandex_vpc_subnet.diplom-network-subnet-b will be created
  + resource "yandex_vpc_subnet" "diplom-network-subnet-b" {
      + created_at     = (known after apply)
      + folder_id      = (known after apply)
      + id             = (known after apply)
      + labels         = (known after apply)
      + name           = "diplom-network-subnet-b"
      + network_id     = (known after apply)
      + v4_cidr_blocks = [
          + "192.168.11.0/24",
        ]
      + v6_cidr_blocks = (known after apply)
      + zone           = "ru-central1-b"
    }

Plan: 7 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + cluster_ips = {
      + external = [
          + (known after apply),
          + (known after apply),
          + (known after apply),
        ]
      + internal = [
          + (known after apply),
          + (known after apply),
          + (known after apply),
        ]
    }

────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.

```

</details>

<details><summary>terraform apply</summary>

```bash
denis@denis-lin(0):~/netology/diplom$ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # random_shuffle.diplom-network-subnet-random will be created
  + resource "random_shuffle" "diplom-network-subnet-random" {
      + id           = (known after apply)
      + input        = [
          + (known after apply),
          + (known after apply),
        ]
      + result       = (known after apply)
      + result_count = 1
    }

  # yandex_compute_instance.k8s-cluster["k8s-node1"] will be created
  + resource "yandex_compute_instance" "k8s-cluster" {
      + created_at                = (known after apply)
      + folder_id                 = (known after apply)
      + fqdn                      = (known after apply)
      + gpu_cluster_id            = (known after apply)
      + hostname                  = "k8s-node1.diplom.local"
      + id                        = (known after apply)
      + maintenance_grace_period  = (known after apply)
      + maintenance_policy        = (known after apply)
      + metadata                  = {
          + "serial-port-enable" = "1"
          + "ssh-keys"           = <<-EOT
                ubuntu:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC7KoRsmTb6EC67YSxCrvoqTLhZcBgEd7lM4LebHU5p7qBtvyxA+oUxkH1Z4HjR2zbKEzDqtFfgREN5gaMFzHG/pa7RgMPR1c5lBynlsCzHDQqKt9417IoXfT71bFo2Mh+wXHo9gcJw9tU+4pab93zMBN7a40XtKWYQFsSQ0TffpprCcBkkqBeuBS9BFh3oaL7hRIOWPmUwpXmhY+Te7oaluj8fPxq/2ahPF1kthLY1NTLQvRGbrxENxsysi0jZwfDkYzWdWqhI5BA8fyG8e1wcpTM+9U8F4sAGMBxI5vEcC1J/Eb940tnEfp+oeZzcrSH9IRVCOgyGnEBh/qloxP7HVlErpX7KGf8h6s18h4d4mi5ylgBBgYfn/Wszej+Yotdgc+TJYihe/dDRulsxEcU/UQjIheqIoIIZApnajk0GdEyOtZpU/cY+CrtBH2czMLlNpn4lEOkcaCQu2XeqKn2yQNiOqz8jsDrVrTRrJHTEZ7+kkUROqmuNtcJdWl+4YH0= denis@denis-lin
            EOT
        }
      + name                      = "k8s-node1"
      + network_acceleration_type = "standard"
      + platform_id               = "standard-v1"
      + service_account_id        = (known after apply)
      + status                    = (known after apply)
      + zone                      = (known after apply)

      + boot_disk {
          + auto_delete = true
          + device_name = (known after apply)
          + disk_id     = (known after apply)
          + mode        = (known after apply)

          + initialize_params {
              + block_size  = (known after apply)
              + description = (known after apply)
              + image_id    = "fd8kdq6d0p8sij7h5qe3"
              + name        = (known after apply)
              + size        = 20
              + snapshot_id = (known after apply)
              + type        = "network-hdd"
            }
        }

      + network_interface {
          + index              = (known after apply)
          + ip_address         = (known after apply)
          + ipv4               = true
          + ipv6               = (known after apply)
          + ipv6_address       = (known after apply)
          + mac_address        = (known after apply)
          + nat                = true
          + nat_ip_address     = (known after apply)
          + nat_ip_version     = (known after apply)
          + security_group_ids = (known after apply)
          + subnet_id          = (known after apply)
        }

      + resources {
          + core_fraction = 100
          + cores         = 2
          + memory        = 2
        }
    }

  # yandex_compute_instance.k8s-cluster["k8s-node2"] will be created
  + resource "yandex_compute_instance" "k8s-cluster" {
      + created_at                = (known after apply)
      + folder_id                 = (known after apply)
      + fqdn                      = (known after apply)
      + gpu_cluster_id            = (known after apply)
      + hostname                  = "k8s-node2.diplom.local"
      + id                        = (known after apply)
      + maintenance_grace_period  = (known after apply)
      + maintenance_policy        = (known after apply)
      + metadata                  = {
          + "serial-port-enable" = "1"
          + "ssh-keys"           = <<-EOT
                ubuntu:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC7KoRsmTb6EC67YSxCrvoqTLhZcBgEd7lM4LebHU5p7qBtvyxA+oUxkH1Z4HjR2zbKEzDqtFfgREN5gaMFzHG/pa7RgMPR1c5lBynlsCzHDQqKt9417IoXfT71bFo2Mh+wXHo9gcJw9tU+4pab93zMBN7a40XtKWYQFsSQ0TffpprCcBkkqBeuBS9BFh3oaL7hRIOWPmUwpXmhY+Te7oaluj8fPxq/2ahPF1kthLY1NTLQvRGbrxENxsysi0jZwfDkYzWdWqhI5BA8fyG8e1wcpTM+9U8F4sAGMBxI5vEcC1J/Eb940tnEfp+oeZzcrSH9IRVCOgyGnEBh/qloxP7HVlErpX7KGf8h6s18h4d4mi5ylgBBgYfn/Wszej+Yotdgc+TJYihe/dDRulsxEcU/UQjIheqIoIIZApnajk0GdEyOtZpU/cY+CrtBH2czMLlNpn4lEOkcaCQu2XeqKn2yQNiOqz8jsDrVrTRrJHTEZ7+kkUROqmuNtcJdWl+4YH0= denis@denis-lin
            EOT
        }
      + name                      = "k8s-node2"
      + network_acceleration_type = "standard"
      + platform_id               = "standard-v1"
      + service_account_id        = (known after apply)
      + status                    = (known after apply)
      + zone                      = (known after apply)

      + boot_disk {
          + auto_delete = true
          + device_name = (known after apply)
          + disk_id     = (known after apply)
          + mode        = (known after apply)

          + initialize_params {
              + block_size  = (known after apply)
              + description = (known after apply)
              + image_id    = "fd8kdq6d0p8sij7h5qe3"
              + name        = (known after apply)
              + size        = 20
              + snapshot_id = (known after apply)
              + type        = "network-hdd"
            }
        }

      + network_interface {
          + index              = (known after apply)
          + ip_address         = (known after apply)
          + ipv4               = true
          + ipv6               = (known after apply)
          + ipv6_address       = (known after apply)
          + mac_address        = (known after apply)
          + nat                = true
          + nat_ip_address     = (known after apply)
          + nat_ip_version     = (known after apply)
          + security_group_ids = (known after apply)
          + subnet_id          = (known after apply)
        }

      + resources {
          + core_fraction = 100
          + cores         = 2
          + memory        = 2
        }
    }

  # yandex_compute_instance.k8s-cluster["k8s-node3"] will be created
  + resource "yandex_compute_instance" "k8s-cluster" {
      + created_at                = (known after apply)
      + folder_id                 = (known after apply)
      + fqdn                      = (known after apply)
      + gpu_cluster_id            = (known after apply)
      + hostname                  = "k8s-node3.diplom.local"
      + id                        = (known after apply)
      + maintenance_grace_period  = (known after apply)
      + maintenance_policy        = (known after apply)
      + metadata                  = {
          + "serial-port-enable" = "1"
          + "ssh-keys"           = <<-EOT
                ubuntu:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC7KoRsmTb6EC67YSxCrvoqTLhZcBgEd7lM4LebHU5p7qBtvyxA+oUxkH1Z4HjR2zbKEzDqtFfgREN5gaMFzHG/pa7RgMPR1c5lBynlsCzHDQqKt9417IoXfT71bFo2Mh+wXHo9gcJw9tU+4pab93zMBN7a40XtKWYQFsSQ0TffpprCcBkkqBeuBS9BFh3oaL7hRIOWPmUwpXmhY+Te7oaluj8fPxq/2ahPF1kthLY1NTLQvRGbrxENxsysi0jZwfDkYzWdWqhI5BA8fyG8e1wcpTM+9U8F4sAGMBxI5vEcC1J/Eb940tnEfp+oeZzcrSH9IRVCOgyGnEBh/qloxP7HVlErpX7KGf8h6s18h4d4mi5ylgBBgYfn/Wszej+Yotdgc+TJYihe/dDRulsxEcU/UQjIheqIoIIZApnajk0GdEyOtZpU/cY+CrtBH2czMLlNpn4lEOkcaCQu2XeqKn2yQNiOqz8jsDrVrTRrJHTEZ7+kkUROqmuNtcJdWl+4YH0= denis@denis-lin
            EOT
        }
      + name                      = "k8s-node3"
      + network_acceleration_type = "standard"
      + platform_id               = "standard-v1"
      + service_account_id        = (known after apply)
      + status                    = (known after apply)
      + zone                      = (known after apply)

      + boot_disk {
          + auto_delete = true
          + device_name = (known after apply)
          + disk_id     = (known after apply)
          + mode        = (known after apply)

          + initialize_params {
              + block_size  = (known after apply)
              + description = (known after apply)
              + image_id    = "fd8kdq6d0p8sij7h5qe3"
              + name        = (known after apply)
              + size        = 20
              + snapshot_id = (known after apply)
              + type        = "network-hdd"
            }
        }

      + network_interface {
          + index              = (known after apply)
          + ip_address         = (known after apply)
          + ipv4               = true
          + ipv6               = (known after apply)
          + ipv6_address       = (known after apply)
          + mac_address        = (known after apply)
          + nat                = true
          + nat_ip_address     = (known after apply)
          + nat_ip_version     = (known after apply)
          + security_group_ids = (known after apply)
          + subnet_id          = (known after apply)
        }

      + resources {
          + core_fraction = 100
          + cores         = 2
          + memory        = 2
        }
    }

  # yandex_vpc_network.diplom-network will be created
  + resource "yandex_vpc_network" "diplom-network" {
      + created_at                = (known after apply)
      + default_security_group_id = (known after apply)
      + folder_id                 = (known after apply)
      + id                        = (known after apply)
      + labels                    = (known after apply)
      + name                      = "diplom-network"
      + subnet_ids                = (known after apply)
    }

  # yandex_vpc_subnet.diplom-network-subnet-a will be created
  + resource "yandex_vpc_subnet" "diplom-network-subnet-a" {
      + created_at     = (known after apply)
      + folder_id      = (known after apply)
      + id             = (known after apply)
      + labels         = (known after apply)
      + name           = "diplom-network-subnet-a"
      + network_id     = (known after apply)
      + v4_cidr_blocks = [
          + "192.168.10.0/24",
        ]
      + v6_cidr_blocks = (known after apply)
      + zone           = "ru-central1-a"
    }

  # yandex_vpc_subnet.diplom-network-subnet-b will be created
  + resource "yandex_vpc_subnet" "diplom-network-subnet-b" {
      + created_at     = (known after apply)
      + folder_id      = (known after apply)
      + id             = (known after apply)
      + labels         = (known after apply)
      + name           = "diplom-network-subnet-b"
      + network_id     = (known after apply)
      + v4_cidr_blocks = [
          + "192.168.11.0/24",
        ]
      + v6_cidr_blocks = (known after apply)
      + zone           = "ru-central1-b"
    }

Plan: 7 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + cluster_ips = {
      + external = [
          + (known after apply),
          + (known after apply),
          + (known after apply),
        ]
      + internal = [
          + (known after apply),
          + (known after apply),
          + (known after apply),
        ]
    }

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

yandex_vpc_network.diplom-network: Creating...
yandex_vpc_network.diplom-network: Creation complete after 3s [id=enpvii24n7ibl5a0aeq7]
yandex_vpc_subnet.diplom-network-subnet-a: Creating...
yandex_vpc_subnet.diplom-network-subnet-b: Creating...
yandex_vpc_subnet.diplom-network-subnet-a: Creation complete after 1s [id=e9b402228ol6e0l6118p]
yandex_vpc_subnet.diplom-network-subnet-b: Creation complete after 2s [id=e2l26vuodeclji2ou2bg]
random_shuffle.diplom-network-subnet-random: Creating...
random_shuffle.diplom-network-subnet-random: Creation complete after 0s [id=-]
yandex_compute_instance.k8s-cluster["k8s-node2"]: Creating...
yandex_compute_instance.k8s-cluster["k8s-node3"]: Creating...
yandex_compute_instance.k8s-cluster["k8s-node1"]: Creating...
yandex_compute_instance.k8s-cluster["k8s-node3"]: Still creating... [10s elapsed]
yandex_compute_instance.k8s-cluster["k8s-node2"]: Still creating... [10s elapsed]
yandex_compute_instance.k8s-cluster["k8s-node1"]: Still creating... [10s elapsed]
yandex_compute_instance.k8s-cluster["k8s-node2"]: Still creating... [20s elapsed]
yandex_compute_instance.k8s-cluster["k8s-node1"]: Still creating... [20s elapsed]
yandex_compute_instance.k8s-cluster["k8s-node3"]: Still creating... [20s elapsed]
yandex_compute_instance.k8s-cluster["k8s-node3"]: Still creating... [30s elapsed]
yandex_compute_instance.k8s-cluster["k8s-node2"]: Still creating... [30s elapsed]
yandex_compute_instance.k8s-cluster["k8s-node1"]: Still creating... [30s elapsed]
yandex_compute_instance.k8s-cluster["k8s-node1"]: Creation complete after 32s [id=fhmjtckrhsjhgfg71f9s]
yandex_compute_instance.k8s-cluster["k8s-node2"]: Creation complete after 32s [id=fhmrgq4pqimt5laee4cm]
yandex_compute_instance.k8s-cluster["k8s-node3"]: Creation complete after 36s [id=fhmu6oc2vdk5hj4dmgg6]

Apply complete! Resources: 7 added, 0 changed, 0 destroyed.

Outputs:

cluster_ips = {
  "external" = [
    "130.193.48.211",
    "130.193.37.164",
    "130.193.49.28",
  ]
  "internal" = [
    "192.168.10.12",
    "192.168.10.30",
    "192.168.10.26",
  ]
}

```

![img3](img/img3.png)

</details>

<details><summary>Получим ini файл с IP адресами</summary>

Сохраняем вывод в json

```bash
terraform output -json > ips.json
```

Преобразуем с помощью скрипта на питоне ips.json в cluster_ips.ini

<details><summary>out_to_ini.py</summary>

```python
import json
from configparser import ConfigParser

# загружаем данные из ips.json
with open('ips.json', 'r') as json_file:
    data = json.load(json_file)

# извлекаем IPs
external_ips = data['cluster_ips']['value']['external']
internal_ips = data['cluster_ips']['value']['internal']

# парсим в разделы
config = ConfigParser()

config['external'] = {ip: '' for ip in external_ips}
config['internal'] = {ip: '' for ip in internal_ips}

# убираем лишнее
with open('cluster_ips.ini', 'w') as ini_file:
    for section in config.sections():
        ini_file.write(f'[{section}]\n')
        for key in config[section]:
            ini_file.write(f'{key}\n')

print("Файл сконвертирован!")

```

</details>

```bash
denis@denis-lin(0):~/netology/diplom$ python3 out_to_ini.py
Файл сконвертирован!
denis@denis-lin(0):~/netology/diplom$ cat cluster_ips.ini
[external]
130.193.48.211
130.193.37.164
130.193.49.28
[internal]
192.168.10.12
192.168.10.30
192.168.10.26
```

</details>


Далее будем использовать плейбук из прошлых ДЗ - который писал сам, плейбук подготавливает хосты, устанавливая все необходимое. 


<details><summary>ansible k8s</summary>

[Репозиторий](https://github.com/VodyakovDenis/diplom-ansible)

---

<details><summary>ansible/ansible.cfg</summary>

```bash
[defaults]
host_key_checking = False
deprecation_warnings=False
```

</details>

<details><summary>ansible/ping.yml</summary>

```bash
---
- hosts: external
  gather_facts: false
  tasks:
    - ping:
```

* Позволит проверить доступность

</details>

<details><summary>ansible/k8s-cluster2.yml</summary>

```bash
---
- name: Prepare to install Kubernetes
  become: true
  hosts: external
  tasks:

    - name: Ensure the keyrings directory exists
      file:
        path: /etc/apt/keyrings
        state: directory

    - name: Create the k8s.gpg keyring file
      file:
        path: /etc/apt/keyrings/k8s.gpg
        state: touch

    - name: Add gpg key for K8S
      apt_key:
        url: https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key
        keyring: /etc/apt/keyrings/k8s.gpg


    - name: Add repository K8S
      apt_repository:
        repo: "deb [signed-by=/etc/apt/keyrings/k8s.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /"
        state: present
        filename: k8s
        mode: 0600

    - name: Load kernel modules
      modprobe:
        name: "{{ item }}"
        state: present
      loop:
        - br_netfilter

    - name: Set Sysctl on all nodes
      sysctl:
        name: "{{ item.name }}"
        value: "{{ item.value }}"
        state: present
      with_items:
        - name: net.ipv4.ip_forward
          value: 1
        - name: net.bridge.bridge-nf-call-iptables
          value: 1
        - name: net.bridge.bridge-nf-call-ip6tables
          value: 1

    - name: Prepare to install Kubernetes | Exec commands
      ansible.builtin.command: "{{ item }}"
      with_items:
        - modprobe overlay
        - modprobe br_netfilter
        - sysctl --system
        - swapoff -a
      register: result
      changed_when: result.rc != 0

    - name: Stop UFW service
      systemd:
        name: ufw
        state: stopped
        enabled: no

    - name: Mask firewalld service
      shell:
        cmd: systemctl mask firewalld

- name: Install Kubernetes
  become: true
  hosts: external
  tasks:
    - name: Install Kubernetes | Install packages
      ansible.builtin.apt:
        name:
          - kubectl
          - kubeadm
          - kubelet
          - containerd
        state: present
      notify: Start kubelet service

  handlers:
    - name: Start kubelet service
      ansible.builtin.service:
        name: kubelet
        enabled: true
        state: started
```

</details>

```bash
denis@denis-lin(0):~/netology/diplom$ mkdir ansible/
denis@denis-lin(0):~/netology/diplom$ mkdir ansible/inventory/
denis@denis-lin(0):~/netology/diplom$ ln -s ~/netology/diplom/cluster_ips.ini ~/netology/diplom/ansible/inventory/cluster_ips.ini
```

```bash
denis@denis-lin(0):~/netology/diplom$ ansible-playbook -i ansible/inventory/cluster_ips.ini ansible/k8s-cluster2.yml -u ubuntu
...
PLAY RECAP *****************************************************************************************************************************
130.193.37.164             : ok=16   changed=11   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
130.193.48.211             : ok=16   changed=11   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
130.193.49.28              : ok=16   changed=11   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

Далее для корректной работы k8s кластера необходимо поправить **/etc/hosts**

<details><summary>ansible/hosts.yml</summary>

```bash
---
- name: 'install k8s-cluster hostname'
  hosts: external
  become: true
  tasks:
    - name: "Add nodes /etc/hosts"
      lineinfile:
        name: /etc/hosts
        line: "{{ item }}"
        state: present
      with_items:
        - "192.168.10.12 k8s-node1"
        - "192.168.10.30 k8s-node2"
        - "192.168.10.26 k8s-node3"
```

</details>

```bash
denis@denis-lin(0):~/netology/diplom$ ansible-playbook -i ansible/inventory/cluster_ips.ini ansible/hosts.yml -u ubuntu
```

```bash
PLAY [install k8s-cluster hostname] ************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
ok: [130.193.49.28]
ok: [130.193.48.211]

TASK [Add nodes /etc/hosts] ********************************************************************************************************
changed: [130.193.37.164] => (item=192.168.10.12 k8s-node1)
changed: [130.193.49.28] => (item=192.168.10.12 k8s-node1)
changed: [130.193.48.211] => (item=192.168.10.12 k8s-node1)
changed: [130.193.37.164] => (item=192.168.10.30 k8s-node2)
changed: [130.193.49.28] => (item=192.168.10.30 k8s-node2)
changed: [130.193.48.211] => (item=192.168.10.30 k8s-node2)
changed: [130.193.37.164] => (item=192.168.10.26 k8s-node3)
changed: [130.193.49.28] => (item=192.168.10.26 k8s-node3)
changed: [130.193.48.211] => (item=192.168.10.26 k8s-node3)

PLAY RECAP *************************************************************************************************************************
130.193.37.164             : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
130.193.48.211             : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
130.193.49.28              : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

</details>


Инициализирует кластер Kubernetes на k8s-node1 (Данный процесс может быть полностью автоматизирован ansible но пока не хватило времени)

<details><summary>Собираем кластер</summary>

```bash
kubeadm init --v=5 --upload-certs --pod-network-cidr=10.244.0.0/16 --service-cidr=10.245.0.0/16 --control-plane-endpoint k8s-node1
...

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join k8s-node1:6443 --token lo4zxy.s3yyostmigxsvu83 \
        --discovery-token-ca-cert-hash sha256:dbad7f19b5066a9b40b16650614f6d196ff4e9e6fd1774c7b4c30bf7967609d1 \
        --control-plane --certificate-key 9a3c32b82b797d25ea88aa42976012850b54f4f29912b513b805b7b47b555fc2

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join k8s-node1:6443 --token lo4zxy.s3yyostmigxsvu83 \
        --discovery-token-ca-cert-hash sha256:dbad7f19b5066a9b40b16650614f6d196ff4e9e6fd1774c7b4c30bf7967609d1

```

```bash
root@k8s-node1:/home/ubuntu# mkdir -p $HOME/.kube
root@k8s-node1:/home/ubuntu# cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
root@k8s-node1:/home/ubuntu# chown $(id -u):$(id -g) $HOME/.kube/config
root@k8s-node1:/home/ubuntu# export KUBECONFIG=/etc/kubernetes/admin.conf
root@k8s-node1:/home/ubuntu# kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
namespace/kube-flannel created
serviceaccount/flannel created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created
```

Дальше мы можем добавить еще одну контрол ноду если необходимо, но в рамках данного задания нам будет достаточно одной контрол ноды, по этому оставшиеся ноды добавляем как воркеры.

```bash
denis@denis-lin(0):~/netology/diplom$ ssh ubuntu@130.193.37.164
...
ubuntu@k8s-node2:~$ sudo su
root@k8s-node2:/home/ubuntu# kubeadm join k8s-node1:6443 --token lo4zxy.s3yyostmigxsvu83 \
>         --discovery-token-ca-cert-hash sha256:dbad7f19b5066a9b40b16650614f6d196ff4e9e6fd1774c7b4c30bf7967609d1
...
This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

```

```bash
denis@denis-lin(0):~/netology/diplom$ ssh ubuntu@130.193.49.28
...
root@k8s-node3:/home/ubuntu# kubeadm join k8s-node1:6443 --token lo4zxy.s3yyostmigxsvu83 \
>         --discovery-token-ca-cert-hash sha256:dbad7f19b5066a9b40b16650614f6d196ff4e9e6fd1774c7b4c30bf7967609d1
...
This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

```

```bash
root@k8s-node1:/home/ubuntu# kubectl get nodes -owide
NAME        STATUS   ROLES           AGE     VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
k8s-node1   Ready    control-plane   13m     v1.28.6   192.168.10.12   <none>        Ubuntu 20.04.4 LTS   5.4.0-124-generic   containerd://1.7.2
k8s-node2   Ready    <none>          9m16s   v1.28.6   192.168.10.30   <none>        Ubuntu 20.04.4 LTS   5.4.0-124-generic   containerd://1.7.2
k8s-node3   Ready    <none>          5m29s   v1.28.6   192.168.10.26   <none>        Ubuntu 20.04.4 LTS   5.4.0-124-generic   containerd://1.7.2
```

Задаем роль воркеров:

```bash
root@k8s-node1:/home/ubuntu# kubectl label node k8s-node2 node-role.kubernetes.io/worker=worker
node/k8s-node2 labeled
root@k8s-node1:/home/ubuntu# kubectl label node k8s-node3 node-role.kubernetes.io/worker=worker
node/k8s-node3 labeled
root@k8s-node1:/home/ubuntu# kubectl get nodes -owide
NAME        STATUS   ROLES           AGE    VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
k8s-node1   Ready    control-plane   15m    v1.28.6   192.168.10.12   <none>        Ubuntu 20.04.4 LTS   5.4.0-124-generic   containerd://1.7.2
k8s-node2   Ready    worker          10m    v1.28.6   192.168.10.30   <none>        Ubuntu 20.04.4 LTS   5.4.0-124-generic   containerd://1.7.2
k8s-node3   Ready    worker          7m8s   v1.28.6   192.168.10.26   <none>        Ubuntu 20.04.4 LTS   5.4.0-124-generic   containerd://1.7.2

```


</details>

<details><summary>Итого</summary>

```bash
root@k8s-node1:/home/ubuntu# kubectl get pods --all-namespaces
NAMESPACE      NAME                                READY   STATUS    RESTARTS   AGE
kube-flannel   kube-flannel-ds-v59jh               1/1     Running   0          2m15s
kube-flannel   kube-flannel-ds-v7jnm               1/1     Running   0          82s
kube-flannel   kube-flannel-ds-v8jhz               1/1     Running   0          96s
kube-system    coredns-5dd5756b68-78xx5            1/1     Running   0          2m52s
kube-system    coredns-5dd5756b68-7nhcx            1/1     Running   0          2m52s
kube-system    etcd-k8s-node1                      1/1     Running   0          3m5s
kube-system    kube-apiserver-k8s-node1            1/1     Running   0          3m5s
kube-system    kube-controller-manager-k8s-node1   1/1     Running   0          3m5s
kube-system    kube-proxy-2l5c6                    1/1     Running   0          82s
kube-system    kube-proxy-nvh7g                    1/1     Running   0          2m52s
kube-system    kube-proxy-qklbt                    1/1     Running   0          96s
kube-system    kube-scheduler-k8s-node1            1/1     Running   0          3m5s

```

</details>

<details><summary>Переносим .kube/config на локальный пк для проверки</summary>

```bash
denis@denis-lin(0):~$ mkdir -p $HOME/.kube
denis@denis-lin(0):~$ nano $HOME/.kube/config
denis@denis-lin(0):~$ sudo nano /etc/hosts
[sudo] password for denis:
denis@denis-lin(0):~$ kubectl get pods --all-namespaces
NAMESPACE      NAME                                READY   STATUS    RESTARTS   AGE
kube-flannel   kube-flannel-ds-v59jh               1/1     Running   0          3m52s
kube-flannel   kube-flannel-ds-v7jnm               1/1     Running   0          2m59s
kube-flannel   kube-flannel-ds-v8jhz               1/1     Running   0          3m13s
kube-system    coredns-5dd5756b68-78xx5            1/1     Running   0          4m29s
kube-system    coredns-5dd5756b68-7nhcx            1/1     Running   0          4m29s
kube-system    etcd-k8s-node1                      1/1     Running   0          4m42s
kube-system    kube-apiserver-k8s-node1            1/1     Running   0          4m42s
kube-system    kube-controller-manager-k8s-node1   1/1     Running   0          4m42s
kube-system    kube-proxy-2l5c6                    1/1     Running   0          2m59s
kube-system    kube-proxy-nvh7g                    1/1     Running   0          4m29s
kube-system    kube-proxy-qklbt                    1/1     Running   0          3m13s
kube-system    kube-scheduler-k8s-node1            1/1     Running   0          4m42s


```

</details>


---
## Создание тестового приложения

> **Ожидаемый результат:**
> 1. Git репозиторий с тестовым приложением и Dockerfile.
> 2. Регистри с собранным docker image. В качестве регистри может быть DockerHub или Yandex Container Registry, созданный также с помощью terraform.


Репозиторий git [diplom-app](https://github.com/VodyakovDenis/diplom-app)   
Репозиторий docker hub [diplom-app](https://hub.docker.com/repository/docker/vodyakovdenis/diplom-app/general)   

<details><summary>docker build</summary>

```bash
denis@denis-lin(0):~/netology/diplom/diplom-app$ sudo docker build -t vodyakovdenis/diplom-app:v1.0 .
DEPRECATED: The legacy builder is deprecated and will be removed in a future release.
            Install the buildx component to build images with BuildKit:
            https://docs.docker.com/go/buildx/

Sending build context to Docker daemon  167.9kB
Step 1/2 : FROM nginx:1.22.0-alpine
1.22.0-alpine: Pulling from library/nginx
213ec9aee27d: Pull complete
1bfd2b69cf63: Pull complete
a19f4cc2e029: Pull complete
4ae981811a6d: Pull complete
7a662f439736: Pull complete
a317c3c2c906: Pull complete
Digest: sha256:addd3bf05ec3c69ef3e8f0021ce1ca98e0eb21117b97ab8b64127e3ff6e444ec
Status: Downloaded newer image for nginx:1.22.0-alpine
 ---> 5685937b6bc1
Step 2/2 : COPY site /usr/share/nginx/html
 ---> 19039b1859c8
Successfully built 19039b1859c8
Successfully tagged vodyakovdenis/diplom-app:v1.0
denis@denis-lin(0):~/netology/diplom/diplom-app$ sudo docker images | grep diplom
vodyakovdenis/diplom-app                        v1.0            19039b1859c8   46 seconds ago   23.6MB
denis@denis-lin(0):~/netology/diplom/diplom-app$ sudo docker push vodyakovdenis/diplom-app:v1.0
The push refers to repository [docker.io/vodyakovdenis/diplom-app]
16692ab9e5c6: Pushed
2215c66bbd37: Mounted from library/nginx
2fced351995b: Mounted from library/nginx
f5e6d03415cf: Mounted from library/nginx
3427d79f4b93: Mounted from library/nginx
e14491c4bad5: Mounted from library/nginx
994393dc58e7: Mounted from library/nginx
v1.0: digest: sha256:55a6cbdc2135301671ee74e089fb84c7714eaf1e87d9504d0594def30c26cce5 size: 1778
```

</details>

<details><summary>Результат</summary>

```bash
denis@denis-lin(0):~/netology/diplom/diplom-app$ sudo docker run -d -m 512m --name diplom-app -p 80:80 vodyakovdenis/diplom-app:v1.0
c6c9a973eaf4b7d93e1a168dad0124de90677eb905b0fdfdb948103259057045
denis@denis-lin(0):~/netology/diplom/diplom-app$ sudo docker ps -a
CONTAINER ID   IMAGE                           COMMAND                  CREATED         STATUS         PORTS     NAMES
c6c9a973eaf4   vodyakovdenis/diplom-app:v1.0   "/docker-entrypoint.…"   9 seconds ago   Up 7 seconds   80/tcp    diplomm-app
```
![img4](img/img4.png)
</details>


---
## Подготовка cистемы мониторинга и деплой приложения

> **Цель:**
> 1. Задеплоить в кластер prometheus, grafana, alertmanager, экспортер основных метрик Kubernetes.   
> 2. Задеплоить тестовое приложение, например, nginx сервер отдающий статическую страницу.   
>  
> **Ожидаемый результат:**
> 1. Git репозиторий с конфигурационными файлами для настройки Kubernetes.
> 2. Http доступ к web интерфейсу grafana.
> 3. Дашборды в grafana отображающие состояние Kubernetes кластера.
> 4. Http доступ к тестовому приложению.

<details><summary>install helm</summary>

* [оф дока](https://helm.sh/docs/intro/install/)

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 && \
chmod 700 get_helm.sh && \
./get_helm.sh
```

</details>

<details><summary>install ingress-controller</summary>

```bash
helm upgrade --install ingress-nginx ingress-nginx \
--repo https://kubernetes.github.io/ingress-nginx \
--namespace ingress-nginx --create-namespace
```

Иногда необходимо "[пропатчить](https://docs.nginx.com/nginx-ingress-controller/installation/installing-nic/installation-with-helm/)" 

```bash
helm pull oci://ghcr.io/nginxinc/charts/nginx-ingress --untar --version 1.1.2 && \
cd nginx-ingress && \
kubectl apply -f crds/
```

</details>

<details><summary>EXTERNAL-IP (на яндексе)</summary>

Арендуем IP у Яндекса

![img5](img/img5.png)

![img5_1](img/img5_1.png)

```bash
KUBE_EDITOR="nano" kubectl edit configmap -n kube-system kube-proxy
```

Установить значения:

```bash
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
ipvs:
  strictARP: true
```

```bash
kubectl apply -f externalIP.yaml
```
[externalIP.yaml](file/ingress-control/externalIP.yaml)

```bash
root@k8s-node1:/home/ubuntu# kubectl -n ingress-nginx get svc
NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.245.246.172   158.160.52.151   80:32278/TCP,443:31163/TCP   83m
ingress-nginx-controller-admission   ClusterIP      10.245.212.160   <none>           443/TCP                      83m

```


![img6](img/img6.png)

</details>

<details><summary>install prometheus + grafana</summary>

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
kubectl create namespace monitoring
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring -f values.yaml
```

[values.yaml](file/monitoring/values.yaml)


```bash
root@k8s-node1:/home/ubuntu/monitoring# kubectl -n monitoring get ingress
NAME                 CLASS   HOSTS                ADDRESS   PORTS   AGE
kube-state-ingress   nginx   grafana.diplom.com             80      10s
```

```
login: admin
password: prom-operator
```

![grafana1](img/grafana1.png)


![grafana2](img/grafana2.png)


![grafana3](img/grafana3.png)

<details><summary>* При ошибке : failed calling webhook</summary>

```bash
Error from server (InternalError): error when creating "ingress.yaml": Internal error occurred: failed calling webhook "validate.nginx.ingress.kubernetes.io": failed to call webhook: Post "https://ingress-nginx-controller-admission.ingress-nginx.svc:443/networking/v1/ingresses?timeout=10s": dial tcp 10.245.55.76:443: i/o timeout
```

Выполнить:

```bash
kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission
```

</details>

</details>

<details><summary>* Если вдруг проблема с DNS</summary>

накаждой ноде нужно:
```bash
systemctl mask firewald
```

затем самое просто грохнуть кор-днс
```bash
kubectl delete pods -n kube-system -l k8s-app=kube-dns
```

проверяем что новые поднялись:
```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
```

для тестов можем поднять dnsutils под
```bash
kubectl apply -f https://k8s.io/examples/admin/dns/dnsutils.yaml
```

и увидем:
```bash
root@dnsutils:/# ping ya.ru
PING ya.ru (5.255.255.242) 56(84) bytes of data.
64 bytes from ya.ru (5.255.255.242): icmp_seq=1 ttl=248 time=1.17 ms
64 bytes from ya.ru (5.255.255.242): icmp_seq=2 ttl=248 time=0.451 ms
```

</details>


<details><summary>* Используем metallb (если установка в локальной сети)</summary>

Для начала необходимо проверить что включен режим IPVS

```bash
KUBE_EDITOR="nano" kubectl edit configmap -n kube-system kube-proxy
```

Установить значения:

```bash
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
ipvs:
  strictARP: true
```

Устанавливаем metallb

```bash
kubectl create namespace metallb-system
helm repo add metallb https://metallb.github.io/metallb
helm install metallb metallb/metallb -n metallb-system
kubectl apply -f metallb-ip.yaml
```

<details><summary>metallb-ip.yaml</summary>

```bash
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default
  namespace: metallb-system
spec:
  addresses:
  - 10.10.3.100-10.10.3.105

kind: L2Advertisement
metadata:
  name: default
  namespace: metallb-system
spec:
  ipAddressPools:
  - default
  interfaces:
  - eth0
  - eth1
```

</details>

**Вывод на локальном кластере:**

```bash
root@k8s-node1:/home/admlocal# kubectl -n ingress-nginx get svc
NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.245.227.235   10.10.3.100   80:32360/TCP,443:32144/TCP   51m
ingress-nginx-controller-admission   ClusterIP      10.245.253.213   <none>        443/TCP                      51m
```

![img7](img/img7.png)

</details>


---
## Установка и настройка CI/CD

> **Цель:**
> 1. Автоматическая сборка docker образа при коммите в репозиторий с тестовым приложением.   
> 2. Автоматический деплой нового docker образа.   
> Можно использовать teamcity, jenkins, GitLab CI или GitHub Actions.   
>
> **Ожидаемый результат:**
> 1. Интерфейс ci/cd сервиса доступен по http.
> 2. При любом коммите в репозиторие с тестовым приложением происходит сборка и отправка в регистр Docker образа.
> 3. При создании тега (например, v1.0.0) происходит сборка и отправка с соответствующим label в регистри, а также деплой соответствующего Docker образа в кластер Kubernetes.

В качестве CI/CD будет использоваться гитлаб развернутый в одной из домашних работ (доступ может быть ограничен):

Репозиторий с ci/cd для gitlab (расположен на github) - [diplom-terraform](https://github.com/VodyakovDenis/diplom-terraform)

Ссылка на проект на внутреннет [gitlab](https://gitlab.it-git.ru/test/diplom-terraform)

<details><summary>Применение CI-CD-terraform pipeline</summary>

![tf1](img/tf1.png)

![tf6](img/tf6.png)

<details><summary>validate</summary>

[raw](https://gitlab.it-git.ru/test/diplom-terraform/-/jobs/328/raw)

```txt
[0KRunning with gitlab-runner 16.2.1 (674e0e29)[0;m
[0K  on gitlab-runner-1 GvwJgJtvC, system ID: s_3b0069d0acea[0;m
section_start:1708003106:prepare_executor
[0K[0K[36;1mPreparing the "docker" executor[0;m[0;m
[0KUsing Docker executor with image hashicorp/terraform:light ...[0;m
[0KPulling docker image hashicorp/terraform:light ...[0;m
[0KUsing docker image sha256:22ae929e925c6bdb60846c05fbee4f15ef15bd02d101e23f34dd6be69cace0f3 for hashicorp/terraform:light with digest hashicorp/terraform@sha256:2734886211a482abaff7efbbfd8bdb32295c9c35dda124092d46f39b69f47c42 ...[0;m
section_end:1708003113:prepare_executor
[0Ksection_start:1708003113:prepare_script
[0K[0K[36;1mPreparing environment[0;m[0;m
Running on runner-gvwjgjtvc-project-3-concurrent-0 via gitlab-runner-1...
section_end:1708003114:prepare_script
[0Ksection_start:1708003114:get_sources
[0K[0K[36;1mGetting source from Git repository[0;m[0;m
[32;1mFetching changes with git depth set to 20...[0;m
Reinitialized existing Git repository in /builds/test/diplom-terraform/.git/
[32;1mChecking out f7372221 as detached HEAD (ref is main)...[0;m
Removing .terraform.lock.hcl
Removing .terraform/

[32;1mSkipping Git submodules setup[0;m
section_end:1708003117:get_sources
[0Ksection_start:1708003117:step_script
[0K[0K[36;1mExecuting "step_script" stage of the job script[0;m[0;m
[0KUsing docker image sha256:22ae929e925c6bdb60846c05fbee4f15ef15bd02d101e23f34dd6be69cace0f3 for hashicorp/terraform:light with digest hashicorp/terraform@sha256:2734886211a482abaff7efbbfd8bdb32295c9c35dda124092d46f39b69f47c42 ...[0;m
[32;1m$ cp .terraformrc /root/.terraformrc[0;m
[32;1m$ rm -rf .terraform[0;m
[32;1m$ terraform --version[0;m
Terraform v1.4.2
on linux_amd64

Your version of Terraform is out of date! The latest version
is 1.7.3. You can update by downloading from https://www.terraform.io/downloads.html
[32;1m$ export GITLAB_ACCESS_TOKEN=$GITLAB_TOCKEN[0;m
[32;1m$ sed -i "s/__TOKEN__/$TOKEN/" variables.tf[0;m
[32;1m$ terraform init[0;m

[0m[1mInitializing the backend...[0m
[0m[32m
Successfully configured the backend "http"! Terraform will automatically
use this backend unless the backend configuration changes.[0m

[0m[1mInitializing provider plugins...[0m
- Finding latest version of yandex-cloud/yandex...
- Finding latest version of hashicorp/random...
- Installing yandex-cloud/yandex v0.108.0...
- Installed yandex-cloud/yandex v0.108.0 (unauthenticated)
- Installing hashicorp/random v3.6.0...
- Installed hashicorp/random v3.6.0 (unauthenticated)

Terraform has created a lock file [1m.terraform.lock.hcl[0m to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.[0m

[33m[33m╷[0m[0m
[33m│[0m [0m[1m[33mWarning: [0m[0m[1mIncomplete lock file information for providers[0m
[33m│[0m [0m
[33m│[0m [0m[0mDue to your customized provider installation methods, Terraform was forced
[33m│[0m [0mto calculate lock file checksums locally for the following providers:
[33m│[0m [0m  - hashicorp/random
[33m│[0m [0m  - yandex-cloud/yandex
[33m│[0m [0m
[33m│[0m [0mThe current .terraform.lock.hcl file only includes checksums for
[33m│[0m [0mlinux_amd64, so Terraform running on another platform will fail to install
[33m│[0m [0mthese providers.
[33m│[0m [0m
[33m│[0m [0mTo calculate additional checksums for another platform, run:
[33m│[0m [0m  terraform providers lock -platform=linux_amd64
[33m│[0m [0m(where linux_amd64 is the platform to generate)
[33m╵[0m[0m
[0m[0m
[0m[1m[32mTerraform has been successfully initialized![0m[32m[0m
[0m[32m
You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.[0m
[32;1m$ terraform validate[0;m
[32m[1mSuccess![0m The configuration is valid.
[0m
section_end:1708003130:step_script
[0Ksection_start:1708003130:cleanup_file_variables
[0K[0K[36;1mCleaning up project directory and file based variables[0;m[0;m
section_end:1708003131:cleanup_file_variables
[0K[32;1mJob succeeded[0;m

```

</details>

<details><summary>plan</summary>

[raw](https://gitlab.it-git.ru/test/diplom-terraform/-/jobs/329/raw)

```txt
[0KRunning with gitlab-runner 16.2.1 (674e0e29)[0;m
[0K  on gitlab-runner-1 GvwJgJtvC, system ID: s_3b0069d0acea[0;m
section_start:1708003135:prepare_executor
[0K[0K[36;1mPreparing the "docker" executor[0;m[0;m
[0KUsing Docker executor with image hashicorp/terraform:light ...[0;m
[0KPulling docker image hashicorp/terraform:light ...[0;m
[0KUsing docker image sha256:22ae929e925c6bdb60846c05fbee4f15ef15bd02d101e23f34dd6be69cace0f3 for hashicorp/terraform:light with digest hashicorp/terraform@sha256:2734886211a482abaff7efbbfd8bdb32295c9c35dda124092d46f39b69f47c42 ...[0;m
section_end:1708003141:prepare_executor
[0Ksection_start:1708003141:prepare_script
[0K[0K[36;1mPreparing environment[0;m[0;m
Running on runner-gvwjgjtvc-project-3-concurrent-0 via gitlab-runner-1...
section_end:1708003142:prepare_script
[0Ksection_start:1708003142:get_sources
[0K[0K[36;1mGetting source from Git repository[0;m[0;m
[32;1mFetching changes with git depth set to 20...[0;m
Reinitialized existing Git repository in /builds/test/diplom-terraform/.git/
[32;1mChecking out f7372221 as detached HEAD (ref is main)...[0;m
Removing .terraform.lock.hcl
Removing .terraform/

[32;1mSkipping Git submodules setup[0;m
section_end:1708003143:get_sources
[0Ksection_start:1708003143:step_script
[0K[0K[36;1mExecuting "step_script" stage of the job script[0;m[0;m
[0KUsing docker image sha256:22ae929e925c6bdb60846c05fbee4f15ef15bd02d101e23f34dd6be69cace0f3 for hashicorp/terraform:light with digest hashicorp/terraform@sha256:2734886211a482abaff7efbbfd8bdb32295c9c35dda124092d46f39b69f47c42 ...[0;m
[32;1m$ cp .terraformrc /root/.terraformrc[0;m
[32;1m$ rm -rf .terraform[0;m
[32;1m$ terraform --version[0;m
Terraform v1.4.2
on linux_amd64

Your version of Terraform is out of date! The latest version
is 1.7.3. You can update by downloading from https://www.terraform.io/downloads.html
[32;1m$ export GITLAB_ACCESS_TOKEN=$GITLAB_TOCKEN[0;m
[32;1m$ sed -i "s/__TOKEN__/$TOKEN/" variables.tf[0;m
[32;1m$ terraform init[0;m

[0m[1mInitializing the backend...[0m
[0m[32m
Successfully configured the backend "http"! Terraform will automatically
use this backend unless the backend configuration changes.[0m

[0m[1mInitializing provider plugins...[0m
- Finding latest version of yandex-cloud/yandex...
- Finding latest version of hashicorp/random...
- Installing yandex-cloud/yandex v0.108.0...
- Installed yandex-cloud/yandex v0.108.0 (unauthenticated)
- Installing hashicorp/random v3.6.0...
- Installed hashicorp/random v3.6.0 (unauthenticated)

Terraform has created a lock file [1m.terraform.lock.hcl[0m to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.[0m

[33m[33m╷[0m[0m
[33m│[0m [0m[1m[33mWarning: [0m[0m[1mIncomplete lock file information for providers[0m
[33m│[0m [0m
[33m│[0m [0m[0mDue to your customized provider installation methods, Terraform was forced
[33m│[0m [0mto calculate lock file checksums locally for the following providers:
[33m│[0m [0m  - hashicorp/random
[33m│[0m [0m  - yandex-cloud/yandex
[33m│[0m [0m
[33m│[0m [0mThe current .terraform.lock.hcl file only includes checksums for
[33m│[0m [0mlinux_amd64, so Terraform running on another platform will fail to install
[33m│[0m [0mthese providers.
[33m│[0m [0m
[33m│[0m [0mTo calculate additional checksums for another platform, run:
[33m│[0m [0m  terraform providers lock -platform=linux_amd64
[33m│[0m [0m(where linux_amd64 is the platform to generate)
[33m╵[0m[0m
[0m[0m
[0m[1m[32mTerraform has been successfully initialized![0m[32m[0m
[0m[32m
You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.[0m
[32;1m$ terraform plan -out "planfile"[0;m
Acquiring state lock. This may take a few moments...

Terraform used the selected providers to generate the following execution
plan. Resource actions are indicated with the following symbols:
  [32m+[0m create[0m

Terraform will perform the following actions:

[1m  # random_shuffle.diplom-network-subnet-random[0m will be created
[0m  [32m+[0m[0m resource "random_shuffle" "diplom-network-subnet-random" {
      [32m+[0m[0m id           = (known after apply)
      [32m+[0m[0m input        = [
          [32m+[0m[0m (known after apply),
          [32m+[0m[0m (known after apply),
        ]
      [32m+[0m[0m result       = (known after apply)
      [32m+[0m[0m result_count = 1
    }

[1m  # yandex_compute_instance.k8s-cluster["k8s-node1"][0m will be created
[0m  [32m+[0m[0m resource "yandex_compute_instance" "k8s-cluster" {
      [32m+[0m[0m created_at                = (known after apply)
      [32m+[0m[0m folder_id                 = (known after apply)
      [32m+[0m[0m fqdn                      = (known after apply)
      [32m+[0m[0m gpu_cluster_id            = (known after apply)
      [32m+[0m[0m hostname                  = "k8s-node1.diplom.local"
      [32m+[0m[0m id                        = (known after apply)
      [32m+[0m[0m maintenance_grace_period  = (known after apply)
      [32m+[0m[0m maintenance_policy        = (known after apply)
      [32m+[0m[0m metadata                  = {
          [32m+[0m[0m "serial-port-enable" = "1"
          [32m+[0m[0m "ssh-keys"           = <<-EOT
                ubuntu:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC7KoRsmTb6EC67YSxCrvoqTLhZcBgEd7lM4LebHU5p7qBtvyxA+oUxkH1Z4HjR2zbKEzDqtFfgREN5gaMFzHG/pa7RgMPR1c5lBynlsCzHDQqKt9417IoXfT71bFo2Mh+wXHo9gcJw9tU+4pab93zMBN7a40XtKWYQFsSQ0TffpprCcBkkqBeuBS9BFh3oaL7hRIOWPmUwpXmhY+Te7oaluj8fPxq/2ahPF1kthLY1NTLQvRGbrxENxsysi0jZwfDkYzWdWqhI5BA8fyG8e1wcpTM+9U8F4sAGMBxI5vEcC1J/Eb940tnEfp+oeZzcrSH9IRVCOgyGnEBh/qloxP7HVlErpX7KGf8h6s18h4d4mi5ylgBBgYfn/Wszej+Yotdgc+TJYihe/dDRulsxEcU/UQjIheqIoIIZApnajk0GdEyOtZpU/cY+CrtBH2czMLlNpn4lEOkcaCQu2XeqKn2yQNiOqz8jsDrVrTRrJHTEZ7+kkUROqmuNtcJdWl+4YH0= denis@denis-lin
            EOT
        }
      [32m+[0m[0m name                      = "k8s-node1"
      [32m+[0m[0m network_acceleration_type = "standard"
      [32m+[0m[0m platform_id               = "standard-v1"
      [32m+[0m[0m service_account_id        = (known after apply)
      [32m+[0m[0m status                    = (known after apply)
      [32m+[0m[0m zone                      = (known after apply)

      [32m+[0m[0m boot_disk {
          [32m+[0m[0m auto_delete = true
          [32m+[0m[0m device_name = (known after apply)
          [32m+[0m[0m disk_id     = (known after apply)
          [32m+[0m[0m mode        = (known after apply)

          [32m+[0m[0m initialize_params {
              [32m+[0m[0m block_size  = (known after apply)
              [32m+[0m[0m description = (known after apply)
              [32m+[0m[0m image_id    = "fd8tckeqoshi403tks4l"
              [32m+[0m[0m name        = (known after apply)
              [32m+[0m[0m size        = 20
              [32m+[0m[0m snapshot_id = (known after apply)
              [32m+[0m[0m type        = "network-hdd"
            }
        }

      [32m+[0m[0m network_interface {
          [32m+[0m[0m index              = (known after apply)
          [32m+[0m[0m ip_address         = (known after apply)
          [32m+[0m[0m ipv4               = true
          [32m+[0m[0m ipv6               = (known after apply)
          [32m+[0m[0m ipv6_address       = (known after apply)
          [32m+[0m[0m mac_address        = (known after apply)
          [32m+[0m[0m nat                = true
          [32m+[0m[0m nat_ip_address     = (known after apply)
          [32m+[0m[0m nat_ip_version     = (known after apply)
          [32m+[0m[0m security_group_ids = (known after apply)
          [32m+[0m[0m subnet_id          = (known after apply)
        }

      [32m+[0m[0m resources {
          [32m+[0m[0m core_fraction = 100
          [32m+[0m[0m cores         = 2
          [32m+[0m[0m memory        = 2
        }
    }

[1m  # yandex_compute_instance.k8s-cluster["k8s-node2"][0m will be created
[0m  [32m+[0m[0m resource "yandex_compute_instance" "k8s-cluster" {
      [32m+[0m[0m created_at                = (known after apply)
      [32m+[0m[0m folder_id                 = (known after apply)
      [32m+[0m[0m fqdn                      = (known after apply)
      [32m+[0m[0m gpu_cluster_id            = (known after apply)
      [32m+[0m[0m hostname                  = "k8s-node2.diplom.local"
      [32m+[0m[0m id                        = (known after apply)
      [32m+[0m[0m maintenance_grace_period  = (known after apply)
      [32m+[0m[0m maintenance_policy        = (known after apply)
      [32m+[0m[0m metadata                  = {
          [32m+[0m[0m "serial-port-enable" = "1"
          [32m+[0m[0m "ssh-keys"           = <<-EOT
                ubuntu:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC7KoRsmTb6EC67YSxCrvoqTLhZcBgEd7lM4LebHU5p7qBtvyxA+oUxkH1Z4HjR2zbKEzDqtFfgREN5gaMFzHG/pa7RgMPR1c5lBynlsCzHDQqKt9417IoXfT71bFo2Mh+wXHo9gcJw9tU+4pab93zMBN7a40XtKWYQFsSQ0TffpprCcBkkqBeuBS9BFh3oaL7hRIOWPmUwpXmhY+Te7oaluj8fPxq/2ahPF1kthLY1NTLQvRGbrxENxsysi0jZwfDkYzWdWqhI5BA8fyG8e1wcpTM+9U8F4sAGMBxI5vEcC1J/Eb940tnEfp+oeZzcrSH9IRVCOgyGnEBh/qloxP7HVlErpX7KGf8h6s18h4d4mi5ylgBBgYfn/Wszej+Yotdgc+TJYihe/dDRulsxEcU/UQjIheqIoIIZApnajk0GdEyOtZpU/cY+CrtBH2czMLlNpn4lEOkcaCQu2XeqKn2yQNiOqz8jsDrVrTRrJHTEZ7+kkUROqmuNtcJdWl+4YH0= denis@denis-lin
            EOT
        }
      [32m+[0m[0m name                      = "k8s-node2"
      [32m+[0m[0m network_acceleration_type = "standard"
      [32m+[0m[0m platform_id               = "standard-v1"
      [32m+[0m[0m service_account_id        = (known after apply)
      [32m+[0m[0m status                    = (known after apply)
      [32m+[0m[0m zone                      = (known after apply)

      [32m+[0m[0m boot_disk {
          [32m+[0m[0m auto_delete = true
          [32m+[0m[0m device_name = (known after apply)
          [32m+[0m[0m disk_id     = (known after apply)
          [32m+[0m[0m mode        = (known after apply)

          [32m+[0m[0m initialize_params {
              [32m+[0m[0m block_size  = (known after apply)
              [32m+[0m[0m description = (known after apply)
              [32m+[0m[0m image_id    = "fd8tckeqoshi403tks4l"
              [32m+[0m[0m name        = (known after apply)
              [32m+[0m[0m size        = 20
              [32m+[0m[0m snapshot_id = (known after apply)
              [32m+[0m[0m type        = "network-hdd"
            }
        }

      [32m+[0m[0m network_interface {
          [32m+[0m[0m index              = (known after apply)
          [32m+[0m[0m ip_address         = (known after apply)
          [32m+[0m[0m ipv4               = true
          [32m+[0m[0m ipv6               = (known after apply)
          [32m+[0m[0m ipv6_address       = (known after apply)
          [32m+[0m[0m mac_address        = (known after apply)
          [32m+[0m[0m nat                = true
          [32m+[0m[0m nat_ip_address     = (known after apply)
          [32m+[0m[0m nat_ip_version     = (known after apply)
          [32m+[0m[0m security_group_ids = (known after apply)
          [32m+[0m[0m subnet_id          = (known after apply)
        }

      [32m+[0m[0m resources {
          [32m+[0m[0m core_fraction = 100
          [32m+[0m[0m cores         = 2
          [32m+[0m[0m memory        = 2
        }
    }

[1m  # yandex_compute_instance.k8s-cluster["k8s-node3"][0m will be created
[0m  [32m+[0m[0m resource "yandex_compute_instance" "k8s-cluster" {
      [32m+[0m[0m created_at                = (known after apply)
      [32m+[0m[0m folder_id                 = (known after apply)
      [32m+[0m[0m fqdn                      = (known after apply)
      [32m+[0m[0m gpu_cluster_id            = (known after apply)
      [32m+[0m[0m hostname                  = "k8s-node3.diplom.local"
      [32m+[0m[0m id                        = (known after apply)
      [32m+[0m[0m maintenance_grace_period  = (known after apply)
      [32m+[0m[0m maintenance_policy        = (known after apply)
      [32m+[0m[0m metadata                  = {
          [32m+[0m[0m "serial-port-enable" = "1"
          [32m+[0m[0m "ssh-keys"           = <<-EOT
                ubuntu:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC7KoRsmTb6EC67YSxCrvoqTLhZcBgEd7lM4LebHU5p7qBtvyxA+oUxkH1Z4HjR2zbKEzDqtFfgREN5gaMFzHG/pa7RgMPR1c5lBynlsCzHDQqKt9417IoXfT71bFo2Mh+wXHo9gcJw9tU+4pab93zMBN7a40XtKWYQFsSQ0TffpprCcBkkqBeuBS9BFh3oaL7hRIOWPmUwpXmhY+Te7oaluj8fPxq/2ahPF1kthLY1NTLQvRGbrxENxsysi0jZwfDkYzWdWqhI5BA8fyG8e1wcpTM+9U8F4sAGMBxI5vEcC1J/Eb940tnEfp+oeZzcrSH9IRVCOgyGnEBh/qloxP7HVlErpX7KGf8h6s18h4d4mi5ylgBBgYfn/Wszej+Yotdgc+TJYihe/dDRulsxEcU/UQjIheqIoIIZApnajk0GdEyOtZpU/cY+CrtBH2czMLlNpn4lEOkcaCQu2XeqKn2yQNiOqz8jsDrVrTRrJHTEZ7+kkUROqmuNtcJdWl+4YH0= denis@denis-lin
            EOT
        }
      [32m+[0m[0m name                      = "k8s-node3"
      [32m+[0m[0m network_acceleration_type = "standard"
      [32m+[0m[0m platform_id               = "standard-v1"
      [32m+[0m[0m service_account_id        = (known after apply)
      [32m+[0m[0m status                    = (known after apply)
      [32m+[0m[0m zone                      = (known after apply)

      [32m+[0m[0m boot_disk {
          [32m+[0m[0m auto_delete = true
          [32m+[0m[0m device_name = (known after apply)
          [32m+[0m[0m disk_id     = (known after apply)
          [32m+[0m[0m mode        = (known after apply)

          [32m+[0m[0m initialize_params {
              [32m+[0m[0m block_size  = (known after apply)
              [32m+[0m[0m description = (known after apply)
              [32m+[0m[0m image_id    = "fd8tckeqoshi403tks4l"
              [32m+[0m[0m name        = (known after apply)
              [32m+[0m[0m size        = 20
              [32m+[0m[0m snapshot_id = (known after apply)
              [32m+[0m[0m type        = "network-hdd"
            }
        }

      [32m+[0m[0m network_interface {
          [32m+[0m[0m index              = (known after apply)
          [32m+[0m[0m ip_address         = (known after apply)
          [32m+[0m[0m ipv4               = true
          [32m+[0m[0m ipv6               = (known after apply)
          [32m+[0m[0m ipv6_address       = (known after apply)
          [32m+[0m[0m mac_address        = (known after apply)
          [32m+[0m[0m nat                = true
          [32m+[0m[0m nat_ip_address     = (known after apply)
          [32m+[0m[0m nat_ip_version     = (known after apply)
          [32m+[0m[0m security_group_ids = (known after apply)
          [32m+[0m[0m subnet_id          = (known after apply)
        }

      [32m+[0m[0m resources {
          [32m+[0m[0m core_fraction = 100
          [32m+[0m[0m cores         = 2
          [32m+[0m[0m memory        = 2
        }
    }

[1m  # yandex_vpc_network.diplom-network[0m will be created
[0m  [32m+[0m[0m resource "yandex_vpc_network" "diplom-network" {
      [32m+[0m[0m created_at                = (known after apply)
      [32m+[0m[0m default_security_group_id = (known after apply)
      [32m+[0m[0m folder_id                 = (known after apply)
      [32m+[0m[0m id                        = (known after apply)
      [32m+[0m[0m labels                    = (known after apply)
      [32m+[0m[0m name                      = "diplom-network"
      [32m+[0m[0m subnet_ids                = (known after apply)
    }

[1m  # yandex_vpc_subnet.diplom-network-subnet-a[0m will be created
[0m  [32m+[0m[0m resource "yandex_vpc_subnet" "diplom-network-subnet-a" {
      [32m+[0m[0m created_at     = (known after apply)
      [32m+[0m[0m folder_id      = (known after apply)
      [32m+[0m[0m id             = (known after apply)
      [32m+[0m[0m labels         = (known after apply)
      [32m+[0m[0m name           = "diplom-network-subnet-a"
      [32m+[0m[0m network_id     = (known after apply)
      [32m+[0m[0m v4_cidr_blocks = [
          [32m+[0m[0m "192.168.10.0/24",
        ]
      [32m+[0m[0m v6_cidr_blocks = (known after apply)
      [32m+[0m[0m zone           = "ru-central1-a"
    }

[1m  # yandex_vpc_subnet.diplom-network-subnet-b[0m will be created
[0m  [32m+[0m[0m resource "yandex_vpc_subnet" "diplom-network-subnet-b" {
      [32m+[0m[0m created_at     = (known after apply)
      [32m+[0m[0m folder_id      = (known after apply)
      [32m+[0m[0m id             = (known after apply)
      [32m+[0m[0m labels         = (known after apply)
      [32m+[0m[0m name           = "diplom-network-subnet-b"
      [32m+[0m[0m network_id     = (known after apply)
      [32m+[0m[0m v4_cidr_blocks = [
          [32m+[0m[0m "192.168.11.0/24",
        ]
      [32m+[0m[0m v6_cidr_blocks = (known after apply)
      [32m+[0m[0m zone           = "ru-central1-b"
    }

[1mPlan:[0m 7 to add, 0 to change, 0 to destroy.
[0m
Changes to Outputs:
  [32m+[0m[0m cluster_ips = {
      [32m+[0m[0m external = [
          [32m+[0m[0m [90mnull[0m[0m,
          [32m+[0m[0m [90mnull[0m[0m,
          [32m+[0m[0m [90mnull[0m[0m,
        ]
      [32m+[0m[0m internal = [
          [32m+[0m[0m [90mnull[0m[0m,
          [32m+[0m[0m [90mnull[0m[0m,
          [32m+[0m[0m [90mnull[0m[0m,
        ]
    }
[90m
─────────────────────────────────────────────────────────────────────────────[0m

Saved the plan to: planfile

To perform exactly these actions, run the following command to apply:
    terraform apply "planfile"
section_end:1708003154:step_script
[0Ksection_start:1708003154:upload_artifacts_on_success
[0K[0K[36;1mUploading artifacts for successful job[0;m[0;m
[32;1mUploading artifacts...[0;m
planfile: found 1 matching artifact files and directories[0;m 
Uploading artifacts as "archive" to coordinator... 201 Created[0;m  id[0;m=329 responseStatus[0;m=201 Created token[0;m=64_zXbTZ
section_end:1708003158:upload_artifacts_on_success
[0Ksection_start:1708003158:cleanup_file_variables
[0K[0K[36;1mCleaning up project directory and file based variables[0;m[0;m
section_end:1708003159:cleanup_file_variables
[0K[32;1mJob succeeded[0;m
```

</details>

<details><summary>apply</summary>

[raw](https://gitlab.it-git.ru/test/diplom-terraform/-/jobs/330/raw)

```txt
[0KRunning with gitlab-runner 16.2.1 (674e0e29)[0;m
[0K  on gitlab-runner-1 GvwJgJtvC, system ID: s_3b0069d0acea[0;m
section_start:1708003675:prepare_executor
[0K[0K[36;1mPreparing the "docker" executor[0;m[0;m
[0KUsing Docker executor with image hashicorp/terraform:light ...[0;m
[0KPulling docker image hashicorp/terraform:light ...[0;m
[0KUsing docker image sha256:22ae929e925c6bdb60846c05fbee4f15ef15bd02d101e23f34dd6be69cace0f3 for hashicorp/terraform:light with digest hashicorp/terraform@sha256:2734886211a482abaff7efbbfd8bdb32295c9c35dda124092d46f39b69f47c42 ...[0;m
section_end:1708003680:prepare_executor
[0Ksection_start:1708003680:prepare_script
[0K[0K[36;1mPreparing environment[0;m[0;m
Running on runner-gvwjgjtvc-project-3-concurrent-0 via gitlab-runner-1...
section_end:1708003681:prepare_script
[0Ksection_start:1708003681:get_sources
[0K[0K[36;1mGetting source from Git repository[0;m[0;m
[32;1mFetching changes with git depth set to 20...[0;m
Reinitialized existing Git repository in /builds/test/diplom-terraform/.git/
[32;1mChecking out f7372221 as detached HEAD (ref is main)...[0;m
Removing .terraform.lock.hcl
Removing .terraform/
Removing planfile

[32;1mSkipping Git submodules setup[0;m
section_end:1708003683:get_sources
[0Ksection_start:1708003683:download_artifacts
[0K[0K[36;1mDownloading artifacts[0;m[0;m
[32;1mDownloading artifacts for plan (329)...[0;m
Downloading artifacts from coordinator... ok      [0;m  host[0;m=gitlab.it-git.ru id[0;m=329 responseStatus[0;m=200 OK token[0;m=64_iQVx-
section_end:1708003684:download_artifacts
[0Ksection_start:1708003684:step_script
[0K[0K[36;1mExecuting "step_script" stage of the job script[0;m[0;m
[0KUsing docker image sha256:22ae929e925c6bdb60846c05fbee4f15ef15bd02d101e23f34dd6be69cace0f3 for hashicorp/terraform:light with digest hashicorp/terraform@sha256:2734886211a482abaff7efbbfd8bdb32295c9c35dda124092d46f39b69f47c42 ...[0;m
[32;1m$ cp .terraformrc /root/.terraformrc[0;m
[32;1m$ rm -rf .terraform[0;m
[32;1m$ terraform --version[0;m
Terraform v1.4.2
on linux_amd64

Your version of Terraform is out of date! The latest version
is 1.7.3. You can update by downloading from https://www.terraform.io/downloads.html
[32;1m$ export GITLAB_ACCESS_TOKEN=$GITLAB_TOCKEN[0;m
[32;1m$ sed -i "s/__TOKEN__/$TOKEN/" variables.tf[0;m
[32;1m$ terraform init[0;m

[0m[1mInitializing the backend...[0m
[0m[32m
Successfully configured the backend "http"! Terraform will automatically
use this backend unless the backend configuration changes.[0m

[0m[1mInitializing provider plugins...[0m
- Finding latest version of yandex-cloud/yandex...
- Finding latest version of hashicorp/random...
- Installing yandex-cloud/yandex v0.108.0...
- Installed yandex-cloud/yandex v0.108.0 (unauthenticated)
- Installing hashicorp/random v3.6.0...
- Installed hashicorp/random v3.6.0 (unauthenticated)

Terraform has created a lock file [1m.terraform.lock.hcl[0m to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.[0m

[33m[33m╷[0m[0m
[33m│[0m [0m[1m[33mWarning: [0m[0m[1mIncomplete lock file information for providers[0m
[33m│[0m [0m
[33m│[0m [0m[0mDue to your customized provider installation methods, Terraform was forced
[33m│[0m [0mto calculate lock file checksums locally for the following providers:
[33m│[0m [0m  - hashicorp/random
[33m│[0m [0m  - yandex-cloud/yandex
[33m│[0m [0m
[33m│[0m [0mThe current .terraform.lock.hcl file only includes checksums for
[33m│[0m [0mlinux_amd64, so Terraform running on another platform will fail to install
[33m│[0m [0mthese providers.
[33m│[0m [0m
[33m│[0m [0mTo calculate additional checksums for another platform, run:
[33m│[0m [0m  terraform providers lock -platform=linux_amd64
[33m│[0m [0m(where linux_amd64 is the platform to generate)
[33m╵[0m[0m
[0m[0m
[0m[1m[32mTerraform has been successfully initialized![0m[32m[0m
[0m[32m
You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.[0m
[32;1m$ terraform apply -auto-approve "planfile"[0;m
[0m[1myandex_vpc_network.diplom-network: Creating...[0m[0m
[0m[1myandex_vpc_network.diplom-network: Creation complete after 4s [id=enp48huj8ab5ubime77m][0m
[0m[1myandex_vpc_subnet.diplom-network-subnet-a: Creating...[0m[0m
[0m[1myandex_vpc_subnet.diplom-network-subnet-b: Creating...[0m[0m
[0m[1myandex_vpc_subnet.diplom-network-subnet-b: Creation complete after 1s [id=e2lbtdq0id3k2avv45sm][0m
[0m[1myandex_vpc_subnet.diplom-network-subnet-a: Creation complete after 1s [id=e9bp6i4phmsbasju46hb][0m
[0m[1mrandom_shuffle.diplom-network-subnet-random: Creating...[0m[0m
[0m[1mrandom_shuffle.diplom-network-subnet-random: Creation complete after 0s [id=-][0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node2"]: Creating...[0m[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node1"]: Creating...[0m[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node3"]: Creating...[0m[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node1"]: Still creating... [10s elapsed][0m[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node2"]: Still creating... [10s elapsed][0m[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node3"]: Still creating... [10s elapsed][0m[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node2"]: Still creating... [20s elapsed][0m[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node1"]: Still creating... [20s elapsed][0m[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node3"]: Still creating... [20s elapsed][0m[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node2"]: Still creating... [30s elapsed][0m[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node1"]: Still creating... [30s elapsed][0m[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node3"]: Still creating... [30s elapsed][0m[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node2"]: Creation complete after 36s [id=fhm10404qco16b0d3j77][0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node3"]: Creation complete after 36s [id=fhmnne713httg57trv06][0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node1"]: Still creating... [40s elapsed][0m[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node1"]: Creation complete after 45s [id=fhm5htkn04et2l8khpok][0m
[0m[1m[32m
Apply complete! Resources: 7 added, 0 changed, 0 destroyed.
[0m[0m[1m[32m
Outputs:

[0mcluster_ips = {
  "external" = [
    "158.160.116.25",
    "158.160.40.26",
    "178.154.207.84",
  ]
  "internal" = [
    "192.168.10.4",
    "192.168.10.10",
    "192.168.10.13",
  ]
}
section_end:1708003744:step_script
[0Ksection_start:1708003744:cleanup_file_variables
[0K[0K[36;1mCleaning up project directory and file based variables[0;m[0;m
section_end:1708003745:cleanup_file_variables
[0K[32;1mJob succeeded[0;m
```

![tf2](img/tf2.png)

![tf3](img/tf3.png)

</details>

<details><summary>destroy</summary>

[raw](https://gitlab.it-git.ru/test/diplom-terraform/-/jobs/331/raw)

```txt
[0KRunning with gitlab-runner 16.2.1 (674e0e29)[0;m
[0K  on gitlab-runner-1 GvwJgJtvC, system ID: s_3b0069d0acea[0;m
section_start:1708004058:prepare_executor
[0K[0K[36;1mPreparing the "docker" executor[0;m[0;m
[0KUsing Docker executor with image hashicorp/terraform:light ...[0;m
[0KPulling docker image hashicorp/terraform:light ...[0;m
[0KUsing docker image sha256:22ae929e925c6bdb60846c05fbee4f15ef15bd02d101e23f34dd6be69cace0f3 for hashicorp/terraform:light with digest hashicorp/terraform@sha256:2734886211a482abaff7efbbfd8bdb32295c9c35dda124092d46f39b69f47c42 ...[0;m
section_end:1708004063:prepare_executor
[0Ksection_start:1708004063:prepare_script
[0K[0K[36;1mPreparing environment[0;m[0;m
Running on runner-gvwjgjtvc-project-3-concurrent-0 via gitlab-runner-1...
section_end:1708004063:prepare_script
[0Ksection_start:1708004063:get_sources
[0K[0K[36;1mGetting source from Git repository[0;m[0;m
[32;1mFetching changes with git depth set to 20...[0;m
Reinitialized existing Git repository in /builds/test/diplom-terraform/.git/
[32;1mChecking out f7372221 as detached HEAD (ref is main)...[0;m
Removing .terraform.lock.hcl
Removing .terraform/
Removing planfile

[32;1mSkipping Git submodules setup[0;m
section_end:1708004064:get_sources
[0Ksection_start:1708004064:step_script
[0K[0K[36;1mExecuting "step_script" stage of the job script[0;m[0;m
[0KUsing docker image sha256:22ae929e925c6bdb60846c05fbee4f15ef15bd02d101e23f34dd6be69cace0f3 for hashicorp/terraform:light with digest hashicorp/terraform@sha256:2734886211a482abaff7efbbfd8bdb32295c9c35dda124092d46f39b69f47c42 ...[0;m
[32;1m$ cp .terraformrc /root/.terraformrc[0;m
[32;1m$ rm -rf .terraform[0;m
[32;1m$ terraform --version[0;m
Terraform v1.4.2
on linux_amd64

Your version of Terraform is out of date! The latest version
is 1.7.3. You can update by downloading from https://www.terraform.io/downloads.html
[32;1m$ export GITLAB_ACCESS_TOKEN=$GITLAB_TOCKEN[0;m
[32;1m$ sed -i "s/__TOKEN__/$TOKEN/" variables.tf[0;m
[32;1m$ terraform init[0;m

[0m[1mInitializing the backend...[0m
[0m[32m
Successfully configured the backend "http"! Terraform will automatically
use this backend unless the backend configuration changes.[0m

[0m[1mInitializing provider plugins...[0m
- Finding latest version of yandex-cloud/yandex...
- Finding latest version of hashicorp/random...
- Installing yandex-cloud/yandex v0.108.0...
- Installed yandex-cloud/yandex v0.108.0 (unauthenticated)
- Installing hashicorp/random v3.6.0...
- Installed hashicorp/random v3.6.0 (unauthenticated)

Terraform has created a lock file [1m.terraform.lock.hcl[0m to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.[0m

[33m[33m╷[0m[0m
[33m│[0m [0m[1m[33mWarning: [0m[0m[1mIncomplete lock file information for providers[0m
[33m│[0m [0m
[33m│[0m [0m[0mDue to your customized provider installation methods, Terraform was forced
[33m│[0m [0mto calculate lock file checksums locally for the following providers:
[33m│[0m [0m  - hashicorp/random
[33m│[0m [0m  - yandex-cloud/yandex
[33m│[0m [0m
[33m│[0m [0mThe current .terraform.lock.hcl file only includes checksums for
[33m│[0m [0mlinux_amd64, so Terraform running on another platform will fail to install
[33m│[0m [0mthese providers.
[33m│[0m [0m
[33m│[0m [0mTo calculate additional checksums for another platform, run:
[33m│[0m [0m  terraform providers lock -platform=linux_amd64
[33m│[0m [0m(where linux_amd64 is the platform to generate)
[33m╵[0m[0m
[0m[0m
[0m[1m[32mTerraform has been successfully initialized![0m[32m[0m
[0m[32m
You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.[0m
[32;1m$ terraform init[0;m

[0m[1mInitializing the backend...[0m

[0m[1mInitializing provider plugins...[0m
- Reusing previous version of yandex-cloud/yandex from the dependency lock file
- Reusing previous version of hashicorp/random from the dependency lock file
- Using previously-installed hashicorp/random v3.6.0
- Using previously-installed yandex-cloud/yandex v0.108.0

[0m[1m[32mTerraform has been successfully initialized![0m[32m[0m
[0m[32m
You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.[0m
[32;1m$ terraform destroy -auto-approve[0;m
[0m[1myandex_vpc_network.diplom-network: Refreshing state... [id=enp48huj8ab5ubime77m][0m
[0m[1myandex_vpc_subnet.diplom-network-subnet-a: Refreshing state... [id=e9bp6i4phmsbasju46hb][0m
[0m[1myandex_vpc_subnet.diplom-network-subnet-b: Refreshing state... [id=e2lbtdq0id3k2avv45sm][0m
[0m[1mrandom_shuffle.diplom-network-subnet-random: Refreshing state... [id=-][0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node1"]: Refreshing state... [id=fhm5htkn04et2l8khpok][0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node2"]: Refreshing state... [id=fhm10404qco16b0d3j77][0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node3"]: Refreshing state... [id=fhmnne713httg57trv06][0m

Terraform used the selected providers to generate the following execution
plan. Resource actions are indicated with the following symbols:
  [31m-[0m destroy[0m

Terraform will perform the following actions:

[1m  # random_shuffle.diplom-network-subnet-random[0m will be [1m[31mdestroyed[0m
[0m  [31m-[0m[0m resource "random_shuffle" "diplom-network-subnet-random" {
      [31m-[0m[0m id           = "-" [90m-> null[0m[0m
      [31m-[0m[0m input        = [
          [31m-[0m[0m "e9bp6i4phmsbasju46hb",
          [31m-[0m[0m "e9bp6i4phmsbasju46hb",
        ] [90m-> null[0m[0m
      [31m-[0m[0m result       = [
          [31m-[0m[0m "e9bp6i4phmsbasju46hb",
        ] [90m-> null[0m[0m
      [31m-[0m[0m result_count = 1 [90m-> null[0m[0m
    }

[1m  # yandex_compute_instance.k8s-cluster["k8s-node1"][0m will be [1m[31mdestroyed[0m
[0m  [31m-[0m[0m resource "yandex_compute_instance" "k8s-cluster" {
      [31m-[0m[0m created_at                = "2024-02-15T13:28:20Z" [90m-> null[0m[0m
      [31m-[0m[0m folder_id                 = "b1gfkh66dgk5tpn7ajko" [90m-> null[0m[0m
      [31m-[0m[0m fqdn                      = "k8s-node1.diplom.local" [90m-> null[0m[0m
      [31m-[0m[0m hostname                  = "k8s-node1.diplom.local" [90m-> null[0m[0m
      [31m-[0m[0m id                        = "fhm5htkn04et2l8khpok" [90m-> null[0m[0m
      [31m-[0m[0m labels                    = {} [90m-> null[0m[0m
      [31m-[0m[0m metadata                  = {
          [31m-[0m[0m "serial-port-enable" = "1"
          [31m-[0m[0m "ssh-keys"           = <<-EOT
                ubuntu:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC7KoRsmTb6EC67YSxCrvoqTLhZcBgEd7lM4LebHU5p7qBtvyxA+oUxkH1Z4HjR2zbKEzDqtFfgREN5gaMFzHG/pa7RgMPR1c5lBynlsCzHDQqKt9417IoXfT71bFo2Mh+wXHo9gcJw9tU+4pab93zMBN7a40XtKWYQFsSQ0TffpprCcBkkqBeuBS9BFh3oaL7hRIOWPmUwpXmhY+Te7oaluj8fPxq/2ahPF1kthLY1NTLQvRGbrxENxsysi0jZwfDkYzWdWqhI5BA8fyG8e1wcpTM+9U8F4sAGMBxI5vEcC1J/Eb940tnEfp+oeZzcrSH9IRVCOgyGnEBh/qloxP7HVlErpX7KGf8h6s18h4d4mi5ylgBBgYfn/Wszej+Yotdgc+TJYihe/dDRulsxEcU/UQjIheqIoIIZApnajk0GdEyOtZpU/cY+CrtBH2czMLlNpn4lEOkcaCQu2XeqKn2yQNiOqz8jsDrVrTRrJHTEZ7+kkUROqmuNtcJdWl+4YH0= denis@denis-lin
            EOT
        } [90m-> null[0m[0m
      [31m-[0m[0m name                      = "k8s-node1" [90m-> null[0m[0m
      [31m-[0m[0m network_acceleration_type = "standard" [90m-> null[0m[0m
      [31m-[0m[0m platform_id               = "standard-v1" [90m-> null[0m[0m
      [31m-[0m[0m status                    = "running" [90m-> null[0m[0m
      [31m-[0m[0m zone                      = "ru-central1-a" [90m-> null[0m[0m

      [31m-[0m[0m boot_disk {
          [31m-[0m[0m auto_delete = true [90m-> null[0m[0m
          [31m-[0m[0m device_name = "fhm7d55ina368mmjs97c" [90m-> null[0m[0m
          [31m-[0m[0m disk_id     = "fhm7d55ina368mmjs97c" [90m-> null[0m[0m
          [31m-[0m[0m mode        = "READ_WRITE" [90m-> null[0m[0m

          [31m-[0m[0m initialize_params {
              [31m-[0m[0m block_size = 4096 [90m-> null[0m[0m
              [31m-[0m[0m image_id   = "fd8tckeqoshi403tks4l" [90m-> null[0m[0m
              [31m-[0m[0m size       = 20 [90m-> null[0m[0m
              [31m-[0m[0m type       = "network-hdd" [90m-> null[0m[0m
            }
        }

      [31m-[0m[0m metadata_options {
          [31m-[0m[0m aws_v1_http_endpoint = 1 [90m-> null[0m[0m
          [31m-[0m[0m aws_v1_http_token    = 2 [90m-> null[0m[0m
          [31m-[0m[0m gce_http_endpoint    = 1 [90m-> null[0m[0m
          [31m-[0m[0m gce_http_token       = 1 [90m-> null[0m[0m
        }

      [31m-[0m[0m network_interface {
          [31m-[0m[0m index              = 0 [90m-> null[0m[0m
          [31m-[0m[0m ip_address         = "192.168.10.4" [90m-> null[0m[0m
          [31m-[0m[0m ipv4               = true [90m-> null[0m[0m
          [31m-[0m[0m ipv6               = false [90m-> null[0m[0m
          [31m-[0m[0m mac_address        = "d0:0d:58:f6:97:01" [90m-> null[0m[0m
          [31m-[0m[0m nat                = true [90m-> null[0m[0m
          [31m-[0m[0m nat_ip_address     = "158.160.116.25" [90m-> null[0m[0m
          [31m-[0m[0m nat_ip_version     = "IPV4" [90m-> null[0m[0m
          [31m-[0m[0m security_group_ids = [] [90m-> null[0m[0m
          [31m-[0m[0m subnet_id          = "e9bp6i4phmsbasju46hb" [90m-> null[0m[0m
        }

      [31m-[0m[0m placement_policy {
          [31m-[0m[0m host_affinity_rules       = [] [90m-> null[0m[0m
          [31m-[0m[0m placement_group_partition = 0 [90m-> null[0m[0m
        }

      [31m-[0m[0m resources {
          [31m-[0m[0m core_fraction = 100 [90m-> null[0m[0m
          [31m-[0m[0m cores         = 2 [90m-> null[0m[0m
          [31m-[0m[0m gpus          = 0 [90m-> null[0m[0m
          [31m-[0m[0m memory        = 2 [90m-> null[0m[0m
        }

      [31m-[0m[0m scheduling_policy {
          [31m-[0m[0m preemptible = false [90m-> null[0m[0m
        }
    }

[1m  # yandex_compute_instance.k8s-cluster["k8s-node2"][0m will be [1m[31mdestroyed[0m
[0m  [31m-[0m[0m resource "yandex_compute_instance" "k8s-cluster" {
      [31m-[0m[0m created_at                = "2024-02-15T13:28:20Z" [90m-> null[0m[0m
      [31m-[0m[0m folder_id                 = "b1gfkh66dgk5tpn7ajko" [90m-> null[0m[0m
      [31m-[0m[0m fqdn                      = "k8s-node2.diplom.local" [90m-> null[0m[0m
      [31m-[0m[0m hostname                  = "k8s-node2.diplom.local" [90m-> null[0m[0m
      [31m-[0m[0m id                        = "fhm10404qco16b0d3j77" [90m-> null[0m[0m
      [31m-[0m[0m labels                    = {} [90m-> null[0m[0m
      [31m-[0m[0m metadata                  = {
          [31m-[0m[0m "serial-port-enable" = "1"
          [31m-[0m[0m "ssh-keys"           = <<-EOT
                ubuntu:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC7KoRsmTb6EC67YSxCrvoqTLhZcBgEd7lM4LebHU5p7qBtvyxA+oUxkH1Z4HjR2zbKEzDqtFfgREN5gaMFzHG/pa7RgMPR1c5lBynlsCzHDQqKt9417IoXfT71bFo2Mh+wXHo9gcJw9tU+4pab93zMBN7a40XtKWYQFsSQ0TffpprCcBkkqBeuBS9BFh3oaL7hRIOWPmUwpXmhY+Te7oaluj8fPxq/2ahPF1kthLY1NTLQvRGbrxENxsysi0jZwfDkYzWdWqhI5BA8fyG8e1wcpTM+9U8F4sAGMBxI5vEcC1J/Eb940tnEfp+oeZzcrSH9IRVCOgyGnEBh/qloxP7HVlErpX7KGf8h6s18h4d4mi5ylgBBgYfn/Wszej+Yotdgc+TJYihe/dDRulsxEcU/UQjIheqIoIIZApnajk0GdEyOtZpU/cY+CrtBH2czMLlNpn4lEOkcaCQu2XeqKn2yQNiOqz8jsDrVrTRrJHTEZ7+kkUROqmuNtcJdWl+4YH0= denis@denis-lin
            EOT
        } [90m-> null[0m[0m
      [31m-[0m[0m name                      = "k8s-node2" [90m-> null[0m[0m
      [31m-[0m[0m network_acceleration_type = "standard" [90m-> null[0m[0m
      [31m-[0m[0m platform_id               = "standard-v1" [90m-> null[0m[0m
      [31m-[0m[0m status                    = "running" [90m-> null[0m[0m
      [31m-[0m[0m zone                      = "ru-central1-a" [90m-> null[0m[0m

      [31m-[0m[0m boot_disk {
          [31m-[0m[0m auto_delete = true [90m-> null[0m[0m
          [31m-[0m[0m device_name = "fhmadov7jg2403jng5d1" [90m-> null[0m[0m
          [31m-[0m[0m disk_id     = "fhmadov7jg2403jng5d1" [90m-> null[0m[0m
          [31m-[0m[0m mode        = "READ_WRITE" [90m-> null[0m[0m

          [31m-[0m[0m initialize_params {
              [31m-[0m[0m block_size = 4096 [90m-> null[0m[0m
              [31m-[0m[0m image_id   = "fd8tckeqoshi403tks4l" [90m-> null[0m[0m
              [31m-[0m[0m size       = 20 [90m-> null[0m[0m
              [31m-[0m[0m type       = "network-hdd" [90m-> null[0m[0m
            }
        }

      [31m-[0m[0m metadata_options {
          [31m-[0m[0m aws_v1_http_endpoint = 1 [90m-> null[0m[0m
          [31m-[0m[0m aws_v1_http_token    = 2 [90m-> null[0m[0m
          [31m-[0m[0m gce_http_endpoint    = 1 [90m-> null[0m[0m
          [31m-[0m[0m gce_http_token       = 1 [90m-> null[0m[0m
        }

      [31m-[0m[0m network_interface {
          [31m-[0m[0m index              = 0 [90m-> null[0m[0m
          [31m-[0m[0m ip_address         = "192.168.10.10" [90m-> null[0m[0m
          [31m-[0m[0m ipv4               = true [90m-> null[0m[0m
          [31m-[0m[0m ipv6               = false [90m-> null[0m[0m
          [31m-[0m[0m mac_address        = "d0:0d:10:10:04:d3" [90m-> null[0m[0m
          [31m-[0m[0m nat                = true [90m-> null[0m[0m
          [31m-[0m[0m nat_ip_address     = "158.160.40.26" [90m-> null[0m[0m
          [31m-[0m[0m nat_ip_version     = "IPV4" [90m-> null[0m[0m
          [31m-[0m[0m security_group_ids = [] [90m-> null[0m[0m
          [31m-[0m[0m subnet_id          = "e9bp6i4phmsbasju46hb" [90m-> null[0m[0m
        }

      [31m-[0m[0m placement_policy {
          [31m-[0m[0m host_affinity_rules       = [] [90m-> null[0m[0m
          [31m-[0m[0m placement_group_partition = 0 [90m-> null[0m[0m
        }

      [31m-[0m[0m resources {
          [31m-[0m[0m core_fraction = 100 [90m-> null[0m[0m
          [31m-[0m[0m cores         = 2 [90m-> null[0m[0m
          [31m-[0m[0m gpus          = 0 [90m-> null[0m[0m
          [31m-[0m[0m memory        = 2 [90m-> null[0m[0m
        }

      [31m-[0m[0m scheduling_policy {
          [31m-[0m[0m preemptible = false [90m-> null[0m[0m
        }
    }

[1m  # yandex_compute_instance.k8s-cluster["k8s-node3"][0m will be [1m[31mdestroyed[0m
[0m  [31m-[0m[0m resource "yandex_compute_instance" "k8s-cluster" {
      [31m-[0m[0m created_at                = "2024-02-15T13:28:20Z" [90m-> null[0m[0m
      [31m-[0m[0m folder_id                 = "b1gfkh66dgk5tpn7ajko" [90m-> null[0m[0m
      [31m-[0m[0m fqdn                      = "k8s-node3.diplom.local" [90m-> null[0m[0m
      [31m-[0m[0m hostname                  = "k8s-node3.diplom.local" [90m-> null[0m[0m
      [31m-[0m[0m id                        = "fhmnne713httg57trv06" [90m-> null[0m[0m
      [31m-[0m[0m labels                    = {} [90m-> null[0m[0m
      [31m-[0m[0m metadata                  = {
          [31m-[0m[0m "serial-port-enable" = "1"
          [31m-[0m[0m "ssh-keys"           = <<-EOT
                ubuntu:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC7KoRsmTb6EC67YSxCrvoqTLhZcBgEd7lM4LebHU5p7qBtvyxA+oUxkH1Z4HjR2zbKEzDqtFfgREN5gaMFzHG/pa7RgMPR1c5lBynlsCzHDQqKt9417IoXfT71bFo2Mh+wXHo9gcJw9tU+4pab93zMBN7a40XtKWYQFsSQ0TffpprCcBkkqBeuBS9BFh3oaL7hRIOWPmUwpXmhY+Te7oaluj8fPxq/2ahPF1kthLY1NTLQvRGbrxENxsysi0jZwfDkYzWdWqhI5BA8fyG8e1wcpTM+9U8F4sAGMBxI5vEcC1J/Eb940tnEfp+oeZzcrSH9IRVCOgyGnEBh/qloxP7HVlErpX7KGf8h6s18h4d4mi5ylgBBgYfn/Wszej+Yotdgc+TJYihe/dDRulsxEcU/UQjIheqIoIIZApnajk0GdEyOtZpU/cY+CrtBH2czMLlNpn4lEOkcaCQu2XeqKn2yQNiOqz8jsDrVrTRrJHTEZ7+kkUROqmuNtcJdWl+4YH0= denis@denis-lin
            EOT
        } [90m-> null[0m[0m
      [31m-[0m[0m name                      = "k8s-node3" [90m-> null[0m[0m
      [31m-[0m[0m network_acceleration_type = "standard" [90m-> null[0m[0m
      [31m-[0m[0m platform_id               = "standard-v1" [90m-> null[0m[0m
      [31m-[0m[0m status                    = "running" [90m-> null[0m[0m
      [31m-[0m[0m zone                      = "ru-central1-a" [90m-> null[0m[0m

      [31m-[0m[0m boot_disk {
          [31m-[0m[0m auto_delete = true [90m-> null[0m[0m
          [31m-[0m[0m device_name = "fhmfk50ke0spq15rv2va" [90m-> null[0m[0m
          [31m-[0m[0m disk_id     = "fhmfk50ke0spq15rv2va" [90m-> null[0m[0m
          [31m-[0m[0m mode        = "READ_WRITE" [90m-> null[0m[0m

          [31m-[0m[0m initialize_params {
              [31m-[0m[0m block_size = 4096 [90m-> null[0m[0m
              [31m-[0m[0m image_id   = "fd8tckeqoshi403tks4l" [90m-> null[0m[0m
              [31m-[0m[0m size       = 20 [90m-> null[0m[0m
              [31m-[0m[0m type       = "network-hdd" [90m-> null[0m[0m
            }
        }

      [31m-[0m[0m metadata_options {
          [31m-[0m[0m aws_v1_http_endpoint = 1 [90m-> null[0m[0m
          [31m-[0m[0m aws_v1_http_token    = 2 [90m-> null[0m[0m
          [31m-[0m[0m gce_http_endpoint    = 1 [90m-> null[0m[0m
          [31m-[0m[0m gce_http_token       = 1 [90m-> null[0m[0m
        }

      [31m-[0m[0m network_interface {
          [31m-[0m[0m index              = 0 [90m-> null[0m[0m
          [31m-[0m[0m ip_address         = "192.168.10.13" [90m-> null[0m[0m
          [31m-[0m[0m ipv4               = true [90m-> null[0m[0m
          [31m-[0m[0m ipv6               = false [90m-> null[0m[0m
          [31m-[0m[0m mac_address        = "d0:0d:17:bb:8e:11" [90m-> null[0m[0m
          [31m-[0m[0m nat                = true [90m-> null[0m[0m
          [31m-[0m[0m nat_ip_address     = "178.154.207.84" [90m-> null[0m[0m
          [31m-[0m[0m nat_ip_version     = "IPV4" [90m-> null[0m[0m
          [31m-[0m[0m security_group_ids = [] [90m-> null[0m[0m
          [31m-[0m[0m subnet_id          = "e9bp6i4phmsbasju46hb" [90m-> null[0m[0m
        }

      [31m-[0m[0m placement_policy {
          [31m-[0m[0m host_affinity_rules       = [] [90m-> null[0m[0m
          [31m-[0m[0m placement_group_partition = 0 [90m-> null[0m[0m
        }

      [31m-[0m[0m resources {
          [31m-[0m[0m core_fraction = 100 [90m-> null[0m[0m
          [31m-[0m[0m cores         = 2 [90m-> null[0m[0m
          [31m-[0m[0m gpus          = 0 [90m-> null[0m[0m
          [31m-[0m[0m memory        = 2 [90m-> null[0m[0m
        }

      [31m-[0m[0m scheduling_policy {
          [31m-[0m[0m preemptible = false [90m-> null[0m[0m
        }
    }

[1m  # yandex_vpc_network.diplom-network[0m will be [1m[31mdestroyed[0m
[0m  [31m-[0m[0m resource "yandex_vpc_network" "diplom-network" {
      [31m-[0m[0m created_at                = "2024-02-15T13:28:15Z" [90m-> null[0m[0m
      [31m-[0m[0m default_security_group_id = "enpjtipc1cccj22qrnne" [90m-> null[0m[0m
      [31m-[0m[0m folder_id                 = "b1gfkh66dgk5tpn7ajko" [90m-> null[0m[0m
      [31m-[0m[0m id                        = "enp48huj8ab5ubime77m" [90m-> null[0m[0m
      [31m-[0m[0m labels                    = {} [90m-> null[0m[0m
      [31m-[0m[0m name                      = "diplom-network" [90m-> null[0m[0m
      [31m-[0m[0m subnet_ids                = [
          [31m-[0m[0m "e2lbtdq0id3k2avv45sm",
          [31m-[0m[0m "e9bp6i4phmsbasju46hb",
        ] [90m-> null[0m[0m
    }

[1m  # yandex_vpc_subnet.diplom-network-subnet-a[0m will be [1m[31mdestroyed[0m
[0m  [31m-[0m[0m resource "yandex_vpc_subnet" "diplom-network-subnet-a" {
      [31m-[0m[0m created_at     = "2024-02-15T13:28:18Z" [90m-> null[0m[0m
      [31m-[0m[0m folder_id      = "b1gfkh66dgk5tpn7ajko" [90m-> null[0m[0m
      [31m-[0m[0m id             = "e9bp6i4phmsbasju46hb" [90m-> null[0m[0m
      [31m-[0m[0m labels         = {} [90m-> null[0m[0m
      [31m-[0m[0m name           = "diplom-network-subnet-a" [90m-> null[0m[0m
      [31m-[0m[0m network_id     = "enp48huj8ab5ubime77m" [90m-> null[0m[0m
      [31m-[0m[0m v4_cidr_blocks = [
          [31m-[0m[0m "192.168.10.0/24",
        ] [90m-> null[0m[0m
      [31m-[0m[0m v6_cidr_blocks = [] [90m-> null[0m[0m
      [31m-[0m[0m zone           = "ru-central1-a" [90m-> null[0m[0m
    }

[1m  # yandex_vpc_subnet.diplom-network-subnet-b[0m will be [1m[31mdestroyed[0m
[0m  [31m-[0m[0m resource "yandex_vpc_subnet" "diplom-network-subnet-b" {
      [31m-[0m[0m created_at     = "2024-02-15T13:28:18Z" [90m-> null[0m[0m
      [31m-[0m[0m folder_id      = "b1gfkh66dgk5tpn7ajko" [90m-> null[0m[0m
      [31m-[0m[0m id             = "e2lbtdq0id3k2avv45sm" [90m-> null[0m[0m
      [31m-[0m[0m labels         = {} [90m-> null[0m[0m
      [31m-[0m[0m name           = "diplom-network-subnet-b" [90m-> null[0m[0m
      [31m-[0m[0m network_id     = "enp48huj8ab5ubime77m" [90m-> null[0m[0m
      [31m-[0m[0m v4_cidr_blocks = [
          [31m-[0m[0m "192.168.11.0/24",
        ] [90m-> null[0m[0m
      [31m-[0m[0m v6_cidr_blocks = [] [90m-> null[0m[0m
      [31m-[0m[0m zone           = "ru-central1-b" [90m-> null[0m[0m
    }

[1mPlan:[0m 0 to add, 0 to change, 7 to destroy.
[0m
Changes to Outputs:
  [31m-[0m[0m cluster_ips = {
      [31m-[0m[0m external = [
          [31m-[0m[0m "158.160.116.25",
          [31m-[0m[0m "158.160.40.26",
          [31m-[0m[0m "178.154.207.84",
        ]
      [31m-[0m[0m internal = [
          [31m-[0m[0m "192.168.10.4",
          [31m-[0m[0m "192.168.10.10",
          [31m-[0m[0m "192.168.10.13",
        ]
    } [90m-> null[0m[0m
[0m[1myandex_vpc_subnet.diplom-network-subnet-b: Destroying... [id=e2lbtdq0id3k2avv45sm][0m[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node3"]: Destroying... [id=fhmnne713httg57trv06][0m[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node2"]: Destroying... [id=fhm10404qco16b0d3j77][0m[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node1"]: Destroying... [id=fhm5htkn04et2l8khpok][0m[0m
[0m[1myandex_vpc_subnet.diplom-network-subnet-b: Destruction complete after 1s[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node1"]: Still destroying... [id=fhm5htkn04et2l8khpok, 10s elapsed][0m[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node3"]: Still destroying... [id=fhmnne713httg57trv06, 10s elapsed][0m[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node2"]: Still destroying... [id=fhm10404qco16b0d3j77, 10s elapsed][0m[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node2"]: Still destroying... [id=fhm10404qco16b0d3j77, 20s elapsed][0m[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node3"]: Still destroying... [id=fhmnne713httg57trv06, 20s elapsed][0m[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node1"]: Still destroying... [id=fhm5htkn04et2l8khpok, 20s elapsed][0m[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node2"]: Still destroying... [id=fhm10404qco16b0d3j77, 30s elapsed][0m[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node3"]: Still destroying... [id=fhmnne713httg57trv06, 30s elapsed][0m[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node1"]: Still destroying... [id=fhm5htkn04et2l8khpok, 30s elapsed][0m[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node3"]: Still destroying... [id=fhmnne713httg57trv06, 40s elapsed][0m[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node2"]: Still destroying... [id=fhm10404qco16b0d3j77, 40s elapsed][0m[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node1"]: Still destroying... [id=fhm5htkn04et2l8khpok, 40s elapsed][0m[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node3"]: Destruction complete after 40s[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node1"]: Destruction complete after 44s[0m
[0m[1myandex_compute_instance.k8s-cluster["k8s-node2"]: Destruction complete after 48s[0m
[0m[1mrandom_shuffle.diplom-network-subnet-random: Destroying... [id=-][0m[0m
[0m[1mrandom_shuffle.diplom-network-subnet-random: Destruction complete after 0s[0m
[0m[1myandex_vpc_subnet.diplom-network-subnet-a: Destroying... [id=e9bp6i4phmsbasju46hb][0m[0m
[0m[1myandex_vpc_subnet.diplom-network-subnet-a: Destruction complete after 3s[0m
[0m[1myandex_vpc_network.diplom-network: Destroying... [id=enp48huj8ab5ubime77m][0m[0m
[0m[1myandex_vpc_network.diplom-network: Destruction complete after 1s[0m
[0m[1m[32m
Destroy complete! Resources: 7 destroyed.
[0msection_end:1708004133:step_script
[0Ksection_start:1708004133:cleanup_file_variables
[0K[0K[36;1mCleaning up project directory and file based variables[0;m[0;m
section_end:1708004133:cleanup_file_variables
[0K[32;1mJob succeeded[0;m
```
![tf4](img/tf4.png)

![tf5](img/tf5.png)

</details>

</details>

<details><summary>Установка GitlabRunner в K8S</summary>

```bash
kubectl create namespace gitlab
```

Дальше создаем обычный раннер в гилабе (можно для конкретного проекта, можно групповой) нам необхомим runner-token который мы прописываем в secret.   
Создаем секрсет в кубуре:

```bash
kubectl apply -f secret.yaml
```

Устанавливаем раннер (values можно отредактировать сразу или в процессе при необходимости будет удобно вносить изменения в конфгурацию раннера):

```bash
helm repo add gitlab https://charts.gitlab.io
helm repo update
helm install gitlab-runner gitlab/gitlab-runner -f my-values.yaml -n gitlab
```

[secret.yaml](file/gitlab/secret.yaml)   
[my-values.yaml](file/gitlab/my-values.yaml)   

Предоставим права администратора сервисной учетной записи «defaul» в пространстве имен «gitlab»

```bash
kubectl create clusterrolebinding gitlab-is-now-admin   --clusterrole=admin   --serviceaccount=gitlab:default
```

* Данный способ создания раннера удобен тем что нам нет необходимости, каждый раз создавать новый раннер.
* В том числе при пересоздании окружения можем использовать тот же раннер


```bash
root@k8s-node1:/home/ubuntu# kubectl -n gitlab get po
NAME                           READY   STATUS    RESTARTS   AGE
gitlab-runner-cd899f6c-c59x5   1/1     Running   0          39s

```

</details>

<details><summary>Http доступ к тестовому приложению</summary>

Репозиторий с ci/cd для gitlab (расположен на github) [diplom-site](https://github.com/VodyakovDenis/diplom-site)

![site1](img/site1.png)

![site2](img/site2.png)

![site3](img/site3.png)

![site4](img/site4.png)

![site5](img/site5.png)

<details><summary>website</summary>

[raw](https://gitlab.it-git.ru/test/diplom-site/-/jobs/332/raw)

```txt
[0KRunning with gitlab-runner 16.8.0 (c72a09b6)[0;m
[0K  on gitlab-runner-cd899f6c-c59x5 WGXp7y8sR, system ID: r_Tgu3xwZqfvt8[0;m
section_start:1708007992:prepare_executor
[0K[0K[36;1mPreparing the "kubernetes" executor[0;m[0;m
[0KUsing Kubernetes namespace: gitlab[0;m
[0KUsing Kubernetes executor with image gcr.io/kaniko-project/executor:v1.11.0-debug ...[0;m
[0KUsing attach strategy to execute scripts...[0;m
section_end:1708007992:prepare_executor
[0Ksection_start:1708007992:prepare_script
[0K[0K[36;1mPreparing environment[0;m[0;m
[0KUsing FF_USE_POD_ACTIVE_DEADLINE_SECONDS, the Pod activeDeadlineSeconds will be set to the job timeout: 1h0m0s...[0;m
Waiting for pod gitlab/runner-wgxp7y8sr-project-2-concurrent-0-obj34nlc to be running, status is Pending
Waiting for pod gitlab/runner-wgxp7y8sr-project-2-concurrent-0-obj34nlc to be running, status is Pending
	ContainersNotInitialized: "containers with incomplete status: [init-permissions]"
	ContainersNotReady: "containers with unready status: [build helper]"
	ContainersNotReady: "containers with unready status: [build helper]"
Waiting for pod gitlab/runner-wgxp7y8sr-project-2-concurrent-0-obj34nlc to be running, status is Pending
	ContainersNotInitialized: "containers with incomplete status: [init-permissions]"
	ContainersNotReady: "containers with unready status: [build helper]"
	ContainersNotReady: "containers with unready status: [build helper]"
Waiting for pod gitlab/runner-wgxp7y8sr-project-2-concurrent-0-obj34nlc to be running, status is Pending
	ContainersNotReady: "containers with unready status: [build helper]"
	ContainersNotReady: "containers with unready status: [build helper]"
Waiting for pod gitlab/runner-wgxp7y8sr-project-2-concurrent-0-obj34nlc to be running, status is Pending
	ContainersNotReady: "containers with unready status: [build helper]"
	ContainersNotReady: "containers with unready status: [build helper]"
Waiting for pod gitlab/runner-wgxp7y8sr-project-2-concurrent-0-obj34nlc to be running, status is Pending
	ContainersNotReady: "containers with unready status: [build helper]"
	ContainersNotReady: "containers with unready status: [build helper]"
Running on runner-wgxp7y8sr-project-2-concurrent-0-obj34nlc via gitlab-runner-cd899f6c-c59x5...

section_end:1708008010:prepare_script
[0Ksection_start:1708008010:get_sources
[0K[0K[36;1mGetting source from Git repository[0;m[0;m
[32;1mFetching changes with git depth set to 20...[0;m
Initialized empty Git repository in /builds/test/diplom-site/.git/
[32;1mCreated fresh repository.[0;m
[32;1mChecking out 06cfd51d as detached HEAD (ref is 0.1.0)...[0;m

[32;1mSkipping Git submodules setup[0;m

section_end:1708008012:get_sources
[0Ksection_start:1708008012:step_script
[0K[0K[36;1mExecuting "step_script" stage of the job script[0;m[0;m
[32;1m$ echo "Authenticating with Docker registry..."[0;m
Authenticating with Docker registry...
[32;1m$ echo "{\"auths\":{\"${REGISTRY}\":{\"auth\":\"$(printf "%s:%s" "${DOCKER_USER}" "${DOCKER_PASS}" | base64 | tr -d '\n')\"}}}" > /kaniko/.docker/config.json[0;m
[32;1m$ echo "Building container..."[0;m
Building container...
[32;1m$ /kaniko/executor --context "${CI_PROJECT_DIR}/website" --dockerfile "${CI_PROJECT_DIR}/website/Dockerfile" --destination "${CONTAINER_NAME}:${CI_COMMIT_TAG}"[0;m
[36mINFO[0m[0001] Retrieving image manifest nginx:1.22.0-alpine 
[36mINFO[0m[0001] Retrieving image nginx:1.22.0-alpine from registry index.docker.io 
[36mINFO[0m[0002] Built cross stage deps: map[]                
[36mINFO[0m[0002] Retrieving image manifest nginx:1.22.0-alpine 
[36mINFO[0m[0002] Returning cached image manifest              
[36mINFO[0m[0002] Executing 0 build triggers                   
[36mINFO[0m[0002] Building stage 'nginx:1.22.0-alpine' [idx: '0', base-idx: '-1'] 
[36mINFO[0m[0002] Unpacking rootfs as cmd COPY site /usr/share/nginx/html requires it. 
[36mINFO[0m[0004] COPY site /usr/share/nginx/html              
[36mINFO[0m[0004] Taking snapshot of files...                  
[36mINFO[0m[0004] Pushing image to vodyakovdenis/diplom-test-site:0.1.0 
[36mINFO[0m[0007] Pushed index.docker.io/vodyakovdenis/diplom-test-site@sha256:37c83059f9f18e6edfc67bc431e849d53d0bb50a537e2605e133b3a55484b7a9 

section_end:1708008021:step_script
[0Ksection_start:1708008021:cleanup_file_variables
[0K[0K[36;1mCleaning up project directory and file based variables[0;m[0;m

section_end:1708008021:cleanup_file_variables
[0K[32;1mJob succeeded[0;m
```

</details>

<details><summary>deploy</summary>

[raw](https://gitlab.it-git.ru/test/diplom-site/-/jobs/334/raw)

```txt
[0KRunning with gitlab-runner 16.8.0 (c72a09b6)[0;m
[0K  on gitlab-runner-cd899f6c-c59x5 WGXp7y8sR, system ID: r_Tgu3xwZqfvt8[0;m
section_start:1708008303:prepare_executor
[0K[0K[36;1mPreparing the "kubernetes" executor[0;m[0;m
[0KUsing Kubernetes namespace: gitlab[0;m
[0KUsing Kubernetes executor with image gcr.io/cloud-builders/kubectl:latest ...[0;m
[0KUsing attach strategy to execute scripts...[0;m
section_end:1708008303:prepare_executor
[0Ksection_start:1708008303:prepare_script
[0K[0K[36;1mPreparing environment[0;m[0;m
[0KUsing FF_USE_POD_ACTIVE_DEADLINE_SECONDS, the Pod activeDeadlineSeconds will be set to the job timeout: 1h0m0s...[0;m
Waiting for pod gitlab/runner-wgxp7y8sr-project-2-concurrent-0-c5kki1hy to be running, status is Pending
Waiting for pod gitlab/runner-wgxp7y8sr-project-2-concurrent-0-c5kki1hy to be running, status is Pending
	ContainersNotReady: "containers with unready status: [build helper]"
	ContainersNotReady: "containers with unready status: [build helper]"
Running on runner-wgxp7y8sr-project-2-concurrent-0-c5kki1hy via gitlab-runner-cd899f6c-c59x5...

section_end:1708008316:prepare_script
[0Ksection_start:1708008316:get_sources
[0K[0K[36;1mGetting source from Git repository[0;m[0;m
[32;1mFetching changes with git depth set to 20...[0;m
Initialized empty Git repository in /builds/test/diplom-site/.git/
[32;1mCreated fresh repository.[0;m
[32;1mChecking out 06cfd51d as detached HEAD (ref is 0.1.0)...[0;m

[32;1mSkipping Git submodules setup[0;m

section_end:1708008327:get_sources
[0Ksection_start:1708008327:step_script
[0K[0K[36;1mExecuting "step_script" stage of the job script[0;m[0;m
[32;1m$ kubectl config set-cluster kubernetes --server="$KUBE_URL" --insecure-skip-tls-verify=true[0;m
Cluster "kubernetes" set.
[32;1m$ kubectl config set-credentials admin --token="$KUBE_TOKEN"[0;m
User "admin" set.
[32;1m$ kubectl config set-context default --cluster=kubernetes --user=admin[0;m
Context "default" created.
[32;1m$ kubectl config use-context default[0;m
Switched to context "default".
[32;1m$ sed -i "s/__VERSION__/$CI_COMMIT_TAG/" diplom-app.yml[0;m
[32;1m$ kubectl apply -f diplom-app.yml[0;m
deployment.apps/diplom-app created
service/diplom-app created
ingress.networking.k8s.io/diplom-app created

section_end:1708008344:step_script
[0Ksection_start:1708008344:cleanup_file_variables
[0K[0K[36;1mCleaning up project directory and file based variables[0;m[0;m

section_end:1708008345:cleanup_file_variables
[0K[32;1mJob succeeded[0;m
```

</details>

```bash
root@k8s-node1:/home/ubuntu# kubectl -n gitlab get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
diplom-app   ClusterIP   10.245.134.185   <none>        80/TCP    2m21s
root@k8s-node1:/home/ubuntu# kubectl -n gitlab get ingress
NAME         CLASS   HOSTS              ADDRESS          PORTS   AGE
diplom-app   nginx   denis.diplom.com   158.160.52.151   80      2m25s
```

![website](img/website.png)

</details>

---

**Комментарии:**   

* Http доступ к тестовому приложению - выполнен через ci/cd в последнем разделе
* Для запуска ci/cd терраформ использовался раннер запущенный на VM (создан там-же в прошлых ДЗ)
* Для запуска ci/cd по публикации веб странички использовался раннер запущенный внутри K8S

**Список репозиториев собранные на github:**

- [diplom-app](https://github.com/VodyakovDenis/diplom-app)   
- [diplom-terraform](https://github.com/VodyakovDenis/diplom-terraform)
- [diplom-site](https://github.com/VodyakovDenis/diplom-site)
- [diplom-ansible](https://github.com/VodyakovDenis/diplom-ansible)


<details><summary>* Полезные ссылки которые пригодились</summary>

https://siebjee.nl/posts/ingress-nginx-context-deadline-exceeded/   
https://cloud.yandex.ru/ru/docs/managed-kubernetes/tutorials/prometheus-grafana-monitoring   
https://cloud.yandex.ru/ru/docs/application-load-balancer/quickstart   
https://docs.nginx.com/nginx-ingress-controller/installation/installing-nic/installation-with-helm/   
https://selectel.ru/blog/tutorials/monitoring-in-k8s-with-prometheus/   
https://ru.stackoverflow.com/questions/931025/%D0%9D%D0%B5-%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%B0%D0%B5%D1%82-dns-kubernetes-%D0%B2%D0%BD%D1%83%D1%82%D1%80%D0%B8-%D0%BA%D0%BE%D0%BD%D1%82%D0%B5%D0%B9%D0%BD%D0%B5%D1%80%D0%BE%D0%B2
https://kubernetes.io/ru/docs/reference/kubectl/cheatsheet/
https://kubernetes.io/docs/reference/config-api/kube-proxy-config.v1alpha1/   
https://helm.sh/ru/docs/intro/using_helm/   
https://nixhub.ru/posts/k8s-cluster-access/   
https://docs.gitlab.com/ee/topics/autodevops/troubleshooting.html   
https://docs.gitlab.com/ee/user/clusters/agent/ci_cd_workflow.html 

</details>