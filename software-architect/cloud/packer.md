---
aliases:
  - Packer
  - HashiCorp
  - HashiCorp Language
  - HCL
  - JSON
  - machine image
  - image
  - virtual machine
  - VM
  - build
  - template
  - specification
  - configuration
  - environment variable
  - variable
  - source
  - provisioner
  - Amazon
  - EC2
  - VMware
  - Docker
  - Ubuntu
  - NGINX
  - packer build
  - packer hc2_upgrade
  - упаковщик
  - машинный образ
  - образ
  - виртуальная машина
  - ВМ
  - сборка
  - шаблон
  - спецификация
  - конфигурация
  - переменная окружения
  - переменная
---

## Packer — упаковщик образов ВМ

**Packer** — это open-source инструмент для создания идентичных машинных образов для множества платформ из одного исходного файла конфигурации. То есть с Packer можно автоматизировать создание образов для Amazon [[EC2]], VMware, Docker и т.д., используя **единый процесс сборки**.

- шаблон — образ ВМ с настроенным софтом

### Создание образа с помощью Packer

Packer работает так: на вход подаётся текстовый файл — спецификация — с описанием сборки образа, на выходе получается готовый образ. Описание образа можно составить на языке HCL (HashiCorp Language) или с помощью обычного JSON. Вариант с HCL рекомендован.

- Готовые конфигурации на JSON можно конвертировать в HCL с помощью команды `packer hc2_upgrade`.
- Пример описания образа с Ubuntu и веб-сервером NGINX на HCL:

```hcl
source "yandex" "ubuntu-nginx" {
  token               = "<OAuth-токен>"
  folder_id           = "<идентификатор_каталога>"
  source_image_family = "ubuntu-2004-lts"
  ssh_username        = "ubuntu"
  use_ipv4_nat        = "true"
  image_description   = "my custom ubuntu with nginx"
  image_family        = "ubuntu-2004-lts"
  image_name          = "my-ubuntu-nginx"
  subnet_id           = "<идентификатор подсети>"
  disk_type           = "network-ssd"
  zone                = "ru-central1-a"
}

build {
  sources = ["source.yandex.ubuntu-nginx"]

  provisioner "shell" {
    inline = ["sudo apt-get update -y",
              "sudo apt-get install -y nginx",
              "sudo systemctl enable nginx.service"
             ]
  }
}
```

**Комментарии к конфигурации**

В конфигурации выше, например, задан ключ `token`. Другой способ — записать IAM-токен или OAuth-токен в переменную окружения `YC_TOKEN`, тогда в самой спецификации можно дополнительно ничего не указывать.

В параметре `image_name` указывается имя образа.

В секции `provisioner` — команды, которые нужно выполнить при сборке образа. В данном случае это установка NGINX.

Конфигурация сохраняется в файл `my-ubuntu-nginx.pkr.hcl`, после чего на её основе создаётся образ ВМ:

```bash
packer build <путь_к_my-ubuntu-nginx.pkr.hcl>
```
