# Curso Reskilling — Linux e Cibersegurança

Repositório de portfólio com a documentação técnica dos laboratórios práticos do
percurso **Reskilling — Linux e Cibersegurança**, promovido pelo Ministério da
Economia Digital de Cabo Verde, em parceria com a Skodji Digital e o World Bank Group.

Cada sessão corresponde a uma pasta com um relatório técnico (`README.md`),
evidências reais recolhidas em ambiente prático (TryHackMe / KillerCoda Ubuntu
Playground) e, quando aplicável, ficheiros de configuração ou excertos de logs
analisados.

---

## Índice de Sessões

| Sessão | Título | Objetivo de Aprendizagem | Documentação |
|---|---|---|---|
| 02 | Auditoria de Sistemas Linux e Análise Avançada de Logs | OA2 — Avaliar | [sessao-02/README.md](sessao-02/README.md) |
| 03 | Hardening de Redes Linux e Configuração de Firewalls | OA3 — Aplicar | [sessao-03/README.md](sessao-03/README.md) |
| 04 | Gestão Segura de Acessos Remotos SSH em Linux | OA4 — Aplicar | [sessao-04/README.md](sessao-04/README.md) |
| 05 | Análise de Vulnerabilidades em Linux e Ferramentas de Auditoria | OA5 — Criar | [sessao-05/README.md](sessao-05/README.md) |

---

## Resumo por Sessão

### Sessão 02 — Forense de Logs SSH
Investigação de um ataque de força bruta SSH através da análise de
`/var/log/auth.log`, usando `grep`, `awk` e `journalctl`, culminando na
identificação do IP atacante, do timestamp de comprometimento e do utilizador
afetado.

### Sessão 03 — Hardening de Firewalls
Configuração de uma política de segurança "bloquear tudo por defeito, abrir
apenas o essencial" usando UFW e iptables, incluindo a distinção entre DROP e
REJECT e o bloqueio de um IP malicioso simulado.

### Sessão 04 — Acessos SSH Seguros
Eliminação da autenticação por password em favor de chaves criptográficas
Ed25519, bloqueio de login root direto e mudança da porta padrão do SSH,
validando sempre a configuração antes de reiniciar o serviço.

### Sessão 05 — Auditoria com Lynis
Execução de uma auditoria automatizada baseada nos CIS Benchmarks com o Lynis,
interpretação do Hardening Score, e proposta de correções para vulnerabilidades
críticas identificadas.

---

## Ambientes Práticos Utilizados

- [TryHackMe](https://tryhackme.com/)
- [KillerCoda Ubuntu Playground](https://killercoda.com/playgrounds/scenario/ubuntu)

## Autor

Percurso Reskilling — Linux e Cibersegurança
Skodji Digital / Ministério da Economia Digital — Governo de Cabo Verde
