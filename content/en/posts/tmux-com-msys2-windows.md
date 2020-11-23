---
title: "Tmux no Windows com Msys2"
date: 2020-11-22T19:39:21-03:00
description: Rodando o multiplexador de terminais Tmux no Windows com Msys2
draft: false
hideToc: true
enableToc: true
enableTocContent: false
tags:
- terminal
- linux
- tmux
- msys2
categories:
- terminal
- linux
image: images/icones/Msys_icone.jpg
---

{{< img src="img/Misaya.png" position="center" >}}

## 

Muitas vezes estamos utilizando Windows no trabalhou ou em casa e sentimos falta de utilizar o Tmux para criar várias sessões de terminal, além de claro, do próprio bash e suas vantagens. Neste breve tutorial irei mostrar como instalar o Tmux através do Msys2 e seus principais atalhos.

Tmux: É um multiplexador de terminal de código aberto para sistemas operacionais Unix-like. Ele permite criar multiplas sessões de terminal para acessar simultaneamente em uma única janela. Ótimo para rodar vários programas de linha de comando ao mesmo tempo.

Msys2: Baseado no Cygwin e Mingw-w64, nos permite rodar aplicações do projeto GNU, Linux ao mesmo tempo que interagimos com o própio Windows. No geral, nos permite rodar aplicações Linux no Windows, assim como o WSL(Windows Subsystem for Linux). Seus pacotes são distribuídos através do Pacman como no Arch Linux.


## :penguin: Instalação e Configuração

Baixe a versão mais atual do Msys2 disponível em: https://repo.msys2.org/distrib/x86_64/

A instalação é next, next, finish. Então não há segredo. Mais detalhes em: https://www.msys2.org/#installation


## Baixe o Tmux

Para baixar o Tmux basta rodar o seguinte comando e aguardar:

{{< codes bash >}}
  {{< code >}}
  ```bash
  pacman -S tmux
  ```
  {{< /code >}}
{{< /codes >}}

Podemos baixar outros pacotes disponíveis através do pacman. Por exemplo, o openssh para abrirmos conexões com diversos servidores:
{{< img src="img/instalando_pacotes.png" position="center" >}}

Outros pacotes você pode encontrar aqui: https://packages.msys2.org/base

Mais detalhes sobre o pacman:

{{< codes bash >}}
  {{< code >}}
  ```bash
  pacman --help
  ```
  {{< /code >}}
{{< /codes >}}


## Alguns atalhos interessantes do Tmux

| Atalho                    | Descrição                                                  |
|---------------------------|------------------------------------------------------------|
| ctrl+b                    | Inicia um comando                                          |
| ctrl+b+c                  | Cria uma nova janela                                       |
| ctrl+b+,                  | Renomear uma janela                                        |
| ctrl+b+p ou n             | Navega para uma próxima janela ou anterior                 |
| ctrl+b+l                  | Navega para a última janela em que você estava             |
| ctrl+b+número da janela   | Navega para a jánela de número x                           |
| ctrl+b+w                  | Liste de janelas para selecionar                           |
| ctrl+b+%                  | Cria um painél vertical                                    |
| ctrl+b+"                  | Cria um painel horizontal                                  |
| ctrl+b+setas              | Utilize Ctrl+b e as setas para navegar entre os painéis    |
| ctrl pressionado+b+setas  | Para aumentar ou diminuir o tamanho do painel              |
| ctrl+b+o                  | Rotaciona entre as janelas                                 |
| ctrl+b+x                  | Permite fechar o painel                                    |
| ctrl+b+d                  | Sair de uma sessão sem fechar, irá permanecer em BG        |


## Alguns comandos para gerenciamento do Tmux

{{< codes bash >}}
  {{< code >}}
  ```bash
  tmux ls
  #Mostra as sessões criadas

  tmux new -s nome_sessão
  #Cria uma nova sessão

  tmux kill-session -t 0
  #Mata uma sessão, 0 = ID da sessão(comando ls)

  tmux attach -t 0
  #Entra em uma sessão, 0 = ID da sessão(comando ls)
  ```
  {{< /code >}}
{{< /codes >}}


## Exemplo de algumas sessões criadas com Msys2 e Tmux

{{< img src="img/tmux4.jpg" position="center" >}}


## Até logo!  :heart:
