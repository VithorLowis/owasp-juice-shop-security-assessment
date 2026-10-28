# Pentest Web | OWASP Juice Shop

### Vithor Lowis Cybersecurity

**Segurança Ofensiva • Análise de Vulnerabilidades • Correção e Validação**

[🇧🇷 Documentação em Português](findings/SQLI-001-authentication-bypass/PT-BR.md) | [🌎 Technical Report in English](findings/SQLI-001-authentication-bypass/EN-US.md)

---

## Sobre o projeto

Este repositório apresenta estudos de caso de testes de intrusão realizados em um laboratório controlado utilizando o OWASP Juice Shop.

O objetivo é demonstrar, na prática, o ciclo completo de um pentest:

1. Reconhecimento e análise da superfície de ataque.
2. Identificação de vulnerabilidades.
3. Exploração controlada.
4. Comprovação do impacto.
5. Análise da causa raiz.
6. Desenvolvimento de correções.
7. Retestes de segurança.
8. Documentação técnica e recomendações.

## Metodologia

Os testes seguem como referência o OWASP Web Security Testing Guide (WSTG).

As evidências são coletadas em ambiente autorizado, com documentação técnica em português e inglês.

---

## Vulnerabilidades investigadas

### SQLI-001 — SQL Injection | Bypass de Autenticação

**Classificação:** CWE-89 — SQL Injection

**Funcionalidade afetada:** Autenticação de usuários

**Status:** Exploração confirmada e correção aprovada nos retestes realizados.

#### Impacto demonstrado

- Bypass de autenticação utilizando senha incorreta.
- Obtenção de sessão autenticada sem a senha legítima.
- Alteração controlada de dados de uma conta do laboratório.
- Confirmação da alteração diretamente no banco SQLite.
- Restauração dos dados originais.

#### Causa raiz

Interpolação direta de entradas controladas pelo usuário em uma consulta SQL.

#### Correção implementada

Substituição da interpolação insegura por parâmetros vinculados.

#### Resultados do reteste

| Teste | Resultado |
|---|---|
| Payload de SQL Injection anteriormente explorável | Bloqueado — HTTP 401 |
| Cadastro legítimo | Funcionando — HTTP 201 |
| Autenticação legítima | Funcionando — HTTP 200 |

A correção bloqueou o payload testado sem comprometer a autenticação legítima.

Esses resultados não garantem a ausência de todas as possíveis variantes de SQL Injection.

**Documentação completa:**

- [Relatório técnico — Português](findings/SQLI-001-authentication-bypass/PT-BR.md)
- [Technical Report — English](findings/SQLI-001-authentication-bypass/EN-US.md)
- [Patch de segurança](findings/SQLI-001-authentication-bypass/remediation/SQLI-001-security.patch)

---

### SQLI-002 — Possível SQL Injection na Pesquisa de Produtos

Durante a análise do código-fonte, foi identificada uma consulta SQL com interpolação potencialmente insegura.

**Status:** Suspeita técnica — investigação pendente.

A vulnerabilidade ainda não foi comprovada por exploração.

[Consultar registro da investigação](notes/SQLI-002-pending-investigation.md)

---

## Escopo e responsabilidade

Todos os testes foram realizados em instâncias autorizadas e isoladas do OWASP Juice Shop, uma aplicação intencionalmente vulnerável para fins educacionais.

Este projeto não representa uma auditoria de sistemas de produção.

Tokens de autenticação, cookies de sessão, credenciais reais e informações pessoais sensíveis não devem ser publicados.

---

## Sobre a Vithor Lowis Cybersecurity

Projeto profissional dedicado à segurança da informação, com foco em:

- Pentest Web e de APIs.
- Pentest Mobile.
- Pentest de Redes e Infraestrutura.
- Análise e correção de vulnerabilidades.
- Segurança de aplicações e hardening.

**Compromisso:** identificar riscos, demonstrar impactos de maneira controlada e propor correções tecnicamente fundamentadas.

---

## Créditos e referências

Este estudo utiliza o **OWASP Juice Shop**, uma aplicação
intencionalmente vulnerável desenvolvida para treinamento
e pesquisa em segurança de aplicações.

Projeto original: OWASP Juice Shop.

O código vulnerável apresentado neste repositório pertence
ao projeto original. As análises, a documentação dos testes,
a proposta de correção e os resultados do reteste foram
produzidos no laboratório da Vithor Lowis Cybersecurity.

Este portfólio é independente e não representa uma auditoria
oficial da OWASP.

A publicação de trechos e alterações do código deve respeitar
a licença aplicável à versão utilizada.
