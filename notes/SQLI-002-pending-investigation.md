# SQLI-002 — Investigação concluída

**Status:** Confirmada, corrigida e retestada.

Este documento preserva o registro histórico da investigação inicial. A hipótese foi posteriormente confirmada por testes controlados de SQL Injection booleana e UNION-based.

**Relatórios finais:**
- [Português](../findings/SQLI-002-product-search/PT-BR.md)
- [English](../findings/SQLI-002-product-search/EN-US.md)

**Patch validado:** [SQLI-002-security.patch](../findings/SQLI-002-product-search/remediation/SQLI-002-security.patch)

---

## Registro original da investigação

# SQLI-002 — Possible SQL Injection in Product Search

## PT-BR — Investigação pendente

**Aplicação:** OWASP Juice Shop
**Arquivo:** routes/search.ts
**Linha:** 23
**Funcionalidade:** Pesquisa de produtos
**Categoria:** Possível SQL Injection
**Status:** SUSPEITA — NÃO EXPLORADA

### Evidência inicial

A consulta SQL utiliza interpolação direta da variável
`criteria` nas condições LIKE sobre as colunas
`name` e `description` da tabela Products.

### Próximas verificações

1. Identificar a origem e o tratamento de `criteria`.
2. Estabelecer uma resposta de referência.
3. Testar entradas SQL controladas.
4. Verificar possível manipulação dos resultados.
5. Avaliar possibilidade de leitura indevida.
6. Documentar impacto, causa raiz e correção.
7. Executar reteste após a correção.

Nenhuma exploração desta hipótese foi realizada até agora.

---

## EN-US — Pending Investigation

**Application:** OWASP Juice Shop
**File:** routes/search.ts
**Line:** 23
**Feature:** Product search
**Category:** Potential SQL Injection
**Status:** SUSPECTED — NOT EXPLOITED

### Initial Evidence

The SQL query directly interpolates the `criteria`
variable into LIKE conditions involving the
`name` and `description` columns of the Products table.

### Planned Validation

1. Identify the origin and handling of `criteria`.
2. Establish a baseline response.
3. Test controlled SQL inputs.
4. Assess possible result manipulation.
5. Evaluate unauthorized data retrieval.
6. Document impact, root cause, and remediation.
7. Perform remediation retesting.

No exploitation of this hypothesis has been performed yet.
