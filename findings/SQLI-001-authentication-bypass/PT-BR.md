# SQLI-001 — SQL Injection com Bypass de Autenticação

## 1. Identificação

- ID: SQLI-001
- Data: 28/10/2026
- Aplicação: OWASP Juice Shop
- Ambiente: Laboratório Docker isolado
- Endpoint vulnerável: POST /rest/user/login
- Parâmetro: email
- Vulnerabilidade: SQL Injection
- CWE: CWE-89
- OWASP WSTG: WSTG-INPV-05
- Status: CONFIRMADA
- Severidade preliminar: Crítica
- CVSS: Pendente de cálculo formal

## 2. Resumo executivo

Foi identificada uma vulnerabilidade de SQL Injection
no mecanismo de autenticação da aplicação.

A manipulação do parâmetro email permitiu contornar
a verificação da senha e obter uma sessão autenticada.

A exploração foi inicialmente demonstrada com uma
conta administrativa do laboratório.

Posteriormente, uma conta de teste foi criada para
validar o impacto sobre a integridade dos dados.

A sessão obtida por SQL Injection permitiu modificar
o campo username dessa conta.

A alteração foi confirmada diretamente no SQLite
e o valor original foi restaurado.

## 3. Escopo

Aplicação:
OWASP Juice Shop

Endereço:
http://172.18.0.2:3000

Ambiente:
Container Docker em rede interna de laboratório.

Todos os testes foram realizados em ambiente
controlado e autorizado.

## 4. Identificação da vulnerabilidade

### Teste 1 — Credenciais inválidas

Resultado:
HTTP 401 Unauthorized

Interpretação:
A autenticação foi corretamente recusada.

### Teste 2 — Aspas simples

Entrada no parâmetro email:

'

Resultado:
HTTP 500 Internal Server Error

A resposta revelou referências internas
ao Sequelize e ao SQLite.

Esse comportamento levantou a hipótese
de manipulação insegura de consultas SQL.

### Teste 3 — Bypass de autenticação

Padrão de payload:

' OR 1=1--

Resultado:
HTTP 200 OK

A aplicação emitiu um JWT associado
à identidade administrativa.

O token não foi armazenado no relatório.

## 5. Validação da sessão

Endpoint:

GET /rest/user/whoami

O token foi enviado pelo cookie:

token=<VALOR_REDACTED>

Resultado:

- Identidade administrativa reconhecida.
- ID administrativo: 1.
- Papel identificado no JWT: admin.

O endpoint reconheceu a sessão quando
o token foi enviado como cookie.

## 6. Prova de impacto sobre a integridade

Foi criada uma conta exclusiva para o teste.

Conta:
pentest-lab-001@example.invalid

ID:
25

A autenticação foi obtida por SQL Injection,
utilizando uma senha intencionalmente incorreta.

A sessão foi utilizada para modificar
o campo username pelo endpoint POST /profile.

### Verificação direta no SQLite

Banco:
 /juice-shop/data/juiceshop.sqlite

Tabela:
Users

Registro:
ID 25

Estado original:

username = ''

Após a alteração:

username = 'SQLI_001_PROVA_INTEGRIDADE'

Após a restauração:

username = ''

### Resultado

- Autenticação indevida: CONFIRMADA
- Sessão reconhecida: CONFIRMADA
- Alteração persistente: CONFIRMADA
- Restauração: CONFIRMADA

Nenhum outro registro foi modificado.

## 7. Impacto

A vulnerabilidade permite contornar
a autenticação e obter sessões de usuários
sem conhecer suas senhas.

No laboratório, foi demonstrada a emissão
de uma sessão administrativa.

Também foi demonstrada a modificação
persistente de informações de uma conta
de teste utilizando uma sessão indevida.

O impacto confirmado envolve:

- Comprometimento da autenticação.
- Acesso não autorizado.
- Comprometimento da integridade dos dados.

