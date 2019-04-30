---
layout: default
---

# HackInOS: 1

[Download:](https://www.vulnhub.com/entry/hackinos-1,295/)


**Name:** HackInOS: 1

**Date release:** 9 Mar 2019

**Description:**

"O HackinOS é uma máquina vulnerável ao estilo CTF de nível iniciante. Criei essa VM para a comunidade de segurança cibernética da minha universidade e para todos os entusiastas da segurança cibernética. Agradeço a Mehmet Oguz Tozkoparan, Ömer Faruk Senyayla e Tufan Gungor por sua ajuda durante a criação deste laboratório." _Author: Fatih Çelik_

**NOTA:** `localhost` está destinado a estar lá!

### Table of Contents

- [Enumeration](#enumeration)
    - [netdiscover](#netdiscover)
    - [nmap](#nmap)
        - [Web Server](#web-server)
- [Find Vulnerabilities](#find-vulnerabilities)
    - [robots.txt](#robots.txt)
- [Exploitation](#exploitation)
    - [Locate php-reverse-shell.php](#locate-php-reverse-shell)
    - [Listening port](#listening-port)
    - [Upload shell.php](#upload-shell)
    

### Enumeration

#### netdiscover

Inicialmente vamos usar o `netdiscover` para encontrar o IP da nossa VM, porém essa máquina é possível visualizar o IP facilmente através da VM, mas geralmente isso não é  possível. Então, recomendo usar o netdiscover para se acostumar.

```
netdiscover -i eth0 -r 192.168.111.1/24
```

![netdiscover1](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/netdiscover.png)

#### nmap

Como de costume utilizamos o `nmap` em nossa enumeração inicial.

```
nmap -sV -sC -oA nmap/initial 192.168.111.136
```
![nmapinitial](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/nmapinit.png)

Podemos observar 2 portas abertas `8000` e `22`, precisamos escanear todas as portas e verificar se existe um outro serviço em uma porta mais alta.

```
nmap -sV -sT -p1-49152 -oA nmap/allports_tcp 192.168.111.136
```

![allports_tcp](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/allports_tcp.png)

##### Web Server

Possivelmente é um `Wordpress`. O site esta quebrado, vamos ajustar isso...

![wordpress](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/wordpress.png)

Todos os links estavam redirecionando para `localhost`, então editamos nosso hosts `/etc/hosts` e adicionamos o IP da VM  para o localhost: Observe que comentei o padrão do kali.

![localhost1](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/localhost.png)

Depois de realizar os ajustes, o site abre perfeitamente:

![site](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/site.png)

### Find Vulnerabilities

#### robots.txt

Enumerando um pouco mais, encontramos o `robots.txt`. Com um file e um dir setados como `Disallow`, vamos navegar ate eles.

![robots](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/robots.png)

Temos um ótimo lugar para fazer o upload de nosso `payload`.

![upload](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/upload.png)

Depois que for feito o upload, podemos localizá lo no dir `/uploads/`.

![uploads](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/uploads.png)

### Exploitation

#### Locate php-reverse-shell.php

Agora vamos localizar um `shell.php` para facilitar nosso trabalho, poderíamos gerar com o `msfvenom`. Mas automatizar as coisas e ganhar tempo é melhor ainda.

```
locate php-reverse-shell
```

![locate](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/locate.png)

Copie para um diretório de sua escolha:

![copy](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/copy.png)

Edite de acordo com suas configurações.

![edite](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/edite.png)

#### Listening port

```
nc -lvp 1337
```

![listen](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/listen.png)

#### Upload shell.php

Olhando o source do upload.php, encontramos uma dica que sugere ir até o github através do link passado.

![git](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/git.png)

[Vulnerable-Machine---Hint](https://github.com/fatihhcelik/Vulnerable-Machine---Hint)




[back](../index.md)
