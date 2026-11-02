# Testes Automatizados — CLYVO Predict

Suite de testes de integração e segurança da API CLYVO Predict, para a **Parte B** da entrega de Compliance, Quality Assurance & Tests (Sprint 4).

**Ferramenta:** Postman / Newman.
O sistema é composto por API REST sem interface web própria (a camada de apresentação é o aplicativo mobile), o que enquadra a entrega na opção *"sistema composto por APIs e componentes de serviços"* prevista no enunciado.

---

## Conteúdo

| Arquivo | O que é |
|---|---|
| `CLYVO_Predict.postman_collection.json` | A collection, com 36 requisições e 71 asserções |
| `CLYVO_Predict.postman_environment.json` | Environment com a variável `baseUrl` |

## Cobertura

| Pasta | Casos | O que valida |
|---|---|---|
| 00 - Setup | 8 | Cadastro e autenticação de 2 tutores e 2 veterinários |
| 01 - Fluxos principais | 8 | CRUD de pets, registro de eventos, Health Score e cache |
| 02 - Segurança e ownership | 16 | Acesso a recursos de terceiros, por perfil e por proprietário |
| 03 - Códigos de erro HTTP | 4 | 401, 403, 404 e 400 |

O enunciado exige no mínimo 4 casos automatizados. A suite entrega **28 casos de teste** (CT01 a CT28), além dos 8 passos de preparação.

---

## Como executar pela interface do Postman

1. **Importar** — botão `Import`, selecione os dois arquivos `.json`.
2. **Selecionar o environment** — canto superior direito, escolha `CLYVO Predict`.
3. **Ajustar a URL** — se a API não estiver em `http://localhost:8080`, edite a variável `baseUrl` no environment.
4. **Executar** — clique nos `...` da collection, depois `Run collection` e `Run CLYVO Predict`.

A collection precisa rodar **na ordem em que está**: a pasta de Setup cria os usuários e guarda os tokens que as demais utilizam.

Cada execução gera e-mails e CRMVs únicos (sufixo com timestamp), então a suite pode ser executada várias vezes seguidas sem conflito de chave única no banco.

## Como executar por linha de comando (Newman)

```bash
npm install -g newman

newman run CLYVO_Predict.postman_collection.json \
       -e CLYVO_Predict.postman_environment.json
```

Apontando para outra URL:

```bash
newman run CLYVO_Predict.postman_collection.json \
       -e CLYVO_Predict.postman_environment.json \
       --env-var "baseUrl=https://sua-api.azurecontainer.io"
```

### Relatório em HTML (recomendado como evidência)

```bash
npm install -g newman-reporter-htmlextra

newman run CLYVO_Predict.postman_collection.json \
       -e CLYVO_Predict.postman_environment.json \
       -r htmlextra --reporter-htmlextra-export relatorio.html
```

Gera um `relatorio.html` com todos os casos, requisições, respostas e o resultado de cada asserção. Serve de anexo à entrega.

---

## Os 28 casos de teste

### Fluxos principais

| Caso | Objetivo | Resultado esperado |
|---|---|---|
| CT01 | Cadastrar pet ignorando o `tutorId` enviado no corpo | 201 · dono = tutor do token, não o 999999 enviado |
| CT02 | Listar pets do tutor autenticado | 200 · todos os pets pertencem a ele |
| CT03 | Consultar o próprio pet | 200 |
| CT04 | Atualizar o próprio pet | 200 · dono não muda |
| CT05 | Registrar evento DOENCA_LEVE | 201 · score 100 para 85 |
| CT06 | Listar pets após o evento | 200 · score 85 na listagem (cache invalidado) |
| CT07 | Consultar o histórico do próprio pet | 200 · contém o evento |
| CT08 | Registrar evento RELATO_SINTOMA | 201 · score 85 para 80 |

### Segurança e ownership

| Caso | Objetivo | Resultado esperado |
|---|---|---|
| CT09 | Tutor B lista pets | 200 · o pet do Tutor A não aparece |
| CT10 | Tutor B consulta pet do Tutor A | 403 |
| CT11 | Tutor B altera pet do Tutor A | 403 |
| CT12 | Tutor B exclui pet do Tutor A | 403 |
| CT13 | Tutor B registra evento no pet do Tutor A | 403 |
| CT14 | Tutor B consulta histórico do pet do Tutor A | 403 |
| CT15 | Conferir o score do pet do Tutor A | 200 · score intacto em 80 |
| CT16 | Tutor A consulta dados do Tutor B | 403 · não vaza e-mail |
| CT17 | Tutor A altera a conta do Tutor B | 403 |
| CT18 | Tutor B faz login com a senha original | 200 · senha não foi trocada |
| CT19 | Tutor A exclui a conta do Tutor B | 403 |
| CT20 | Tutor consulta a própria conta | 200 · sem o campo senha |
| CT21 | Tutor lista a base de tutores | 403 |
| CT22 | Veterinário lista tutores | 200 |
| CT23 | Veterinário A altera o Veterinário B | 403 |
| CT24 | Veterinário A altera a própria conta | 200 |

### Códigos de erro HTTP

| Caso | Objetivo | Resultado esperado |
|---|---|---|
| CT25 | Consultar pet inexistente | 404 |
| CT26 | Requisição sem token | 401 |
| CT27 | Requisição com token inválido | 401 |
| CT28 | Cadastrar pet com dados inválidos | 400 · lista dos campos em erro |

---

## Rastreabilidade com o backlog do Azure Boards

O enunciado exige que os testes estejam de acordo com o release plan e as tarefas das sprints realizadas. Mapeamento:

| Casos | PBI no Azure Boards |
|---|---|
| S01 a S04 | #30 — Login por perfil vet e tutor com autenticação real |
| S05 a S08, CT26, CT27 | #31 — Autenticação e autorização com Spring Security |
| CT01 a CT04 | #4 — CRUD de pets na API |
| CT05, CT08 | #20 — Score Engine v1 |
| CT06, CT15 | #21 — Recálculo automático do score por evento |
| CT07 | #7 — Registro de consulta com diagnóstico e prescrição |
| CT09 a CT24 | #31 — Autenticação e autorização com Spring Security |
| CT25, CT28 | #51 — Estrutura da API Spring Boot em camadas |
| Esta suite | #65 — Testes automatizados |

---

## O que gravar no vídeo

O enunciado pede evidência da **configuração, execução e resultados**. Roteiro sugerido, de 3 a 5 minutos:

1. **Configuração** — Postman aberto, mostrar a collection importada, abrir o environment e apontar a variável `baseUrl`. Abrir um caso (o CT01 é o melhor) e mostrar a aba **Tests**, com as asserções em JavaScript.
2. **Execução** — `Run collection`, com a tela do runner visível durante a execução.
3. **Resultados** — a tela final com o total de asserções aprovadas. Abrir dois ou três casos para mostrar o detalhe: recomendo o **CT01** (o `tutorId` do corpo é ignorado), o **CT17** (tentativa de sequestro de conta bloqueada com 403) e o **CT18** (a senha original continua funcionando, provando que o CT17 não teve efeito).
4. **Opcional** — rodar pelo Newman no terminal e abrir o `relatorio.html`.

Narre o que cada caso prova. O avaliador precisa entender a intenção do teste, não só ver telas passando.

---

## Observação sobre a validação desta suite

A collection foi executada ponta a ponta antes da entrega: 36 requisições, 71 asserções, todas aprovadas contra uma implementação de referência da API com as correções da Sprint 4.

Se algum caso de segurança (CT09 a CT24) falhar com status 200 onde se espera 403, significa que a correspondente correção de ownership não está ativa no código em execução.
