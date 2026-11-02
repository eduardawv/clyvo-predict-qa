# Plano de Testes Manuais - CLYVO Predict - Sprint 4
## Parte A | Azure Boards + Azure Test Plans

> Formato seguindo o tutorial "Plano de testes no Azure - passo a passo" (Prof. Douglas Andrade).
> Regra base: **um caso de teste por criterio de aceite**, titulo iniciado pelo **status code**,
> dado de teste em **parametro @**, cada passo com **resultado esperado**.

---

# 0. ORDEM DE EXECUCAO (nao pule etapas)

| # | Etapa | Onde | Responsavel sugerido |
|---|-------|------|----------------------|
| 0 | Ativar licenca Basic + Test Plans | Organization settings > Billing | Quem administra a org |
| 1 | Reescrever os criterios de aceite em Gherkin | Boards > Work Items (cada PBI) | Compliance |
| 2 | Criar os casos pelo card do PBI (Add Test) | Boards > Boards | Compliance |
| 3 | Escrever os Steps de cada caso | Test Plans > Edit test case | Compliance |
| 4 | Executar no Test Runner | Test Plans > Execute | Todos (para o grafico "Run by") |
| 5 | Registrar bug do caso que falhar | Test Runner > Create bug | Quem executou |
| 6 | Criar os 2 graficos | Test Plans > Chart | Compliance |
| 7 | Conferir contador no card | Boards > Boards | Compliance |

**Atencao na etapa 4:** o grafico "Run by" so faz sentido se mais de uma pessoa executar.
Divida os casos entre pelo menos 2 integrantes.

---

# 1. CRITERIOS DE ACEITE EM GHERKIN

Abra cada PBI em **Boards > Work Items**, campo **Acceptance Criteria**, e substitua o texto
atual pelos cenarios abaixo. Cada cenario vira um caso de teste na secao 2.

---

## PBI: Autenticacao e autorizacao com Spring Security

```
CENARIO 1 - Login com credenciais validas
DADO um usuario cadastrado com e-mail e senha validos
QUANDO ele enviar POST /api/auth/login com essas credenciais
ENTAO o sistema retorna 200 e um token JWT valido

CENARIO 2 - Login com senha incorreta
DADO um usuario cadastrado
QUANDO ele enviar POST /api/auth/login com a senha errada
ENTAO o sistema retorna 401 e nao devolve token

CENARIO 3 - Rota protegida sem token
DADO um cliente sem autenticacao
QUANDO ele chamar GET /api/pets sem o header Authorization
ENTAO o sistema retorna 401

CENARIO 4 - Perfil sem permissao na rota
DADO um usuario autenticado com perfil TUTOR
QUANDO ele chamar GET /api/tutores, rota exclusiva de VETERINARIO
ENTAO o sistema retorna 403 e nao expoe a lista de tutores
```

---

## PBI: CRUD de pets na API

```
CENARIO 1 - Cadastro de pet valido
DADO um tutor autenticado
QUANDO ele enviar POST /api/pets com nome, especie e data de nascimento validos
ENTAO o sistema retorna 201 e o pet fica vinculado ao tutor autenticado

CENARIO 2 - Cadastro sem campo obrigatorio
DADO um tutor autenticado
QUANDO ele enviar POST /api/pets sem o campo nome
ENTAO o sistema retorna 400 e informa qual campo esta invalido

CENARIO 3 - Acesso a pet de outro tutor
DADO dois tutores distintos, cada um com seu pet
QUANDO o tutor A chamar GET /api/pets/{id} usando o id do pet do tutor B
ENTAO o sistema retorna 403 e nao expoe os dados do pet

CENARIO 4 - Pet inexistente
DADO um tutor autenticado
QUANDO ele chamar GET /api/pets/{id} com um id que nao existe
ENTAO o sistema retorna 404
```

---

## PBI: Cadastro de vacina com data de reforco

```
CENARIO 1 - Registro de vacina completo
DADO um pet cadastrado e um veterinario autenticado
QUANDO ele enviar POST /api/eventos com tipo VACINA, data de aplicacao e data de reforco
ENTAO o sistema retorna 201 e grava as duas datas

CENARIO 2 - Vacina sem data de aplicacao
DADO um pet cadastrado e um veterinario autenticado
QUANDO ele enviar POST /api/eventos do tipo VACINA sem data de aplicacao
ENTAO o sistema retorna 400 e nao grava o registro
```

---

## PBI: Score Engine v1

```
CENARIO 1 - Score dentro da faixa valida
DADO um pet com vacinas e consultas registradas
QUANDO o score for consultado em GET /api/pets/{id}/score
ENTAO o sistema retorna 200 e um numero inteiro entre 0 e 100

CENARIO 2 - Score de pet de outro tutor
DADO dois tutores distintos
QUANDO o tutor A consultar o score do pet do tutor B
ENTAO o sistema retorna 403
```