Não foram executadas operações administrativas
de escrita nem extração de dados de terceiros.

## 8. Causa raiz

Há forte evidência de construção insegura
de consultas SQL com entrada controlada
pelo usuário.

A confirmação detalhada no código-fonte
ainda está pendente.

## 9. Recomendações

1. Utilizar consultas parametrizadas.
2. Evitar concatenação de entradas em SQL.
3. Utilizar métodos seguros do ORM.
4. Validar a autenticação no servidor.
5. Aplicar tratamento seguro de erros.
6. Desabilitar stack traces em produção.
7. Criar testes automatizados de regressão.
8. Revisar os demais pontos de entrada SQL.

## 10. Validação da correção

Status: PENDENTE

Após a correção, repetir os testes
com credenciais inválidas e payloads SQL.

O comportamento esperado é a rejeição
da autenticação sem emissão de token.

## 11. Tratamento das evidências

- Nenhum JWT foi publicado.
- Nenhum cookie de sessão foi publicado.
- Nenhuma credencial real foi exposta.
- As alterações foram limitadas à conta de teste.
- O estado original foi restaurado.

## 12. Conclusão

SQL Injection confirmada no endpoint
de autenticação do OWASP Juice Shop.

A exploração demonstrou bypass de autenticação,
obtenção de sessão válida e modificação
persistente de dados no SQLite.

A correção e o reteste permanecem pendentes.

## 13. Causa raiz confirmada no código-fonte

Arquivo: routes/login.ts
Linha: 34

A aplicação utiliza models.sequelize.query() com
interpolação direta do parâmetro req.body.email.

A entrada controlada pelo usuário é incorporada
à consulta SQL sem parametrização.

O payload com aspas simples e comentário SQL
permitiu excluir a verificação da senha da
condição efetivamente executada.

A análise do código confirma a causa raiz
da vulnerabilidade SQLI-001.

A correção deve utilizar consultas parametrizadas,
sem interpolação direta de entradas externas.

## Reteste da correção — Evidências

### Ambiente

- Aplicação: OWASP Juice Shop
- Container original: juice-shop-lab
- Container corrigido: juice-shop-fixed
- Correção: parâmetros vinculados na consulta SQL de autenticação
- Arquivo corrigido: build/routes/login.js

### Teste 1 — SQL Injection após correção

**Entrada:** payload de bypass de autenticação anteriormente confirmado.

**Resultado observado:**
- HTTP: 401 Unauthorized
- Token emitido: não
- Autenticação concedida: não

**Conclusão:** PASSOU. O payload testado não permitiu autenticação.

### Teste 2 — Cadastro legítimo

**Resultado observado:**
- HTTP: 201 Created

**Conclusão:** PASSOU. A criação de uma conta de teste funcionou.

### Teste 3 — Autenticação legítima

**Resultado observado:**
- HTTP: 200 OK
- Token emitido: sim
- Autenticação concedida: sim

**Conclusão:** PASSOU. O login legítimo continuou funcionando.

### Avaliação final

A correção passou nos testes executados e impediu a exploração
anteriormente demonstrada, preservando o fluxo de autenticação
legítima.

O resultado não constitui garantia de ausência de outras
variantes de SQL Injection.

Nenhuma credencial, cookie de sessão ou token foi incluído
nesta documentação.

## Código vulnerável e correção

### Código vulnerável

```typescript
const sql = `SELECT * FROM Users WHERE email = '${req.body.email}'`;
```

A entrada do usuário é interpolada diretamente na consulta SQL.

### Código corrigido

```typescript
const sql = "SELECT * FROM Users WHERE email = $email";
const options = { bind: { email: req.body.email } };
```

Os parâmetros vinculados separam os dados da estrutura SQL.

### Reteste

- SQL Injection: HTTP 401, sem token.
- Login legítimo: HTTP 200.

Os trechos são simplificados para demonstrar a causa raiz e a técnica de correção.
