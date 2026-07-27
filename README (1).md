# Reskilling — Linux e Cibersegurança
## Portfólio Técnico Completo — Sessões 1 a 5

**Aluno:** Arilson do Rosário
**Formador:** Péricles Borges
**Programa:** Skodji Digital — Ministério da Economia Digital, Governo de Cabo Verde

---

## Índice de Sessões

| # | Sessão | Objetivo |
|---|---|---|
| 1 | [Introdução ao Linux para Comandos de Segurança e Rede](#sessão-01--introdução-ao-linux-para-comandos-de-segurança-e-rede) | OA1 — Analisar |
| 2 | [Auditoria de Sistemas Linux e Análise Avançada de Logs](#sessão-02--auditoria-de-sistemas-linux-e-análise-avançada-de-logs) | OA2 — Avaliar |
| 3 | [Hardening de Redes Linux e Configuração de Firewalls](#sessão-03--hardening-de-redes-linux-e-configuração-de-firewalls) | OA3 — Aplicar |
| 4 | [Gestão Segura de Acessos Remotos SSH em Linux](#sessão-04--gestão-segura-de-acessos-remotos-ssh-em-linux) | OA4 — Aplicar |
| 5 | [Análise de Vulnerabilidades em Linux e Ferramentas de Auditoria](#sessão-05--análise-de-vulnerabilidades-em-linux-e-ferramentas-de-auditoria) | OA5 — Criar |

---
---

# Sessão 01 — Introdução ao Linux para Comandos de Segurança e Rede

**Objetivo de Aprendizagem:** OA1 — Analisar

## Contexto

Reconhecimento e enumeração de um alvo remoto Windows utilizando comandos core de
rede e scanning do Linux, com o objetivo de identificar portas abertas, serviços em
execução, versões exatas de software, e o sistema operativo do alvo.

| | |
|---|---|
| **Alvo remoto** | 10.129.169.121 (WIN-SCAN, Windows Server) |
| **Data** | 2026-07-14 |

## Metodologia

1. Inspecionar a configuração de rede local (`ip a`)
2. Listar sockets locais em escuta (`ss -tuln`)
3. Correr um scan completo de portas/serviços contra o alvo remoto com `nmap`
4. Analisar o output do scan para identificar portas abertas, serviços e versões
5. Determinar o sistema operativo do alvo através de fingerprinting de serviços
6. Documentar os achados e propor o próximo passo de investigação

## Portas Abertas Identificadas

**Total: 5 portas abertas** (de 1000 escaneadas; as restantes 995 estão filtradas)

| Porta | Protocolo | Estado | Serviço | Versão Detetada |
|---|---|---|---|---|
| 21 | TCP | open | ftp | FileZilla ftpd |
| 53 | TCP | open | domain | Simple DNS Plus |
| 80 | TCP | open | http | Microsoft IIS httpd 10.0 |
| 135 | TCP | open | msrpc | Microsoft Windows RPC |
| 3389 | TCP | open | ms-wbt-server | Microsoft Terminal Services (RDP) |

**Sistema operativo do alvo:** Windows (confirmado via `rdp-ntlm-info`)

## Output Completo do Scan Nmap

```
Nmap scan report for 10.129.169.121
Host is up (0.00050s latency).
Not shown: 995 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           FileZilla ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: Potentially risky methods: TRACE
135/tcp  open  msrpc         Microsoft Windows RPC
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info:
|   Target_Name: WIN-SCAN
|   Product_Version: 10.0.17763
MAC Address: 0A:28:0D:1D:EE:D1 (Unknown)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

## Análise e Observações

> 🔴 **Risco Alto — FTP anónimo:** o script `ftp-anon` confirmou acesso com o código
> FTP 230 (login bem-sucedido) — qualquer utilizador não autenticado pode navegar ou
> obter ficheiros do servidor.

> 🟠 **Atenção — HTTP TRACE ativo (IIS):** pode ser explorado em ataques de
> cross-site tracing (XST) para contornar proteções de cookies HttpOnly.

> 🟠 **Atenção — RPC/RDP expostos externamente:** ambos alvos comuns para força
> bruta e movimento lateral em ambientes Windows.

> 🔵 **Info:** a versão exata do Windows (build 10.0.17763) foi revelada via
> fingerprinting NTLM/RDP, reduzindo a margem de pesquisa de vulnerabilidades para
> um atacante.

## Próximo Passo Recomendado

Testar o acesso FTP anónimo para identificar eventuais ficheiros expostos.

## Resumo de Indicadores

| Achado | Nível de Risco | Recomendação |
|---|---|---|
| Login FTP anónimo permitido | Alto | Desativar acesso anónimo; exigir autenticação |
| Método HTTP TRACE ativo (IIS) | Médio | Desativar o método TRACE na configuração do IIS |
| RPC (135) exposto externamente | Médio | Restringir via firewall apenas a hosts confiáveis |
| RDP (3389) exposto externamente | Alto | Restringir via firewall/VPN; aplicar MFA e bloqueio de conta |
| Versão do Windows divulgada | Baixo–Médio | Restringir fingerprinting NTLM/RDP; manter sistema atualizado |

---
---

# Sessão 02 — Auditoria de Sistemas Linux e Análise Avançada de Logs

**Objetivo de Aprendizagem:** OA2 — Avaliar
**Ambiente prático:** TryHackMe (Intro to Logs + Linux Server Forensics)

## Contexto

Um servidor da infraestrutura foi alvo de conexões anómalas. Este relatório documenta
a investigação forense realizada aos logs de autenticação (`/var/log/auth.log`) para
determinar a origem, o sucesso e o impacto do ataque de força bruta SSH.

## Metodologia

- Navegação até ao diretório de logs (`cd /var/log/`)
- Isolamento das tentativas falhadas de login (`grep "Failed password"`)
- Contagem de tentativas por IP (`awk` + `sort` + `uniq -c`)
- Confirmação de sucesso do ataque (`grep -E "Accepted password|Accepted publickey"`)

## Resultados

| Item | Valor |
|---|---|
| **IP do Atacante** | `10.129.151.161` |
| **Timestamp do Comprometimento** | `Jul 17 12:02:35` |
| **Utilizador Afetado** | `fred` |

## Linha Temporal do Ataque

| Timestamp | Evento | IP |
|---|---|---|
| Jul 17 11:59:56 | Failed password (tentativa 1) | 10.129.151.161 |
| Jul 17 12:00:13 | Failed password (tentativa 2) | 10.129.151.161 |
| Jul 17 12:01:23 | Failed password (tentativa 3) | 10.129.151.161 |
| Jul 17 12:02:10 | Failed password (tentativa 4) | 10.129.151.161 |
| Jul 17 12:02:35 | **Accepted password — COMPROMETIMENTO** | 10.129.151.161 |

> 🔴 **Risco Alto:** o comprometimento ocorreu após apenas 4 tentativas falhadas,
> num intervalo de cerca de 2 minutos e 39 segundos — sugere password fraca ou
> previsível.

## Evidências (Excertos dos Logs)

**Tentativas falhadas**
```
$ grep "Failed password" auth.log
Jul 17 11:59:56 ip-10-129-136-117 sshd[1308]: Failed password for fred from 10.129.151.161 port 55070 ssh2
Jul 17 12:00:13 ip-10-129-136-117 sshd[1308]: Failed password for fred from 10.129.151.161 port 55070 ssh2
Jul 17 12:01:23 ip-10-129-136-117 sshd[1316]: Failed password for fred from 10.129.151.161 port 45300 ssh2
Jul 17 12:02:10 ip-10-129-136-117 sshd[1316]: Failed password for fred from 10.129.151.161 port 45300 ssh2
```

**Ranking de IPs por número de tentativas**
```
$ grep "Failed password" auth.log | awk '{print $11}' | sort | uniq -c | sort -nr
      4 10.129.151.161
```

**Login com sucesso**
```
$ grep -E "Accepted password|Accepted publickey" auth.log
Jul 17 12:02:35 ip-10-129-136-117 sshd[1316]: Accepted password for fred from 10.129.151.161 port 45300 ssh2
```

## Indicador de Comprometimento (IoC)

| Tipo | Valor |
|---|---|
| IP suspeito | 10.129.151.161 |
| Utilizador visado | fred |
| Timestamp do sucesso | Jul 17 12:02:35 |
| Método de autenticação usado | password |

## Recomendações

> 🟢 **Boa prática prioritária:** desativar autenticação por password, permitir
> apenas chaves SSH (`PasswordAuthentication no`)

- Configurar Fail2ban para bloqueio automático de IPs após tentativas repetidas
- Restringir acesso SSH via firewall/VPN de gestão
- Ativar MFA para autenticação remota
- Monitorização contínua dos logs de autenticação

---
---

# Sessão 03 — Hardening de Redes Linux e Configuração de Firewalls

**Objetivo de Aprendizagem:** OA3 — Aplicar
**Ambiente prático:** KillerCoda Ubuntu Playground

## Contexto

Configuração de uma política defensiva estrita para impedir acessos não autorizados
a serviços críticos do servidor, combinando UFW e iptables, seguindo o princípio
"bloquear tudo por defeito, abrir apenas o essencial".

## Metodologia

1. Verificação do estado inicial do UFW (`sudo ufw status`)
2. Definição das políticas por defeito — bloquear entrada, permitir saída
3. Criação de regra específica para permitir SSH (porta 22/tcp) e ativação da firewall
4. Simulação de bloqueio de um IP malicioso fictício via `iptables` (chain INPUT)
5. Persistência do estado do iptables em `/etc/iptables/rules.v4`

## Política Aplicada

| Parâmetro | Valor |
|---|---|
| Default incoming | **DENY** |
| Default outgoing | **ALLOW** |
| Default routed | **DENY** |
| Porta aberta | 22/tcp (SSH) |
| IP bloqueado (simulação) | 203.0.113.50 |
| Método de bloqueio | DROP (silencioso) |

## Regras UFW Ativas

```
$ sudo ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), deny (routed)

To                         Action      From
22/tcp                     ALLOW IN    Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
```

## Listagem iptables (excerto relevante)

```
Chain INPUT (policy DROP 40 packets, 2400 bytes)
    0     0 DROP     all  --  any  any  203.0.113.50  anywhere

Chain ufw-user-input (1 references)
    0     0 ACCEPT  tcp  --  any  any  anywhere  anywhere  tcp dpt:ssh
```

> 🔴 `-A INPUT -s 203.0.113.50/32 -j DROP` — regra que confirma o bloqueio
> silencioso do IP malicioso simulado.

## Persistência das Regras

```
$ sudo mkdir -p /etc/iptables
$ sudo iptables-save | sudo tee /etc/iptables/rules.v4
-A INPUT -s 203.0.113.50/32 -j DROP
COMMIT
```

## Justificação da Política Aplicada

> 🔵 **Info:** "deny incoming" por defeito — qualquer serviço não explicitamente
> autorizado fica automaticamente inacessível, aplicando o princípio do menor
> privilégio ao nível da rede.

> 🔵 **Info:** DROP em vez de REJECT — descarta o tráfego silenciosamente,
> dificultando ferramentas de reconhecimento como o `nmap`.

> 🟠 **Regra de ouro:** a regra de SSH foi sempre criada **antes** de ativar a
> política `default deny incoming` — evita perder o acesso remoto à máquina
> durante a configuração.

---
---

# Sessão 04 — Gestão Segura de Acessos Remotos SSH em Linux

**Objetivo de Aprendizagem:** OA4 — Aplicar
**Ambiente prático:** KillerCoda Ubuntu Playground

## Contexto

Proteger o canal de gestão remota do servidor Ubuntu, eliminando a autenticação
tradicional por password e migrando para autenticação criptográfica (chave Ed25519),
complementada com bloqueio de login root direto e mudança da porta padrão do SSH.

## Metodologia

1. Criação de um utilizador de teste (`testuser`) e preparação do diretório `.ssh`
2. Geração de um par de chaves Ed25519 (`ssh-keygen -t ed25519`)
3. Transferência da chave pública para o servidor (`ssh-copy-id`)
4. Teste do login por chave **antes** de qualquer alteração ao `sshd_config`
5. Edição do `sshd_config`: bloqueio de root, desativação de password, mudança de porta
6. Validação obrigatória da sintaxe (`sudo sshd -t`) antes do restart
7. Reinício do serviço SSH e resolução de um conflito com `ssh.socket`
8. Teste final de login por chave, na nova porta, confirmando o sucesso

## Resultados

| Item | Valor |
|---|---|
| Utilizador de teste | testuser |
| Tipo de chave | ed25519 |
| Fingerprint | SHA256:W76wgnpn1sCqKR2KYXbzbZ4G2fUAaddAy1uyFWveIts |

## Linhas Modificadas do sshd_config

```
PermitRootLogin no
PasswordAuthentication no
Port 2222
```

## Cópia Limpa do sshd_config Corrigido

```
PermitRootLogin no
Subsystem sftp internal-sftp
PasswordAuthentication no
Port 2222
```

## Validação da Configuração

```
$ sudo sshd -t
(sem output — sintaxe válida)
```

## Reinício do Serviço — Nota Técnica Importante

> 🟠 **Atenção:** erro encontrado — `Unit sshd.service could not be found.` No
> Ubuntu/Debian o serviço chama-se **ssh**, não sshd. Após corrigir, o serviço
> continuou a escutar na porta 22 devido à **ativação por socket do systemd**
> (`ssh.socket`), que sobrepõe-se ao `Port` definido no `sshd_config`.

```bash
sudo systemctl disable --now ssh.socket
sudo systemctl enable --now ssh.service
sudo systemctl restart ssh.service
```

```
sshd[2109]: Server listening on 0.0.0.0 port 2222.
sshd[2109]: Server listening on :: port 2222.
```

## Evidência de Login Bem-Sucedido via Chave

```
$ ssh -i ~/.ssh/id_ed25519 -p 2222 testuser@localhost
Last login: Wed Jul 22 12:28:35 2026 from 127.0.0.1
testuser@ubuntu:~$
```

- ✅ Via chave privada, sem qualquer pedido de password
- ✅ Na porta 2222 (porta padrão desativada)
- ✅ Como utilizador normal (login root direto bloqueado)

## Justificação das Escolhas

> 🔵 **PasswordAuthentication no:** elimina por completo o vetor de força bruta —
> sem password para adivinhar, não há ataque possível.

> 🔵 **PermitRootLogin no:** obriga escalada explícita via `sudo`, registada em
> log — melhora a rastreabilidade.

> 🔵 **Port 2222:** reduz drasticamente o ruído de scanners automáticos e bots.

---
---

# Sessão 05 — Análise de Vulnerabilidades em Linux e Ferramentas de Auditoria

**Objetivo de Aprendizagem:** OA5 — Criar
**Ambiente prático:** KillerCoda Ubuntu Playground

## Contexto

Execução de um exame de auditoria técnica automatizada com o **Lynis** para
identificar desvios de conformidade em relação aos standards de segurança
recomendados (CIS Benchmarks), seguida da seleção e correção proposta para duas
vulnerabilidades críticas nas áreas de Authentication e Filesystem.

## Metodologia

1. Instalação do Lynis (`sudo apt install lynis -y`)
2. Execução da auditoria completa (`sudo lynis audit system`)
3. Registo do Hardening Score inicial, contagem de Warnings e Suggestions
4. Seleção de 2 Suggestions críticas nas áreas de Authentication e Filesystem
5. Pesquisa da correção recomendada na base de dados oficial da Cisofy

## Resultados da Auditoria Inicial

| Métrica | Valor |
|---|---|
| **Hardening Index** | **60 / 100** |
| Tests performed | 262 |
| Warnings | 1 |
| Suggestions | 50 |

> 🟠 **Warning encontrado:** "Found one or more vulnerable packages." [PKGS-7392]

> 🟢 **Componente confirmado:** Firewall [V] encontrada e ativa — validando o
> trabalho de hardening da Sessão 3.

## Vulnerabilidade 1 — [AUTH-9262]

**Categoria:** Authentication — Ausência de módulo PAM para testar força de passwords

```
* Install a PAM module for password strength testing like
  pam_cracklib or pam_passwdqc [AUTH-9262]
  https://cisofy.com/lynis/controls/AUTH-9262/
```

> 🟠 **Risco:** sem um módulo PAM de verificação de qualidade, o sistema aceita
> passwords fracas ou triviais na criação/alteração de contas.

**Correção recomendada (Cisofy):**
```bash
sudo apt install libpam-passwdqc -y
```

## Vulnerabilidade 2 — [FILE-7524]

**Categoria:** Filesystem — Permissões de ficheiros a restringir

```
* Consider restricting file permissions [FILE-7524]
  - Solution : Use chmod to change file permissions
  https://cisofy.com/lynis/controls/FILE-7524/
```

Ficheiros/diretórios sinalizados: `/etc/crontab`, `/etc/ssh/sshd_config`,
`/etc/cron.d`, `/etc/cron.daily`, `/etc/cron.hourly`, `/etc/cron.weekly`,
`/etc/cron.monthly`

> 🟠 **Risco:** permissões demasiado abertas podem levar a escalada de
> privilégios ou persistência de um atacante através de tarefas cron maliciosas.

**Correção recomendada (Cisofy):**
```bash
sudo chmod 600 /etc/ssh/sshd_config
sudo chmod 600 /etc/crontab
sudo chmod 700 /etc/cron.d /etc/cron.daily /etc/cron.hourly \
  /etc/cron.weekly /etc/cron.monthly
```

## Ligação com Sessões Anteriores

> 🔵 O Lynis confirmou de forma independente que a firewall está ativa,
> validando o hardening da Sessão 3. O único Warning encontrado não está
> relacionado com SSH ou firewall, sugerindo que as configurações das Sessões 3
> e 4 se mantiveram corretas.

## Próximos Passos Recomendados

1. Aplicar as duas correções propostas acima
2. Correr novamente `sudo lynis audit system --quick`
3. Comparar o novo Hardening Index com o valor inicial (60/100)

---

*Portfólio elaborado no âmbito do percurso Reskilling — Linux e Cibersegurança — Skodji Digital.*
