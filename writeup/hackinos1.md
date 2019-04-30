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

### Enumeration

#### netdiscover

```
netdiscover -i eth0 -r 192.168.111.1/24
```
Inicialmente vamos usar o `netdiscover` para encontrar o IP da nossa VM, porém essa máquina é possível visualizar o IP facilmente através da VM, mas geralmente isso não é  possível. Então, recomendo usar o netdiscover para se acostumar.

![netdiscover](/assets/images/hackinos1/netdiscover.png)

#### nmap


[back](../index.md)
