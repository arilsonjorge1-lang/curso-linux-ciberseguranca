# Reskilling — Linux e Cibersegurança

Portfólio técnico do percurso **Reskilling — Linux e Cibersegurança**, desenvolvido no
âmbito do programa Skodji Digital (Ministério da Economia Digital / Governo de Cabo
Verde, em parceria com o World Bank Group).

Cada sessão combina teoria com prática guiada em ambientes reais (TryHackMe,
KillerCoda), culminando num mini-relatório técnico documentado e reprodutível.

---

## Índice de Sessões

| # | Sessão | Objetivo (OA) | Ferramentas Principais |
|---|---|---|---|
| 1 | [Introdução ao Linux para Comandos de Segurança e Rede](./sessao-01/README.md) | OA1 — Analisar | `ip a`, `ss`, `nmap` |
| 2 | [Auditoria de Sistemas Linux e Análise Avançada de Logs](./sessao-02/README.md) | OA2 — Avaliar | `grep`, `awk`, `journalctl`, `/var/log/auth.log` |
| 3 | [Hardening de Redes Linux e Configuração de Firewalls](./sessao-03/README.md) | OA3 — Aplicar | `UFW`, `iptables`, `netfilter` |
| 4 | [Gestão Segura de Acessos Remotos SSH em Linux](./sessao-04/README.md) | OA4 — Aplicar | `ssh-keygen` (Ed25519), `sshd_config`, `systemd` |
| 5 | [Análise de Vulnerabilidades em Linux e Ferramentas de Auditoria](./sessao-05/README.md) | OA5 — Criar | `Lynis`, CIS Benchmarks |

---

## Sessão 1 — Introdução ao Linux para Comandos de Segurança e Rede

**Objetivo:** realizar reconhecimento e enumeração de um alvo remoto Windows usando
comandos core de rede e scanning do Linux, identificando portas abertas, serviços em
execução, versões exatas, e o sistema operativo do alvo.

**O que foi feito:**
- Inspeção da configuração de rede local (`ip a`) e sockets em escuta (`ss -tuln`)
- Scan completo de portas/serviços contra o alvo remoto com `nmap`
- Identificação de 5 portas abertas e as respetivas versões exatas de serviço
- Fingerprinting do SO via informação NTLM do RDP
- Análise de risco e recomendação do próximo passo de investigação

**Resultado-chave:** alvo `WIN-SCAN` (Windows Server, build 10.0.17763) com login FTP
anónimo permitido, HTTP TRACE ativo no IIS, e RPC/RDP expostos externamente.

📄 [Relatório completo →](./sessao-01/README.md)

---

## Sessão 2 — Auditoria de Sistemas Linux e Análise Avançada de Logs

**Objetivo:** investigar um ataque de força bruta SSH através da análise de logs de
autenticação, reconstruindo a timeline do ataque até identificar o momento exato do
comprometimento.

**O que foi feito:**
- Localização e filtragem de `/var/log/auth.log` com `grep`
- Contagem e ranking de IPs suspeitos com `awk` + `sort` + `uniq -c`
- Confirmação do sucesso do ataque (`Accepted password`)
- Identificação do IP atacante, utilizador afetado e timestamp do comprometimento

**Resultado-chave:** IP atacante `10.129.151.161`, utilizador `fred`, comprometimento às
`12:02:35`, após 4 tentativas falhadas.

📄 [Relatório completo →](./sessao-02/README.md)

---

## Sessão 3 — Hardening de Redes Linux e Configuração de Firewalls

**Objetivo:** configurar uma política defensiva estrita ("bloquear tudo por defeito,
abrir apenas o essencial") combinando UFW e iptables.

**O que foi feito:**
- Definição de políticas por defeito (`deny incoming` / `allow outgoing`)
- Permissão explícita da porta SSH antes da ativação da firewall
- Simulação de bloqueio de um IP malicioso via `iptables` (DROP)
- Persistência das regras em `/etc/iptables/rules.v4`

**Resultado-chave:** apenas a porta 22/tcp aberta; IP `203.0.113.50` bloqueado
silenciosamente (DROP) na chain INPUT.

📄 [Relatório completo →](./sessao-03/README.md)

---

## Sessão 4 — Gestão Segura de Acessos Remotos SSH em Linux

**Objetivo:** eliminar a autenticação por password em favor de chaves criptográficas
Ed25519, bloquear login root direto e mudar a porta padrão do SSH.

**O que foi feito:**
- Geração de par de chaves Ed25519 e distribuição via `ssh-copy-id`
- Reconfiguração do `sshd_config`: `PasswordAuthentication no`, `PermitRootLogin no`,
  `Port 2222`
- Validação obrigatória da sintaxe (`sshd -t`) antes do restart
- Resolução de um conflito real com `ssh.socket` do systemd
- Confirmação de login bem-sucedido apenas com chave, na nova porta

**Resultado-chave:** login SSH funcional exclusivamente por chave privada, na porta
2222, sem acesso root direto.

📄 [Relatório completo →](./sessao-04/README.md)

---

## Sessão 5 — Análise de Vulnerabilidades em Linux e Ferramentas de Auditoria

**Objetivo:** auditar objetivamente a robustez do sistema com o Lynis, interpretar o
Hardening Score e propor correções para vulnerabilidades críticas.

**O que foi feito:**
- Instalação e execução completa do Lynis (`lynis audit system`)
- Registo do Hardening Score inicial e contagem de Warnings/Suggestions
- Seleção de 2 vulnerabilidades críticas (Authentication e Filesystem)
- Pesquisa e documentação da correção oficial (base de dados Cisofy)

**Resultado-chave:** Hardening Score inicial de **60/100**; vulnerabilidades
`[AUTH-9262]` (falta de módulo PAM de reforço de password) e `[FILE-7524]`
(permissões de ficheiros a restringir) identificadas e corrigidas.

📄 [Relatório completo →](./sessao-05/README.md)

---

## Progressão do Percurso

```
Sessão 1 (Reconhecer)  →  Sessão 2 (Detetar)  →  Sessão 3 (Bloquear)  →  Sessão 4 (Fechar Acessos)  →  Sessão 5 (Auditar)
      Nmap                     Logs                  Firewall                    SSH                      Lynis
```

O percurso segue uma progressão lógica: começamos por **reconhecer** a superfície de
ataque de um alvo (scanning de portas e serviços), depois aprendemos a **detetar** um
ataque já ocorrido (forense de logs), a **bloquear** tráfego indesejado (firewall), a
**fechar** o vetor de ataque mais comum (SSH por password), e por fim a **auditar** de
forma objetiva e sistemática o estado geral de segurança do sistema — fechando o ciclo
de reconhecimento → deteção → prevenção → validação.

---

## Estrutura do Repositório

```
.
├── README.md                 (este ficheiro — índice geral)
├── sessao-01/
│   └── README.md              (Reconhecimento com Nmap)
├── sessao-02/
│   └── README.md              (Auditoria de Logs)
├── sessao-03/
│   └── README.md              (Firewalls UFW/iptables)
├── sessao-04/
│   └── README.md              (SSH com chaves Ed25519)
└── sessao-05/
    └── README.md              (Auditoria com Lynis)
```

---

## Ambientes Práticos Utilizados

- **TryHackMe** — Intro to Logs, Linux Server Forensics, Network Security Essentials,
  Linux Strength Training, Linux Process Analysis
- **KillerCoda** — Ubuntu Playground (usado em todas as sessões práticas)

---

*Portfólio desenvolvido no âmbito do percurso Reskilling — Linux e Cibersegurança,
Skodji Digital.*
