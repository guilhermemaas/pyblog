---
title: "Zabbix em 3 camadas"
date: 2020-11-23T15:44:47-03:00
description: Instalando o Zabbix em 3 camadas com CentOS e MariaDB
draft: false
hideToc: true
enableToc: true
enableTocContent: false
tags:
- terminal
- linux
- zabbix
- centos
categories:
- terminal
- linux
- monitoramento
image: images/icones/zabbix.jpg
---

{{< img src="img/zabbix_3_camadas_mariadb.jpg" position="center" >}}

## 

Esse é um guia rápido, prático e direto sobre como instalar o Zabbix em três camadas utilizando CentOS como sistema operacional, PostgreSQL como SGDB e Apache para servir o front-end. Neste caso possuo três máquinas virtuais distintas no VirtualBox.

Esse passo a passo não aborda a instalação do VirtualBox. Assim sendo é necessário que se tenha o entendimento de como instalar o sistema operacional nas máquinas virtuais, como configurar a rede, como se conectar via ssh e os comandos básicos utilizados no terminal.

Todos os comandos possuem um comentário informando o motivo pelo qual está sendo executado.


{{< notice info "This is a success type of notice" >}}
Estes são os IP's que serão utilizados durante a instalação:

MariaDB: 192.168.0.118
Zabbix Backend: 192.168.0.117
Zabbix Front-end: 192.168.0.119
{{< /notice >}}


## 🐧 Configurações base para os três servidores

Os passos abaixo podem ser executados nos três servidores. Tratam-se de configuração de Timezone(data), serviço NTP e firewall.

{{< codes bash >}}
  {{< code >}}
  ```bash
  timedatectl status
  #Verificar o timezone que está configurado no servidor.

  timedatectl list-timezones
  #Lista todos timezones disponíveis.

  timedatectl set-timezone Amercia/Sao_Paulo
  #Seta o timezone de São Paulo.

  date
  #Verifique se a hora está configurada corretamente.

  dnf -y install chrony
  #Instala o Chrony(Cliente NTP) para sincronizar a data e hora automaticamente.
  #Por padrão o Chrony sincroniza o horário a cada 64 segundos.

  systemctl enable --now chronyd 
  #Habilita o Chrony como serviço.

  service chronyd restart
  #Valide se o serviço sobe corretamente.

  date
  #Novamente date para validar a data e horário do servidor.

  firewall-cmd --permanent --add-service=ntp
  firewall-cmd --reload
  #Adiciona regra de firewall para o NTP.

  dnf install -y net-tools vim nano epel-release wget curl tcpdump htop
  #Algumas ferramentas úties na administração do servidor.

  getenforce
  #Verificar o status do selinux para desativar.

  vim /etc/selinux/config
  SELINUX=disabled
  #Neste caso, altere a chave acima para "disabled" e reinicie o servidor.

  getenforce 0
  getenforce
  #getenforce 0 desativa o selinux sem a necessidade de reiniciar. Utilize getenforce novamente para validar.
  ```
  {{< /code >}}
{{< /codes >}}


## 🦦 Servidor do MariaDB

{{< codes bash >}}
  {{< code >}}
  ```bash
  dnf info mariadb-server
  #Chega a versão disponível do MariaDB Server.

  dnf -y install mariadb-server
  #Instala o MariaDB.

  systemctl enable --now mariadb
  systemctl status mariadb
  #Habilita e ativa o serviço do MariaDB.

  mysql_secure_installation
  #Defina uma senha para o usuário root do MariaDB.

  mysql -u root -p
  #Conecte-se como root.

  create database zabbix character set utf8 collate utf8_bin;
  #Crie a base de dados para o Zabbix.

  create user 'zabbix'@'localhost' identified by 'xpto';
  #Crie um usuário local para acesso a base do Zabbix.

  grant all privileges on zabbix.* to 'zabbix'@'localhost';
  #Dê previlégios para o usuário criado.
  
  flush privileges;
  #Recarregue os previlégios.

  #Crie um usuário para acesso a partir do Zabbix Server:
  create user 'zabbix_server'@'192.168.0.117' identified by 'xpto';
  grant all privileges on zabbix.* to 'zabbix_server'@'192.168.0.117';
  flush privileges;

  #Crie um usuário para acesso a partir do Zabbix Front-end:
  create user 'zabbix_web'@'192.168.0.119' identified by 'xpto';
  grant all privileges on zabbix.* to 'zabbix_web'@'192.168.0.119';
  flush privileges;

  firewall-cmd --permanent --add-port=3306/tcp
  firewall-cmd --reload
  #Adicione uma regra para liberar a porta 3306 do MariaDB.

  vim /etc/my.cnf.d/mariadb-server.cnf
  bind-address = 192.168.0.118
  #Configure o MariaDB para aceitar conexões remotas(neste parâmetro utilize o pŕopio IP do servidor MariaDB).

  systemctl restart mariadb
  #Reinicie o servidor do MariaDB para aplicar as configurações.
  ```
  {{< /code >}}
{{< /codes >}}


