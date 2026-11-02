# CLYVO Predict — Compliance, Quality Assurance & Tests

Entrega da **Sprint 4** da disciplina de Compliance, Quality Assurance & Tests.

**Challenge FIAP 2026 · 2TDSPX · Projeto CLYVO Predict**

---

## Links da entrega

| Item | Link |
|---|---|
| **Plano de testes manuais (Azure Test Plans)** | `COLAR_AQUI_O_LINK_DO_AZURE` |
| **Vídeo dos testes automatizados** | `COLAR_AQUI_O_LINK_DO_VIDEO` |

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

## Parte A — Plano de testes manuais

**14 casos de teste** distribuídos em **5 suítes**, uma por Product Backlog Item, dentro do plano de teste da Sprint 4 no Azure Test Plans.

Documento completo: [`plano-de-testes/PLANO_TESTES_PARTE_A.md`](plano-de-testes/PLANO_TESTES_PARTE_A.md)

| Suíte (PBI) | Casos |
|---|---|
| Autenticação e autorização com Spring Security | 4 |
| CRUD de pets na API | 4 |
| Cadastro de vacina com data de reforço | 2 |
| Score Engine v1 | 2 |
| Listagem de pets ordenada por score | 2 |

Cada critério de aceite dos PBIs foi escrito em **Gherkin** (DADO / QUANDO / ENTÃO) e deu origem a um caso de teste correspondente. Os casos seguem o padrão de nomenclatura iniciado pelo **status code HTTP esperado**, o que mantém a ligação visível entre o critério de aceite e o teste.

Cada caso documenta:

- objetivo, expresso no próprio título
- dados de entrada, em **parâmetros** com valores predefinidos
- resultado esperado por passo
- procedimento passo a passo executável no Test Runner

Os casos cobrem **fluxos de sucesso e de exceção**, e estão vinculados aos PBIs das Sprints 2, 3 e 4 conforme o release plan do projeto.

### Rastreabilidade com a Sprint 3

O caso `[403] Tutor acessa pet de outro tutor é bloqueado` reproduz a falha de controle de acesso apontada no feedback da Sprint 3. O bug foi registrado no Azure a partir da execução contra a versão anterior, corrigido na branch `sprint-04` e reexecutado com resultado aprovado.

---

## Parte B — Testes automatizados

Suíte com **28 casos de teste** e **71 asserções** automatizadas em Postman.

```
postman/
├── CLYVO_Predict.postman_collection.json    a collection
├── CLYVO_Predict.postman_environment.json   variáveis de ambiente
└── README.md                                documentação detalhada
```

### Cobertura

| Pasta | Casos | O que valida |
|---|---|---|
| 00 - Setup | 8 | Cadastro e autenticação de 2 tutores e 2 veterinários |
| 01 - Fluxos principais | 8 | CRUD de pets, eventos de saúde, Health Score e cache |
| 02 - Segurança e ownership | 16 | Acesso a recursos de terceiros, por perfil e por proprietário |
| 03 - Códigos de erro HTTP | 4 | 401, 403, 404 e 400 |

### Execução

```bash
npm install -g newman newman-reporter-htmlextra

newman run postman/CLYVO_Predict.postman_collection.json \
  -e postman/CLYVO_Predict.postman_environment.json \
  -r cli,htmlextra \
  --reporter-htmlextra-export evidencias/relatorio-execucao.html
```

As pastas devem rodar na ordem: a `00 - Setup` cria os usuários e guarda os tokens que as demais consomem.

Instruções completas, incluindo execução pela interface do Postman, em [`postman/README.md`](postman/README.md).

---

## Evidências

A pasta `evidencias/` contém o relatório de execução gerado pelo Newman, com o detalhe de cada requisição, resposta e asserção, além do print do resumo da execução pela interface do Postman.

---

## Estrutura do repositório

```
.
├── README.md
├── plano-de-testes/
│   ├── PLANO_TESTES_PARTE_A.md              roteiro dos 14 casos manuais
│   └── CLYVO_Predict_PlanoTestes_ParteA.docx
├── postman/
│   ├── CLYVO_Predict.postman_collection.json
│   ├── CLYVO_Predict.postman_environment.json
│   └── README.md
└── evidencias/
    └── relatorio-execucao.html
```