---

## PBI: Listagem de pets ordenada por score

```
CENARIO 1 - Listagem retorna apenas os pets do usuario
DADO um tutor autenticado com 2 pets e outro tutor com 1 pet
QUANDO o tutor A chamar GET /api/pets
ENTAO o sistema retorna 200 e apenas os 2 pets dele

CENARIO 2 - Ordenacao por score crescente
DADO pets com scores diferentes
QUANDO o painel chamar GET /api/pets?sort=score,asc
ENTAO o sistema retorna 200 e os pets ordenados do menor para o maior score
```

---

# 2. CASOS DE TESTE

Total: **14 casos** em **5 suites**.
Criacao: card do PBI > tres pontos > **Add Test** > digitar o titulo > Enter.
Depois: Test Plans > passar o mouse no caso > tres pontos > **Edit test case** > preencher Steps.

---

## SUITE 1 - Autenticacao e autorizacao com Spring Security

### [200] Login com credenciais validas retorna token

| N | Action | Expected result |
|---|--------|-----------------|
| 1 | Enviar POST para @baseUrl/api/auth/login com Content-Type application/json | Requisicao aceita pelo servidor |
| 2 | Informar no corpo o e-mail @email e a senha @senha | Corpo da requisicao montado com as credenciais |
| 3 | Executar a requisicao | Resposta com status 200 |
| 4 | Inspecionar o corpo da resposta | Campo token presente, em formato JWT com tres blocos separados por ponto |

**Parameter values:** baseUrl = `http://localhost:8080` | email = `vet.teste@clyvo.com` | senha = `Senha@123`

---

### [401] Login com senha incorreta e recusado

| N | Action | Expected result |
|---|--------|-----------------|
| 1 | Enviar POST para @baseUrl/api/auth/login | Requisicao aceita pelo servidor |
| 2 | Informar o e-mail @email e a senha @senhaErrada | Corpo da requisicao montado |
| 3 | Executar a requisicao | Resposta com status 401 |
| 4 | Inspecionar o corpo da resposta | Nenhum token e retornado e a mensagem nao revela se o e-mail existe |

**Parameter values:** baseUrl = `http://localhost:8080` | email = `vet.teste@clyvo.com` | senhaErrada = `SenhaErrada999`

---

### [401] Requisicao sem token e recusada

| N | Action | Expected result |
|---|--------|-----------------|
| 1 | Enviar GET para @baseUrl/api/pets sem o header Authorization | Requisicao aceita pelo servidor |
| 2 | Executar a requisicao | Resposta com status 401 |
| 3 | Inspecionar o corpo da resposta | Nenhum dado de pet e retornado |

**Parameter values:** baseUrl = `http://localhost:8080`

---

### [403] Tutor acessa rota exclusiva de veterinario e bloqueado

| N | Action | Expected result |
|---|--------|-----------------|
| 1 | Autenticar com o usuario @emailTutor e guardar o token retornado | Status 200 e token obtido |
| 2 | Enviar GET para @baseUrl/api/tutores com o header Authorization Bearer e o token do tutor | Requisicao aceita pelo servidor |
| 3 | Executar a requisicao | Resposta com status 403 |
| 4 | Inspecionar o corpo da resposta | A lista de tutores nao e retornada |

**Parameter values:** baseUrl = `http://localhost:8080` | emailTutor = `tutor.a@clyvo.com`

---

## SUITE 2 - CRUD de pets na API

### [201] Pet valido e cadastrado

| N | Action | Expected result |
|---|--------|-----------------|
| 1 | Autenticar com o tutor @emailTutor e guardar o token | Status 200 e token obtido |
| 2 | Enviar POST para @baseUrl/api/pets com o token no header Authorization | Requisicao aceita pelo servidor |
| 3 | Informar no corpo nome @nomePet, especie @especie e data de nascimento @nascimento | Corpo da requisicao montado |
| 4 | Executar a requisicao | Resposta com status 201 e o corpo contem o id gerado |
| 5 | Consultar GET @baseUrl/api/pets com o mesmo token | O pet @nomePet aparece na lista do tutor autenticado |

**Parameter values:** baseUrl = `http://localhost:8080` | emailTutor = `tutor.a@clyvo.com` | nomePet = `Luna` | especie = `CANINO` | nascimento = `2021-03-15`

---

### [400] Pet sem nome e recusado

| N | Action | Expected result |
|---|--------|-----------------|
| 1 | Autenticar com o tutor @emailTutor e guardar o token | Status 200 e token obtido |
| 2 | Enviar POST para @baseUrl/api/pets omitindo o campo nome | Requisicao aceita pelo servidor |
| 3 | Executar a requisicao | Resposta com status 400 |
| 4 | Inspecionar o corpo da resposta | A mensagem identifica o campo nome como invalido |
| 5 | Consultar GET @baseUrl/api/pets | Nenhum pet novo foi criado |

