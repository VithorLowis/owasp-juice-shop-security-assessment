# SQLI-002 — SQL Injection na pesquisa de produtos

**Aplicação:** OWASP Juice Shop
**Classificação:** CWE-89 / OWASP WSTG-INPV-05
**Severidade:** Crítica (preliminar; CVSS não calculado)
**Status:** Corrigida e retestada
**Ambiente:** Laboratório Docker autorizado

## Resumo executivo

A pesquisa pública permitia manipular consultas SQL e acessar outras tabelas.

## Endpoint vulnerável

`GET /rest/products/search?q=...`

## Causa raiz e remediação

A consulta original interpolava a entrada do usuário diretamente no SQL. O patch abaixo substitui a interpolação por um parâmetro vinculado.

### Patch validado do código-fonte

~~~diff
--- a/routes/search.ts
+++ b/routes/search.ts
@@ -20,7 +20,14 @@
   return (req: Request, res: Response, next: NextFunction) => {
     let criteria: any = req.query.q === 'undefined' ? '' : req.query.q ?? ''
     criteria = (criteria.length <= 200) ? criteria : criteria.substring(0, 200)
-    models.sequelize.query(`SELECT * FROM Products WHERE ((name LIKE '%${criteria}%' OR description LIKE '%${criteria}%') AND deletedAt IS NULL) ORDER BY name`) // vuln-code-snippet vuln-line unionSqlInjectionChallenge dbSchemaChallenge
+    models.sequelize.query(
+      "SELECT * FROM Products WHERE ((name LIKE $search OR description LIKE $search) AND deletedAt IS NULL) ORDER BY name",
+      {
+        bind: {
+          search: "%" + String(criteria) + "%"
+        }
+      }
+    ) // vuln-code-snippet vuln-line unionSqlInjectionChallenge dbSchemaChallenge
       .then(([products]: any) => {
         const dataString = JSON.stringify(products)
         if (challengeUtils.notSolved(challenges.unionSqlInjectionChallenge)) { // vuln-code-snippet hide-start
~~~

## Prova de conceito

### SQL Injection baseada em condições booleanas

~~~sql
apple')) OR 1=1--
~~~
Resultado observado: HTTP 200, 56 produtos.

~~~sql
apple')) OR 1=2--
~~~
Resultado observado: HTTP 200, 0 produtos.

### SQL Injection baseada em UNION

~~~sql
apple')) UNION SELECT 999999,'SQLI_002_CONFIRMED','TESTE_CONTROLADO',0,0,0,0,0,0,0--
~~~
Resultado observado: HTTP 200, marcador artificial retornado.

## Exposição de dados do banco

Foram identificadas 22 tabelas, 13 colunas de Users e 24 usuários. Cinco registros e cinco hashes foram recuperados em testes controlados.

~~~sql
SELECT name FROM sqlite_master WHERE type='table';
SELECT name FROM pragma_table_info('Users');
SELECT COUNT(*) FROM Users;
~~~

Nenhuma informação pessoal ou hash de senha recuperado é publicado.

## Validação após a remediação

A consulta foi corrigida com parâmetros vinculados. A pesquisa legítima retornou 3 produtos; os vetores SQLi testados retornaram 0.

| Teste | HTTP | Resultado |
|---|---:|---|
| Pesquisa legítima | 200 | 3 produtos |
| SQLi booleana | 200 | 0 produtos |
| SQLi UNION | 200 | 0 produtos |
| Tentativa de acesso à tabela Users | 200 | 0 produtos |

Os testes de regressão da SQLI-001 retornaram HTTP 401, sem emissão de tokens.

## Descoberta adicional

O armazenamento de senhas com MD5 sem salt individual está documentado separadamente como CRYPTO-001; a correção permanece pendente.

## Escopo e limitações

Os testes foram realizados exclusivamente em contêineres Docker autorizados. A validação abrange os vetores testados, não todas as possibilidades de ataque.

## Referências

CWE-89; OWASP WSTG-INPV-05; OWASP Juice Shop.
