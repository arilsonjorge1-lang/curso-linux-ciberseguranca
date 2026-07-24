# Sessão 01 — Introdução ao Linux para Comandos de Segurança e Rede

**Curso:** Reskilling — Linux e Cibersegurança
**Objetivo de Aprendizagem:** OA1 — Analisar
**Formador:** Péricles Borges
**Aluno:** Arilson do Rosário
**Data:** 2026-07-14

## Contexto

Reconhecimento e enumeração de um alvo remoto Windows utilizando comandos core de
rede e scanning do Linux, com o objetivo de identificar portas abertas, serviços em
execução, versões exatas de software, e o sistema operativo do alvo.

**Alvo remoto:** `10.129.169.121` (`WIN-SCAN`, Windows Server)

---

## Metodologia

1. Inspecionar a configuração de rede local (`ip a`)
2. Listar sockets locais em escuta (`ss -tuln`)
3. Correr um scan completo de portas/serviços contra o alvo remoto com `nmap`
4. Analisar o output do scan para identificar portas abertas, serviços e versões
5. Determinar o sistema operativo do alvo através de fingerprinting de serviços
6. Documentar os achados e propor o próximo passo de investigação

---

## Resumo dos Resultados

### Portas Abertas Identificadas

**Total: 5 portas abertas** (de 1000 portas escaneadas; as restantes 995 estão filtradas)

| Porta | Protocolo | Estado | Serviço | Versão Detetada |
|---|---|---|---|---|
| 21 | TCP | open | ftp | FileZilla ftpd |
| 53 | TCP | open | domain | Simple DNS Plus |
| 80 | TCP | open | http | Microsoft IIS httpd 10.0 |
| 135 | TCP | open | msrpc | Microsoft Windows RPC |
| 3389 | TCP | open | ms-wbt-server | Microsoft Terminal Services (RDP) |

**Sistema operativo do alvo:** Windows (confirmado via `rdp-ntlm-info`)

---

## Ambiente Local — Output Completo dos Comandos

### `ip a`
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host noprefixroute
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc fq_codel state UP group default qlen 1000
    link/ether 12:6d:36:ce:48:f4 brd ff:ff:ff:ff:ff:ff
    inet 172.30.1.2/24 brd 172.30.1.255 scope global dynamic noprefixroute enp1s0
    inet6 fe80::4b79:eed3:82b5:ca70/64 scope link
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1454 qdisc noqueue state DOWN group default
    link/ether 0a:ff:25:b6:b9:16 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
```

### `ss -tuln`
```
Netid  State    Recv-Q  Send-Q  Local Address:Port          Peer Address:Port
udp    UNCONN   0       0       127.0.0.54:53               0.0.0.0:*
udp    UNCONN   0       0       127.0.0.53%lo:53             0.0.0.0:*
udp    UNCONN   0       0       172.30.1.2:68                0.0.0.0:*
udp    UNCONN   0       0       172.30.1.2%enp1s0:68         0.0.0.0:*
udp    UNCONN   0       0       [fe80::8b84:a7da:9:71b7]%enp1s0:546  [::]:*
tcp    LISTEN   0       4096    0.0.0.0:22                   0.0.0.0:*
tcp    LISTEN   0       4096    127.0.0.54:53                0.0.0.0:*
tcp    LISTEN   0       4096    127.0.0.1:33331               0.0.0.0:*
tcp    LISTEN   0       128     0.0.0.0:40200                 0.0.0.0:*
tcp    LISTEN   0       511     0.0.0.0:40205                 0.0.0.0:*
tcp    LISTEN   0       4096    127.0.0.53%lo:53              0.0.0.0:*
tcp    LISTEN   0       4096    [::]:22                       [::]:*
tcp    LISTEN   0       4096    *:40300                       *:*
tcp    LISTEN   0       4096    *:40305                       *:*
```

### Output Completo do Scan Nmap
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-07-14 10:15 UTC
Nmap scan report for ip-10-129-169-121.eu-west-3.compute.internal (10.129.169.121)
Host is up (0.00050s latency).
Not shown: 995 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           FileZilla ftpd
| ftp-syst:
|_  SYST: UNIX emulated by FileZilla
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods:
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
135/tcp  open  msrpc         Microsoft Windows RPC
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info:
|   Target_Name: WIN-SCAN
|   NetBIOS_Domain_Name: WIN-SCAN
|   NetBIOS_Computer_Name: WIN-SCAN
|   DNS_Domain_Name: win-scan
|   DNS_Computer_Name: win-scan
|   Product_Version: 10.0.17763
|_  System_Time: 2026-07-14T10:15:21+00:00
MAC Address: 0A:28:0D:1D:EE:D1 (Unknown)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Nmap done: 1 IP address (1 host up) scanned in 46.62 seconds
```

---

## Análise e Observações

O alvo é um Windows Server com uma superfície de ataque considerável:

- **FTP (21/tcp) permite login anónimo** — uma configuração incorreta grave. O script
  `ftp-anon` confirmou acesso com o código FTP 230 (login bem-sucedido), significando
  que qualquer utilizador não autenticado pode tentar navegar ou obter ficheiros do
  servidor.
- **IIS (80/tcp) expõe o método HTTP TRACE** como potencialmente arriscado, o que pode
  ser explorado em certos ataques de cross-site tracing (XST) para contornar
  proteções de cookies HttpOnly.
- **RPC (135/tcp) e RDP (3389/tcp) estão acessíveis externamente**, ambos alvos
  comuns para ataques de força bruta e movimento lateral em ambientes Windows.
- A **versão exata do Windows (build 10.0.17763)** foi revelada através do
  fingerprinting NTLM via RDP, o que reduz a margem de pesquisa para um atacante à
  procura de vulnerabilidades conhecidas para essa build específica.

## Próximo Passo Recomendado

Testar o acesso FTP anónimo para identificar eventuais ficheiros expostos, já que o
scan confirmou que o login anónimo é permitido na porta 21.

---

## Resumo de Indicadores

| Achado | Nível de Risco | Recomendação |
|---|---|---|
| Login FTP anónimo permitido | Alto | Desativar acesso anónimo; exigir autenticação |
| Método HTTP TRACE ativo (IIS) | Médio | Desativar o método TRACE na configuração do IIS |
| RPC (135) exposto externamente | Médio | Restringir via firewall apenas a hosts confiáveis |
| RDP (3389) exposto externamente | Alto | Restringir via firewall/VPN; aplicar MFA e políticas de bloqueio de conta |
| Versão do Windows divulgada | Baixo–Médio | Restringir fingerprinting NTLM/RDP onde possível; manter o sistema atualizado |

---

*Relatório elaborado no âmbito da Sessão 1 do percurso Reskilling — Linux e Cibersegurança.*
