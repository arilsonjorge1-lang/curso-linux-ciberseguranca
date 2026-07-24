# Sessão 05 — Análise de Vulnerabilidades em Linux e Ferramentas de Auditoria

**Curso:** Reskilling — Linux e Cibersegurança
**Objetivo de Aprendizagem:** OA5 — Criar
**Ambiente prático:** KillerCoda Ubuntu Playground

## Contexto

Execução de um exame de auditoria técnica automatizada com o **Lynis** para identificar
desvios de conformidade em relação aos standards de segurança recomendados
(CIS Benchmarks), seguida da seleção e correção proposta para duas vulnerabilidades
críticas nas áreas de Authentication e Filesystem.

---

## Metodologia

1. Instalação do Lynis (`sudo apt install lynis -y`)
2. Execução da auditoria completa (`sudo lynis audit system`)
3. Registo do Hardening Score inicial, contagem de Warnings e Suggestions
4. Seleção de 2 Suggestions críticas nas áreas de Authentication e Filesystem
5. Pesquisa da correção recomendada na base de dados oficial da Cisofy
6. Documentação da medida corretiva proposta para cada uma

---

## Resultados da Auditoria Inicial

| Métrica | Valor |
|---|---|
| **Hardening Index** | **60 / 100** |
| **Tests performed** | 262 |
| **Warnings** | 1 |
| **Suggestions** | 50 |

### Ambiente auditado
```
Operating system name: Ubuntu
Operating system version: 24.04
Kernel version: 6.8.0
Hardware platform: x86_64
```

### Warning encontrado
```
! Found one or more vulnerable packages. [PKGS-7392]
https://cisofy.com/lynis/controls/PKGS-7392/
```

### Componentes verificados
```
- Firewall              [V]  (encontrada e ativa)
- Malware scanner       [X]  (não encontrado)
```

---

## Vulnerabilidades Selecionadas e Correções Propostas

### 1. [AUTH-9262] — Ausência de módulo PAM para testar força de passwords

**Categoria:** Authentication

**Output original do Lynis:**
```
* Install a PAM module for password strength testing like pam_cracklib or pam_passwdqc [AUTH-9262]
  https://cisofy.com/lynis/controls/AUTH-9262/
```

**Risco:** sem um módulo PAM de verificação de qualidade, o sistema aceita passwords
fracas ou triviais na criação/alteração de contas, aumentando a exposição a ataques de
dicionário e força bruta caso a autenticação por password volte a estar ativa nalgum
serviço.

**Correção recomendada (Cisofy):** instalar um módulo PAM de reforço de password,
como `libpam-passwdqc` ou `libpam-cracklib`, e configurá-lo em `/etc/pam.d/common-password`.

```bash
sudo apt install libpam-passwdqc -y
```

Após a instalação, o PAM passa a exigir passwords com maior complexidade em qualquer
alteração futura de credenciais.

---

### 2. [FILE-7524] — Permissões de ficheiros a restringir

**Categoria:** Filesystem

**Output original do Lynis:**
```
* Consider restricting file permissions [FILE-7524]
  - Details : See screen output or log file
  - Solution : Use chmod to change file permissions
  https://cisofy.com/lynis/controls/FILE-7524/
```

Ficheiros/diretórios sinalizados no scan (secção "File Permissions"):
```
File: /etc/crontab                [ SUGGESTION ]
File: /etc/ssh/sshd_config        [ SUGGESTION ]
Directory: /etc/cron.d            [ SUGGESTION ]
Directory: /etc/cron.daily        [ SUGGESTION ]
Directory: /etc/cron.hourly       [ SUGGESTION ]
Directory: /etc/cron.weekly       [ SUGGESTION ]
Directory: /etc/cron.monthly      [ SUGGESTION ]
```

**Risco:** permissões demasiado abertas em ficheiros de configuração sensíveis
(`sshd_config`, `crontab`) ou nos diretórios de tarefas agendadas permitem que
utilizadores sem privilégios os leiam ou, no pior caso, os modifiquem — podendo levar
a escalada de privilégios ou persistência de um atacante através de tarefas cron
maliciosas.

**Correção recomendada (Cisofy):** usar `chmod` para restringir as permissões destes
ficheiros/diretórios ao mínimo necessário.

```bash
sudo chmod 600 /etc/ssh/sshd_config
sudo chmod 600 /etc/crontab
sudo chmod 700 /etc/cron.d /etc/cron.daily /etc/cron.hourly /etc/cron.weekly /etc/cron.monthly
```

---

## Ligação com Sessões Anteriores

O Lynis confirmou de forma independente que a **firewall está ativa** (`[V]`),
validando o trabalho de hardening realizado na Sessão 3. O único Warning encontrado
(pacotes vulneráveis) não está relacionado com SSH ou firewall, sugerindo que as
configurações aplicadas nas Sessões 3 e 4 se mantiveram corretas nesta instância.

---

## Excerto do Relatório Lynis (Resumo Final)

```
Hardening index : 60 [############        ]
Tests performed  : 262
Plugins enabled  : 1

Components:
- Firewall              [V]
- Malware scanner        [X]

Scan mode:
Normal [V]  Forensics [ ]  Integration [ ]  Pentest [ ]

Warnings (1):
----------------------------
! Found one or more vulnerable packages. [PKGS-7392]
  https://cisofy.com/lynis/controls/PKGS-7392/

Files:
- Test and debug information  : /var/log/lynis.log
- Report data                 : /var/log/lynis-report.dat
```

---

## Próximos Passos Recomendados

1. Aplicar as duas correções propostas acima
2. Correr novamente `sudo lynis audit system --quick`
3. Comparar o novo Hardening Index com o valor inicial (60/100) para confirmar
   a melhoria mensurável

---

*Relatório elaborado no âmbito da Sessão 5 do percurso Reskilling — Linux e Cibersegurança.*
