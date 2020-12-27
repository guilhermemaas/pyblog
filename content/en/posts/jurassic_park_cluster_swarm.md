---
title: "50 containers Nginx com Docker Swarm"
date: 2020-12-26T18:50:58-03:00
description: Neste breve tutorial iremos subir um cluster Swarm com alguns nodes e escalar um serviço com Nginx para 50 containers
draft: false
hideToc: true
enableToc: true
enableTocContent: false
tags:
- docker
- swarm
- nginx
categories:
- docker
- containers
image: images/icones/swarm.jpg
---

{{< img src="img/jurassic_park_islands.png" position="center" >}}
## 

Neste breve tutorial irei demonstrar como:

- Iniciar um cluster Swarm.
- Adicionar hosts ao cluster.
- Criar uma máquina virtual em uma linha de comando com o driver para Virtual Box e Docker Machine.
- Criar um serviço escalando 50 containers de Nginx distribuídos entre os nós do cluster.

Requisitos: Já ter dado uma brincada com Docker, ou não.


{{< notice info "This is a success type of notice" >}}
Irei subir seis nodes,nós no cluster:
Node 1 - isla-nublar - Node principal (Manager).
Node 2 - isla-sorna.
Node 3 - isla-muerta.
Node 4 - isla-pena.
Node 5 - isla-tacano.
Node 6 - isla-mataceros.
{{< /notice >}}

Obs.: Sim, eu dei o nome das ilhas do Jurassic Park para os hosts. Eu gosto de nomear as coisas que vou fazendo para estudar com nomes de ficção. Então se acostume a ver muita coisa sobre Jurassic Park ou Star Wars por aqui.


## :whale: Iniciando o Cluster:

Para montar o ambiente eu vou utilizar alguns hosts virtuais utilizando o VirtualBox da Oracle. Vai do seu gosto nesse caso. O interessante é que o Docker tem por padrão um driver para criar máquinas virtuais para nos servir como servidores de containers. "Docker Machines". Abaixo irei demonstrar. Mas inicialmente, subi o node 1 até o 3 em três máquinas com sistema operacional CentOS 8. E os outros três nodes irei subir utilizando o docker-machine.

Obs.: Para facilitar, você pode subir apenas o Node 1 que trataremos como "Principal", ou seja, o "Manager" do cluster, no CentOS e o restante subir de forma "Automágica" com docker-machine.

Na Máquina que representa o Node 1, temos que instalar o docker. Para isso, siga os passos deste tutorial da documentação oficial:
https://docs.docker.com/engine/install/centos/

{{< codes bash >}}
  {{< code >}}
  ```bash
  #Para iniciar o cluster, basta executar o seguinte comando:
  docker swarm init
  ```
  {{< /code >}}
{{< /codes >}}

Por padrão a mensagem abaixo é exibida, contendo o comando com o token que iremos utilizar para introduzir novas máquinas ao cluster:

{{< notice info "This is a success type of notice" >}}
[root@isla-nublar ~]# docker swarm init
Swarm initialized: current node (teafdmhkglu777eby437b15mb) is now a manager.
                                                                                                                                                           
To add a worker to this swarm, run the following command:
                                                                                                                                                           
docker swarm join --token SWMTKN-1-1xbfaag9w44yghanzigc8vdw2c7we2gj7v032jcyl3rtqspcve-adzhmq6y982a3hwmenkfv50ef 192.168.0.110:2377
                                                                                                                                                           
To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
{{< /notice >}}

Note que esse node é automaticamente definido como Manager do cluster.


## :whale: Criando Docker Machines para introduzir no cluster:

Para instalação do binário do Docker Machine, execute o script abaixo:
{{< codes bash >}}
  {{< code >}}
  ```bash
  #Instalação do Docker Machine:
  #Obs.: Busque no repositório git pela versão mais nova.
  base=https://github.com/docker/machine/releases/download/v0.16.0 &&
    curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine &&
    sudo mv /tmp/docker-machine /usr/local/bin/docker-machine &&
    chmod +x /usr/local/bin/docker-machine
  ```
  {{< /code >}}
{{< /codes >}}

Como citado lá em cima, eu já tenho 3 máquinas com CentOS, que são dos nodes 1 até 3. Então nesse caso vou criar 3 novas máquinas virtuais:
{{< codes bash >}}
  {{< code >}}
  ```bash
  docker-machine create --driver virtualbox isla-pena
  docker-machine create --driver virtualbox isla-tacano
  docker-machine create --driver virtualbox isla-mataceros
  ```
  {{< /code >}}
{{< /codes >}}

Abaixo podemos ver que as máquinas virtuais já estão rodando através do Virtual Box:

