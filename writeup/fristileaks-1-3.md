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


## Information Gathering

### Route Analysis

#### Netdiscover

`netdiscover -i eth0 -r 10.0.1.0/24`

![netdiscover1](https://raw.githubusercontent.com/PentestMafia/Vulnhub/master/assets/images/fristileaks-1-3/2019-05-18_09-05.png)