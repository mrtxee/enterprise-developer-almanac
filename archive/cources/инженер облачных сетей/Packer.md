---
aliases:
  - Packer
---
## Packer — упаковщик образов вм

**Packer** — это _open-source_ инструмент для **создания идентичных машинных образов** для множества платформ из одного исходного файла конфигурации. Т.е с пакером можно автоматизировать создание образов для _Amazon_ _[[EC2]]_, _VMware_, _Docker_ и т.д, используя **единый процесс сборки.**

- шаблон — образ ВМ с настроенным софтом

Packer docs: [https://www.packer.io/docs/](https://www.packer.io/docs/)

### как создать образ при помощи Packer

Packer работает так: на входе вы даёте ему текстовый файл — **спецификацию** — с описанием сборки образа, а на выходе получаете готовый образ. Описание образа можно составить на языке HCL ([HashiCorp Language](https://www.packer.io/docs/templates/hcl_templates)) или с помощью обычного JSON. Вариант с `HCL` рекомендован

- готовые конфигурации в JSON, то их можно конвертировать в HCL с помощью команды [packer hc2_upgrade](https://www.packer.io/docs/commands/hcl2_upgrade).
    
- пример описания образа с _Ubuntu_ и веб-сервером NGINX на HCL
    
    ```jsx
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
    
    - комментарии
        
        В конфигурации выше, например, задан ключ `token`. Еще один способ — записать [IAM-токен](https://cloud.yandex.ru/docs/iam/concepts/authorization/iam-token) или [OAuth-токен](https://cloud.yandex.ru/docs/iam/concepts/authorization/oauth-token) в переменную окружения `YC_TOKEN`, тогда в самой спецификации можно дополнительно ничего не указывать.
        
        В параметре `image_name` мы указываем имя образа.
        
        В секции `provisioner` — команды, которые нужно выполнить при сборке образа. В нашем случае это установка _NGINX_.
        
        Сохраним конфигурацию в файл `my-ubuntu-nginx.pkr.hcl` и попросим _Packer_ на его основе создать образ ВМ:
        
        `packer build <путь_к_my-ubuntu-nginx.pkr.hcl>`