{{< img src="img/docker_machines.png" position="center" >}}
## 

Para se conectar via SSH a estas máquinas, basta executar o seguinte comando:
{{< codes bash >}}
  {{< code >}}
  ```bash
  #Lista as docker-machines criadas:
  docker-machine ls

  #Conecta via SSH:
  docker-machine ssh isla-pena

  #Para ver os detalhes de cada docker-machine:
  docker-machine inspect isla-pena
  ```
  {{< /code >}}
{{< /codes >}}

Abaixo estou conectado via SSH nos seis servidores virtuais que serão os nós/nodes do cluster Swarm(3x CentOS 8 e 3x Docker Machine com TinyCore Linux):

{{< img src="img/docker_swarm_join.png" position="center" >}}
## 

Basta executar o comando "docker swarm join" para introduzir os nós no cluster:
{{< codes bash >}}
  {{< code >}}
  ```bash
  docker swarm join --token SWMTKN-1-1xbfaag9w44yghanzigc8vdw2c7we2gj7v032jcyl3rtqspcve-adzhmq6y982a3hwmenkfv50ef 192.168.0.110:2377
  ```
  {{< /code >}}
{{< /codes >}}

Após isso podemos listar todos os nós do cluster com o comando "docker node ls":

{{< img src="img/docker_node_ls.png" position="center" >}}
## 

## :while: Criando um serviço no cluster:

Para realizar o Deploy de uma imagem Docker precisamos criar um serviço. Frequentemente o serviço é uma imagem de um microserviço em um ambiente distribuido. Nesse caso vamos criar um serviço utilizando a Imagem do Nginx.

Lembrando que podemos definir vários parâmetros para os containers que irão subir, tais como: Quantidade de réplicas, Overlay de Network, limites de CPU e memória, entre outros. 

Quando você faz deploy de um serviço para o Swarm, o Swarm Manager aceita as definições do seu serviço e então levanta réplicas, ou seja, containers nos nós do cluster.

Abaixo temos uma imagem da documentação oficial demonstrando o funcionamento de um serviço no cluster:

{{< img src="img/services-diagram.png" position="center" >}}
## 

Mãos a obra:
{{< codes bash >}}
  {{< code >}}
  ```bash
  #Vamos criar uma rede, e subir todos os containers nessa rede:
  #A rede precisa ser do tipo Overlay para o Swarm. Assim todos os containers podem conversar entre si.
  docker network create -d overlay jurassic-park

  #Criando o service:
  docker service create --name nginx-jurassic-park --replicas 6 --network jurassic-park -p 8181:80 nginx

  #Para verificar o que está rodando nos nodes:
  docker service ls
  ```
  {{< /code >}}
{{< /codes >}}

Veja que os containers foram divididos entre os nós do cluster igualmente. Note também que as requisições irão ser balanceadas automaticamente entre os nós e containers. Ou seja, quando uma requisição bater na porta 8181 de qualquer nó, irá cair de forma equilibrada em um dos nós na porta 80. É um balanceamento de carga no melhor estilo "Round-Robin", RR.

Note também que quando criamos o serviço o docker automaticamente baixou a imagem em cada nó:

{{< img src="img/service_create.png" position="center" >}}
## 

Para verificar a distribuição dos containers nos nós basta executar o seguinte comando:
{{< codes bash >}}
  {{< code >}}
  ```bash
  docker service ps nginx-jurassic-park
  ```
  {{< /code >}}
{{< /codes >}}


## :whale: Agora vamos a parte divertida! Escalar para 50 containers Nginx:

Primeiro vamos testar uma requisição em qualquer IP do cluster na porta 8181 para verificar se o Nginx responde:

Para descobrir o IP de um dos nós execute o comando abaixo:
{{< codes bash >}}
  {{< code >}}
  ```bash
  docker node inspect isla-tacano --pretty | grep Address
  ```
  {{< /code >}}
{{< /codes >}}

Dica: Acompanhe o log do serviço(O log em tempo real de todos os containers):
{{< codes bash >}}
  {{< code >}}
  ```bash
  
  ```
  {{< /code >}}
{{< /codes >}}

Chamamos o endereço no Firefox:

{{< img src="img/request_nginx_firefox.png" position="center" >}}
## 

Agora vamos ao que interessa!

Para escalar para 50 containers basta executar o comando abaixo:
{{< codes bash >}}
  {{< code >}}
  ```bash
  docker service scale webserver=50
  ```
  {{< /code >}}
{{< /codes >}}

Execute o comando "docker service ps nginx-jurassic-park para validar, teremos 50 containers rodando:

{{< img src="img/50_nginx.png" position="center" >}}

## Até logo!  :heart:
