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

### SQLI-002 — SQL Injection na Pesquisa de Produtos

**Classificação:** CWE-89 — SQL Injection

**Endpoint:** `GET /rest/products/search?q=...`

**Severidade:** Crítica (preliminar; CVSS não calculado)

**Status:** Exploração confirmada, correção implementada e retestes aprovados.

#### Impacto demonstrado

- Manipulação de consultas SQL por condições booleanas.
- Execução de consultas UNION SELECT.
- Enumeração de 22 tabelas do SQLite.
- Identificação de 24 registros na tabela Users.
- Recuperação controlada de registros e hashes de senha.

#### Correção implementada

Substituição da interpolação SQL por parâmetros vinculados (`bind`).

#### Resultados dos retestes

| Teste | Resultado |
|---|---|
| Pesquisa legítima | HTTP 200 — 3 produtos |
| SQL Injection booleana | HTTP 200 — 0 produtos |
| SQL Injection UNION | HTTP 200 — 0 produtos |
| Tentativa de acesso à tabela Users | HTTP 200 — 0 produtos |

Os vetores testados foram bloqueados sem comprometer a pesquisa legítima.

**Documentação completa:**

- [Relatório técnico — Português](findings/SQLI-002-product-search/PT-BR.md)
- [Technical Report — English](findings/SQLI-002-product-search/EN-US.md)
- [Patch de segurança](findings/SQLI-002-product-search/remediation/SQLI-002-security.patch)

### CRYPTO-001 — Armazenamento Inseguro de Senhas (MD5)

**Status:** Vulnerabilidade confirmada; correção pendente.

Durante a investigação SQLI-002, foi identificado o uso de MD5 sem salt individual para armazenamento de senhas.

[Consultar documentação técnica](notes/CRYPTO-001-weak-password-hashing.md)

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