**Parameter values:** baseUrl = `http://localhost:8080` | emailTutor = `tutor.a@clyvo.com`

---

### [403] Tutor acessa pet de outro tutor e bloqueado

| N | Action | Expected result |
|---|--------|-----------------|
| 1 | Autenticar com o tutor @emailTutorB e cadastrar um pet, guardando o id retornado | Status 201 e id do pet obtido |
| 2 | Autenticar com o tutor @emailTutorA e guardar o token dele | Status 200 e token obtido |
| 3 | Enviar GET para @baseUrl/api/pets/@idPetB usando o token do tutor A | Requisicao aceita pelo servidor |
| 4 | Executar a requisicao | Resposta com status 403 |
| 5 | Inspecionar o corpo da resposta | Nenhum dado do pet do tutor B e exposto |

**Parameter values:** baseUrl = `http://localhost:8080` | emailTutorA = `tutor.a@clyvo.com` | emailTutorB = `tutor.b@clyvo.com` | idPetB = `(id obtido no passo 1)`

> Este caso e o que reproduz a falha apontada no feedback da Sprint 3. Se a API em teste
> ainda retornar 200 aqui, marque X vermelho e abra o bug (secao 3).

---

### [404] Pet inexistente retorna nao encontrado

| N | Action | Expected result |
|---|--------|-----------------|
| 1 | Autenticar com o tutor @emailTutor e guardar o token | Status 200 e token obtido |
| 2 | Enviar GET para @baseUrl/api/pets/@idInexistente com o token | Requisicao aceita pelo servidor |
| 3 | Executar a requisicao | Resposta com status 404 |
| 4 | Inspecionar o corpo da resposta | A mensagem indica recurso nao encontrado, sem stack trace |

**Parameter values:** baseUrl = `http://localhost:8080` | emailTutor = `tutor.a@clyvo.com` | idInexistente = `999999`

---

## SUITE 3 - Cadastro de vacina com data de reforco

### [201] Vacina com data de aplicacao e reforco e registrada

| N | Action | Expected result |
|---|--------|-----------------|
| 1 | Autenticar com o veterinario @emailVet e guardar o token | Status 200 e token obtido |
| 2 | Enviar POST para @baseUrl/api/eventos com o token no header | Requisicao aceita pelo servidor |
| 3 | Informar petId @idPet, tipo VACINA, data de aplicacao @dataAplicacao e reforco @dataReforco | Corpo da requisicao montado |
| 4 | Executar a requisicao | Resposta com status 201 e id do evento gerado |
| 5 | Consultar GET @baseUrl/api/pets/@idPet/eventos | O evento aparece com as duas datas gravadas |

**Parameter values:** baseUrl = `http://localhost:8080` | emailVet = `vet.teste@clyvo.com` | idPet = `(id de um pet existente)` | dataAplicacao = `2026-10-01` | dataReforco = `2027-10-01`

---

### [400] Vacina sem data de aplicacao e recusada

| N | Action | Expected result |
|---|--------|-----------------|
| 1 | Autenticar com o veterinario @emailVet e guardar o token | Status 200 e token obtido |
| 2 | Enviar POST para @baseUrl/api/eventos do tipo VACINA omitindo a data de aplicacao | Requisicao aceita pelo servidor |
| 3 | Executar a requisicao | Resposta com status 400 |
| 4 | Consultar GET @baseUrl/api/pets/@idPet/eventos | Nenhum evento novo foi gravado |

**Parameter values:** baseUrl = `http://localhost:8080` | emailVet = `vet.teste@clyvo.com` | idPet = `(id de um pet existente)`

---

## SUITE 4 - Score Engine v1

### [200] Score do pet esta entre 0 e 100

| N | Action | Expected result |
|---|--------|-----------------|
| 1 | Autenticar com o tutor @emailTutor e guardar o token | Status 200 e token obtido |
| 2 | Enviar GET para @baseUrl/api/pets/@idPet/score com o token | Requisicao aceita pelo servidor |
| 3 | Executar a requisicao | Resposta com status 200 |
| 4 | Inspecionar o valor retornado | O score e um numero inteiro maior ou igual a 0 e menor ou igual a 100 |

**Parameter values:** baseUrl = `http://localhost:8080` | emailTutor = `tutor.a@clyvo.com` | idPet = `(id de um pet do tutor A)`

---

### [403] Score de pet de outro tutor e bloqueado

| N | Action | Expected result |
|---|--------|-----------------|
| 1 | Autenticar com o tutor @emailTutorA e guardar o token | Status 200 e token obtido |
| 2 | Enviar GET para @baseUrl/api/pets/@idPetB/score usando o token do tutor A | Requisicao aceita pelo servidor |
| 3 | Executar a requisicao | Resposta com status 403 |
| 4 | Inspecionar o corpo da resposta | Nenhum score do pet do tutor B e exposto |

