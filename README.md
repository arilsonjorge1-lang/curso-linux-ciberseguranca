# Sessão 02 — Auditoria de Sistemas Linux e Análise Avançada de Logs

**Curso:** Reskilling — Linux e Cibersegurança
**Objetivo de Aprendizagem:** OA2 — Avaliar
**Ambiente prático:** TryHackMe (Intro to Logs + Linux Server Forensics)

## Contexto

Um servidor da infraestrutura foi alvo de conexões anómalas. Este relatório documenta
a investigação forense realizada aos logs de autenticação (/var/log/auth.log) para
determinar a origem, o sucesso e o impacto do ataque de força bruta SSH.

---

## Metodologia

- Navegação até ao diretório de logs (`cd /var/log/`)
- Isolamento das tentativas falhadas de login (`grep "Failed password"`)
- Contagem de tentativas por IP (`awk` + `sort` + `uniq -c`)
- Confirmação de sucesso do ataque (`grep -E "Accepted password|Accepted publickey"`)

---

## Resultados

| Item | Valor |
|---|---|
| **IP do Atacante** | `10.129.151.161` |
| **Timestamp do Comprometimento** | `Jul 17 12:02:35` |
| **Utilizador Afetado** | `fred` |

---

## Linha Temporal do Ataque

| Timestamp | Evento | IP |
|---|---|---|
| Jul 17 11:59:56 | Failed password (tentativa 1) — utilizador fred | 10.129.151.161 |
| Jul 17 12:00:13 | Failed password (tentativa 2) — utilizador fred | 10.129.151.161 |
| Jul 17 12:01:23 | Failed password (tentativa 3) — utilizador fred | 10.129.151.161 |
| Jul 17 12:02:10 | Failed password (tentativa 4) — utilizador fred | 10.129.151.161 |
| Jul 17 12:02:35 | **Accepted password — COMPROMETIMENTO** | 10.129.151.161 |

---

## Evidências (Excertos dos Logs)

### Tentativas falhadas
```
$ grep "Failed password" auth.log
Jul 17 11:59:56 ip-10-129-136-117 sshd[1308]: Failed password for fred from 10.129.151.161 port 55070 ssh2
Jul 17 12:00:13 ip-10-129-136-117 sshd[1308]: Failed password for fred from 10.129.151.161 port 55070 ssh2
Jul 17 12:01:23 ip-10-129-136-117 sshd[1316]: Failed password for fred from 10.129.151.161 port 45300 ssh2
Jul 17 12:02:10 ip-10-129-136-117 sshd[1316]: Failed password for fred from 10.129.151.161 port 45300 ssh2
```

### Ranking de IPs por número de tentativas
```
$ grep "Failed password" auth.log | awk '{print $11}' | sort | uniq -c | sort -nr
      4 10.129.151.161
```

### Login com sucesso (evidência do comprometimento)
```
$ grep -E "Accepted password|Accepted publickey" auth.log
Jul 17 12:02:35 ip-10-129-136-117 sshd[1316]: Accepted password for fred from 10.129.151.161 port 45300 ssh2
```

---

## Indicador de Comprometimento (IoC)

| Tipo | Valor |
|---|---|
| IP suspeito | 10.129.151.161 |
| Utilizador visado | fred |
| Timestamp do sucesso | Jul 17 12:02:35 |
| Método de autenticação usado | password |

---

## Recomendações

- Desativar autenticação por password, permitir apenas chaves SSH (`PasswordAuthentication no`)
- Configurar Fail2ban para bloqueio automático de IPs após tentativas repetidas
- Restringir acesso SSH via firewall/VPN de gestão
- Ativar MFA para autenticação remota
- Monitorização contínua dos logs de autenticação

---

*Relatório elaborado no âmbito da Sessão 2 do percurso Reskilling — Linux e Cibersegurança.*