## 🚨 Servidor do Zabbix Backend(Zabbix Server)

{{< codes bash >}}
  {{< code >}}
  ```bash
  rpm -ivh http://repo.zabbix.com/zabbix/5.2/rhel/8/x86_64/zabbix-release-5.2-1.el8.noarch.rpm
  #Intale o pacote do Zabbix do repositório oficial. Neste caso é possível validar se há uma versão mais rescente.

  dnf clean all
  #Limpar o cache e remover repositórios antigos.

  dnf -y install zabbix-server
  #Instale o Zabbix Server.

  dnf -y install mariadb
  #Instale o cliente do MariaDB.

  zcat /usr/share/doc/zabbix-server-mysql/create.sql.gz | mysql -h 192.168.0.118 -u zabbix_server -p zabbix
  #Utilize o comando acima para popular/carregar a estrutura/esquema inicial da base do Zabbix.

  #Verifique se as tabelas foram criadas/populadas:
  mysql -u root -p
  use zabbix;
  show tables;
  
  #Edite o arquivo de configuração do Zabbix, alterando as chaves abaixo:
  vim /etczabbix/zabbix_server.conf 
  DBHost=192.168.0.118
  DBUser=zabbix_server
  DBPassword=xpto
 
  #Verificar se nao esta logando erros:
  tail -f -n 20 /var/log/zabbix/zabbix_server.log

  firewall-cmd --permanent --add-port=10051/tcp
  firewall-cmd --reload
  #Criar regra no firewall para Zabbix Server.
  ```
  {{< /code >}}
{{< /codes >}}


## 🚨 Servidor do Zabbix Front-end(Zabbix Web)

{{< codes bash >}}
  {{< code >}}
  ```bash
  rpm -ivh http://repo.zabbix.com/zabbix/5.2/rhel/8/x86_64/zabbix-release-5.2-1.el8.noarch.rpm
  #Intale o pacote do Zabbix do repositório oficial. Neste caso é possível validar se há uma versão mais rescente.

  dnf clean all
  #Limpar o cache e remover repositórios antigos.

  dnf -y install zabbix-web-mysql zabbix-apache-conf
  #Instale o Zabbix Front-end(Web).

  vim /etc/php-fpm.d/zabbix.conf
  php_value[date.timezone] = America/Sao_Paulo
  #Ajustar o timezone do arquivo de config do Zabbix pra America/Sao_Paulo.

  systemctl enable --now httpd php-fpm
  systemctl status httpd php-fpm
  #Habilitar a inicialização do serviço do Apache.
  ```
  {{< /code >}}
{{< /codes >}}

A partir deste momento você já pode acessar o Zabbix Web:

{{< alert theme="info" dir="ltr" >}}
http://192.168.0.119/zabbix/
{{< /alert >}}

Você será direcionado para a URL a seguinte URL:

{{< alert theme="info" dir="ltr" >}}
http://192.168.0.119/zabbix/setup.php
{{< /alert >}}

{{< img src="img/zabbix_config.png" position="center" >}}


Basta seguir os passos de validação. Em seguida você será direcionado para a tela de login. As credenciais padrão são as que seguem abaixo:


{{< alert theme="info" dir="ltr" >}}
Usuário: Admin
Senha: zabbix
{{< /alert >}}


## Até logo!  :heart:
