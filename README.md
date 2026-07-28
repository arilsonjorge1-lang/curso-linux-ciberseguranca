# Reskilling — Linux e Cibersegurança

**Portfólio Técnico Completo — Sessões 1 a 6**

| | |
|---|---|
| **Aluno** | Arilson do Rosário |
| **Formador** | Péricles Borges |
| **Instituição** | Skodji Digital — Ministério da Economia Digital, Governo de Cabo Verde |

---

## Índice de Sessões

| # | Sessão | Objetivo | Link |
|---|---|---|---|
| 1 | Introdução ao Linux para Comandos de Segurança e Rede | OA1 — Analisar | [sessao-01](./sessao-01/README.md) |
| 2 | Auditoria de Sistemas Linux e Análise Avançada de Logs | OA2 — Avaliar | [sessao-02](./sessao-02/README.md) |
| 3 | Hardening de Redes Linux e Configuração de Firewalls | OA3 — Aplicar | [sessao-03](./sessao-03/README.md) |
| 4 | Gestão Segura de Acessos Remotos SSH em Linux | OA4 — Aplicar | [sessao-04](./sessao-04/README.md) |
| 5 | Análise de Vulnerabilidades em Linux e Ferramentas de Auditoria | OA5 — Criar | [sessao-05](./sessao-05/README.md) |
| 6 | Desafio Mini-CTF Defensivo Linux (Integração OA1–OA5) | Criar | [sessao-06](./sessao-06/README.md) |

## Estrutura do Repositório

```
.
├── README.md                  ← este ficheiro (índice)
├── sessao-01/README.md
├── sessao-02/README.md
├── sessao-03/README.md
├── sessao-04/README.md
├── sessao-05/README.md
└── sessao-06/
    ├── README.md               ← relatório técnico completo (Identificação → Contenção → Remediação → Validação)
    ├── sshd_config              ← cópia limpa da configuração SSH pós-hardening
    └── ufw-rules.txt            ← output de `ufw status verbose`
```

## Progressão Pedagógica

O percurso segue uma progressão ascendente na taxonomia de Bloom, culminando num exercício integrador:

**Analisar** (S1) → **Avaliar** (S2) → **Aplicar** (S3, S4) → **Criar** (S5, S6)

A Sessão 6 é o desafio integrador: aplica em simultâneo o reconhecimento de rede (S1), a auditoria de logs/contas (S2), o hardening de firewall (S3) e de SSH (S4), fechando com uma validação de postura de segurança (S5), tudo num único cenário de resposta a incidente.

---

*Portfólio elaborado no âmbito do percurso Reskilling — Linux e Cibersegurança — Skodji Digital.*
