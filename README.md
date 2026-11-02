# CLYVO Predict — Compliance, Quality Assurance & Tests

Entrega da **Sprint 4** da disciplina de Compliance, Quality Assurance & Tests.

**Challenge FIAP 2026 · 2TDSPX · Projeto CLYVO Predict**

---

## Links da entrega

| Item | Link |
|---|---|
| **Plano de testes manuais (Azure Test Plans)** | [Abrir o plano de testes da Sprint 4](https://dev.azure.com/clyvo-predict-2tdspx/CLYVO%20Predict/_testPlans/execute?planId=138&suiteId=139) |
| **Vídeo dos testes automatizados** | [Assistir ao vídeo](COLAR_AQUI_O_LINK_DO_VIDEO) |

---

## Equipe

| Nome | RM |
|---|---|
| Camily Vitória Pereira Maciel | 566520 |
| Eduarda Weiss Ventura | 564434 |
| Lucas Nunes Soares | 566503 |
| Maria Gabriela Landim Severo | 565146 |
| Samara Porto Souza | 559072 |

---

## Sobre o sistema testado

O **CLYVO Predict** é uma plataforma de gestão veterinária preventiva. O sistema calcula um **Health Score** de 0 a 100 por pet a partir do histórico clínico e dispara alertas quando a nota cai.

A aplicação é composta por uma **API REST em Java/Spring Boot** com banco Oracle, consumida por um aplicativo mobile em React Native e por um microsserviço de IA em Python.

Como o backend não possui camada de visualização web própria — a apresentação é o aplicativo mobile — a automação foi feita com **Postman/Newman**, que é a opção prevista no enunciado para sistemas compostos por APIs e componentes de serviços.

---

## Resultados

| Parte | Execução | Resultado |
|---|---|---|
| **A — Testes manuais** | 14 casos no Azure Test Plans | 12 aprovados · 2 reprovados, com bug registrado |
| **B — Testes automatizados** | 36 requisições · 71 asserções | 71 aprovadas · 0 falhas |

### Defeitos encontrados pela Parte A

| Bug | Caso que reproduz | Severidade |
|---|---|---|
| Login com senha incorreta retorna 400 em vez de 401 | `[401] Login com senha incorreta é recusado` | 3 - Medium |
| Ordenação de `GET /api/pets` é ignorada após uma listagem anterior, porque a chave do cache não considera a ordenação | `[200] Listagem ordenada por score crescente` | 2 - High |

Os dois bugs estão registrados no Azure, vinculados aos casos que os reproduzem, com evidência e correção sugerida.

---

## Parte A — Plano de testes manuais

**14 casos de teste** em **5 suítes**, uma por Product Backlog Item, no plano de teste da Sprint 4 do Azure Test Plans.

Documento completo: [`repo-qa/plano-de-testes/PLANO_TESTES_PARTE_A.md`](repo-qa/plano-de-testes/PLANO_TESTES_PARTE_A.md)

| Suíte (PBI) | Casos |
|---|---|
| Autenticação e autorização com Spring Security | 4 |
| CRUD de pets na API | 4 |
| Recálculo automático do score por evento | 2 |
| Score Engine v1 | 2 |
| Listagem de pets ordenada por score | 2 |

Cada critério de aceite dos PBIs foi escrito em **Gherkin** (DADO / QUANDO / ENTÃO) e deu origem a um caso de teste. O título de cada caso começa pelo **status code HTTP esperado**, o que mantém visível a ligação entre o critério de aceite e o teste.

Cada caso documenta:

- objetivo, expresso no próprio título
- dados de entrada, em **parâmetros** com valores predefinidos
- resultado esperado por passo
- procedimento passo a passo, executado no Test Runner

Os casos cobrem **fluxos de sucesso e de exceção** e estão vinculados aos PBIs das Sprints 2, 3 e 4, conforme o release plan do projeto.

### Rastreabilidade com a Sprint 3

O caso `[403] Tutor acessa pet de outro tutor e é bloqueado` valida a correção da falha de controle de acesso apontada no feedback da Sprint 3.

---

## Parte B — Testes automatizados

Suíte com **28 casos de teste** e **71 asserções** em Postman.

### Cobertura

| Pasta | Casos | O que valida |
|---|---|---|
| 00 - Setup | 8 | Cadastro e autenticação de 2 tutores e 2 veterinários |
| 01 - Fluxos principais | 8 | CRUD de pets, eventos de saúde, Health Score e cache |
| 02 - Segurança e ownership | 16 | Acesso a recursos de terceiros, por perfil e por proprietário |
| 03 - Códigos de erro HTTP | 4 | 401, 403, 404 e 400 |

### Execução

Com a API rodando em `http://localhost:8080`, dentro da pasta `repo-qa`:

```
npm install -g newman newman-reporter-htmlextra

newman run postman/CLYVO_Predict.postman_collection.json -e postman/CLYVO_Predict.postman_environment.json -r cli,htmlextra --reporter-htmlextra-export evidencias/relatorio-execucao.html
```

As pastas devem rodar na ordem: a `00 - Setup` cria usuários novos a cada execução e guarda os tokens que as demais consomem.

Instruções completas, incluindo a execução pela interface do Postman: [`repo-qa/postman/README.md`](repo-qa/postman/README.md)

---

## Evidências

Pasta [`repo-qa/evidencias/`](repo-qa/evidencias/):

| Arquivo | Conteúdo |
|---|---|
| `relatorio-execucao.html` | Relatório do Newman com cada requisição, resposta e asserção. Para visualizar, baixe o arquivo e abra no navegador. |
| `resumo-postman.png` | Resumo da execução pela interface do Postman |
| `azure-pbi-gherkin.png` | Critérios de aceite em Gherkin em um PBI |
| `azure-progresso-plano.png` | Resultado da execução das 5 suítes |
| `azure-graficos.png` | Gráficos por Outcome e por Run by |
| `azure-bugs.png` | Os dois bugs registrados |

---

## Estrutura do repositório

```
.
├── README.md
└── repo-qa/
    ├── plano-de-testes/
    │   ├── PLANO_TESTES_PARTE_A.md                critérios em Gherkin e roteiro dos 14 casos
    │   └── CLYVO_Predict_PlanoTestes_ParteA.docx
    ├── postman/
    │   ├── CLYVO_Predict.postman_collection.json
    │   ├── CLYVO_Predict.postman_environment.json
    │   └── README.md
    └── evidencias/
        ├── relatorio-execucao.html
        ├── resumo-postman.png
        └── azure-*.png
```