**Parameter values:** baseUrl = `http://localhost:8080` | emailTutorA = `tutor.a@clyvo.com` | idPetB = `(id de um pet do tutor B)`

---

## SUITE 5 - Listagem de pets ordenada por score

### [200] Listagem retorna apenas os pets do usuario autenticado

| N | Action | Expected result |
|---|--------|-----------------|
| 1 | Garantir que o tutor @emailTutorA tem @qtdPetsA pets e o tutor @emailTutorB tem outros pets | Base preparada com pets de dois tutores |
| 2 | Autenticar com o tutor @emailTutorA e guardar o token | Status 200 e token obtido |
| 3 | Enviar GET para @baseUrl/api/pets com o token do tutor A | Resposta com status 200 |
| 4 | Contar os itens retornados | A lista traz exatamente @qtdPetsA itens |
| 5 | Conferir os nomes retornados | Nenhum pet do tutor B aparece na lista |

**Parameter values:** baseUrl = `http://localhost:8080` | emailTutorA = `tutor.a@clyvo.com` | emailTutorB = `tutor.b@clyvo.com` | qtdPetsA = `2`

---

### [200] Listagem vem ordenada por score crescente

| N | Action | Expected result |
|---|--------|-----------------|
| 1 | Autenticar com o usuario @emailVet e guardar o token | Status 200 e token obtido |
| 2 | Enviar GET para @baseUrl/api/pets?sort=score,asc com o token | Resposta com status 200 |
| 3 | Ler os scores na ordem em que vieram | Cada score e maior ou igual ao anterior |
| 4 | Conferir o primeiro item da lista | O pet de menor score aparece em primeiro lugar |

**Parameter values:** baseUrl = `http://localhost:8080` | emailVet = `vet.teste@clyvo.com`

---

# 3. REGISTRO DE BUG

O tutorial exige pelo menos um bug no fluxo (slide 15 e checkpoint do slide 20).
No Test Runner, no passo que falhar:

1. Marcar o **X vermelho** no passo exato que falhou (nao no caso inteiro).
2. Escrever no campo COMMENT o que aconteceu de fato, com o valor observado.
   Exemplo: `Esperado 403. Retornou 200 com o corpo completo do pet do tutor B.`
3. Clicar em **Create bug**, anexar o print da resposta e salvar.

Titulo sugerido do bug:
`[403 esperado] GET /api/pets/{id} devolve 200 para pet de outro tutor`

Descricao do bug:
```
Passos: ver caso de teste [403] Tutor acessa pet de outro tutor e bloqueado
Resultado esperado: 403 Forbidden
Resultado obtido: 200 OK com o corpo completo do pet
Impacto: quebra de isolamento entre tutores, exposicao de dado clinico de terceiro
Severidade: 1 - Critical
```

**Se todos os 14 casos passarem na primeira execucao**, voce fica sem bug e perde esse item
do checkpoint. Nesse caso, execute o caso [403] Tutor acessa pet de outro tutor contra a
branch anterior a correcao (sprint-03), registre o bug, depois volte para a branch corrigida,
re-execute e marque Passed. Esse e o ciclo real de QA: bug encontrado, corrigido e reteste.

---

# 4. GRAFICOS

Na suite, aba **Chart** > **New** > **New test result chart**:

| Grafico | Chart Type | Group by |
|---------|-----------|----------|
| 1 | Pie | Outcome |
| 2 | Pie | Run by |

O grafico "Run by" so fica util se mais de uma pessoa executar casos. Divida assim:

| Integrante | Suites |
|------------|--------|
| Eduarda | Suite 1 e Suite 2 |
| Samara | Suite 3 e Suite 4 |
| Camily | Suite 5 |

---

# 5. CHECKLIST FINAL

- [ ] Licenca Basic + Test Plans ativa para quem escreve os casos
- [ ] Criterios de aceite dos 5 PBIs reescritos em DADO/QUANDO/ENTAO
- [ ] 14 casos criados pelo card do PBI (Add Test), nao direto no Test Plans
- [ ] Todo titulo comeca por [status code]
- [ ] Todo passo tem Action e Expected result preenchidos
- [ ] Nenhum valor fixo escrito no texto do passo, tudo em parametro @
- [ ] Save & Close em cada caso (sem isso o roteiro some)
- [ ] 14 casos executados no Test Runner, passo a passo
- [ ] Ao menos 1 bug aberto com comentario e print
- [ ] Grafico Pie por Outcome criado
- [ ] Grafico Pie por Run by criado
- [ ] Contador de testes visivel no card de cada PBI no board
- [ ] Pelo menos 2 integrantes como executores
