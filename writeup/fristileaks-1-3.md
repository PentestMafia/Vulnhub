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
    - [nmap](#nmap)

## Information Gathering

### Route Analysis

#### Netdiscover

Iniciamos obtendo algumas informações necessárias para podermos continuar, ou seja, precisamos fazer o levantamento de informações. Como não foi passado nosso `target`, precisamos descobrir qual `IP` esta associado a máquina vuln.

`netdiscover -i eth0 -r 10.0.1.0/24`

![netdiscover1](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_09-05.png)

### Nmap

Depois de obter nosso alvo, podemos usar o `nmap` e iniciar a enumeração e assim pegar mais informações. Por exemplo: Portas abertas, versões de serviços e etc.

`nmap -sV -sC -oN nmap/initial 10.0.1.110`

![nmap1](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_09-35.png)

