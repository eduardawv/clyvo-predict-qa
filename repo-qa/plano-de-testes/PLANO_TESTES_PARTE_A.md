# Plano de Testes Manuais — CLYVO Predict — Sprint 4
## Parte A | Azure Boards + Azure Test Plans

> Formato do tutorial "Plano de testes no Azure — passo a passo" (Prof. Douglas Andrade).
> Regras: **um caso de teste por critério de aceite**, título iniciado pelo **status code**,
> dado de teste em **parâmetro @**, cada passo com **resultado esperado**.
>
> Endpoints, corpos e códigos de resposta conferidos no código da branch `sprint-04`.

---

# 0. ANTES DE COMEÇAR

## Onde cada coisa acontece

| Etapa | Onde | Precisa da API rodando? |
|-------|------|-------------------------|
| Reescrever critérios em Gherkin | Azure, qualquer computador | Não |
| Criar os casos e escrever os passos | Azure, qualquer computador | Não |
| Executar os casos | Azure + Swagger, **no computador onde a API roda** | **Sim** |

A API roda em `http://localhost:8080`, que só existe no computador da Eduarda.
Quem for executar precisa estar nesse computador. Para o gráfico "Run by" mostrar mais
de um nome, cada pessoa entra no Azure com a própria conta, numa janela anônima do navegador.

## Dados de teste (criados no Bloco 0.3 do guia)

| Usuário | E-mail | Senha |
|---------|--------|-------|
| Tutor A | tutor.a@clyvo.com | Senha@123 |
| Tutor B | tutor.b@clyvo.com | Senha@123 |
| Veterinário | vet.teste@clyvo.com | Senha@123 |

| Pet | Dono | ID (anotar) |
|-----|------|-------------|
| Luna | Tutor A | ____ |
| Thor | Tutor A | ____ |
| Mel | Tutor B | ____ |

## Como executar cada passo no Swagger

- **Login e autorização:** chamar o endpoint de login, copiar o valor de `token`, clicar em
  **Authorize**, colar e confirmar. Para trocar de usuário: **Authorize → Logout** e repetir.
- **Chamar um endpoint:** expandir → **Try it out** → preencher → **Execute** → conferir
  **Server response**.

## Ordem de execução

Suíte 1 → 2 → 3 → 4 → 5. Nenhum caso depende de outro, mas essa ordem facilita a leitura dos resultados.

## Dois casos devem falhar — e isso é esperado

A análise do código encontrou dois defeitos reais. Os casos abaixo vão reproduzi-los:

- `[401] Login com senha incorreta é recusado` — a API devolve 400
- `[200] Listagem ordenada por score crescente` — a ordenação é ignorada

**Não corrijam esses defeitos antes de executar.** O ciclo de QA é: executar → falhar →
abrir bug → corrigir → reexecutar. Detalhes na seção 3.

---

# 1. CRITÉRIOS DE ACEITE EM GHERKIN

Para cada PBI abaixo: **Boards → Work Items** → abrir o PBI → campo **Acceptance Criteria**
→ apagar o texto atual → colar o bloco → **Save**.

O Gherkin vai **no PBI**. O caso de teste usa Action / Expected result.

---

## PBI `Autenticacao e autorizacao com Spring Security`

```
CENÁRIO 1 - Login com credenciais válidas
DADO um tutor cadastrado com e-mail e senha válidos
QUANDO ele enviar POST /api/tutores/login com essas credenciais
ENTÃO o sistema retorna 200 e um token JWT

CENÁRIO 2 - Login com senha incorreta
DADO um tutor cadastrado
QUANDO ele enviar POST /api/tutores/login com a senha errada
ENTÃO o sistema retorna 401 e não devolve token

CENÁRIO 3 - Rota protegida sem token
DADO um cliente sem autenticação
QUANDO ele chamar GET /api/pets sem o header Authorization
ENTÃO o sistema retorna 401

CENÁRIO 4 - Perfil sem permissão na rota
DADO um usuário autenticado com perfil TUTOR
QUANDO ele chamar GET /api/tutores, rota exclusiva de VETERINARIO
ENTÃO o sistema retorna 403 e não expõe a lista de tutores
```

---

## PBI `CRUD de pets na API`

```
CENÁRIO 1 - Cadastro de pet válido
DADO um tutor autenticado
QUANDO ele enviar POST /api/pets com nome, espécie, idade e peso válidos
ENTÃO o sistema retorna 201, o pet nasce com score 100 e fica vinculado ao tutor autenticado

CENÁRIO 2 - Cadastro sem campo obrigatório
DADO um tutor autenticado
QUANDO ele enviar POST /api/pets sem o campo nome
ENTÃO o sistema retorna 400 e informa qual campo está inválido

CENÁRIO 3 - Acesso a pet de outro tutor
DADO dois tutores, cada um com seu pet
QUANDO o tutor B chamar GET /api/pets/{id} com o id de um pet do tutor A
ENTÃO o sistema retorna 403 e não expõe os dados do pet

CENÁRIO 4 - Pet inexistente
DADO um tutor autenticado
QUANDO ele chamar GET /api/pets/{id} com um id que não existe
ENTÃO o sistema retorna 404
```

---

## PBI `Recalculo automatico do score por evento`

```
CENÁRIO 1 - Evento clínico atualiza o score
DADO um pet cadastrado e um veterinário autenticado
QUANDO o veterinário registrar em POST /api/eventos um evento do tipo DOENCA_LEVE
ENTÃO o sistema retorna 201 e o score do pet cai 15 pontos na mesma operação

CENÁRIO 2 - Evento sem data
DADO um pet cadastrado e um veterinário autenticado
QUANDO ele enviar POST /api/eventos sem a data do evento
ENTÃO o sistema retorna 400 e o score do pet não muda
```

---

## PBI `Score Engine v1`

```
CENÁRIO 1 - Score não ultrapassa 100
DADO um pet com score 100
QUANDO for registrado um evento do tipo VACINA, que soma 10 pontos
ENTÃO o score do pet permanece 100

CENÁRIO 2 - Score nunca fica negativo
DADO um pet cadastrado
QUANDO forem registrados três eventos do tipo ACIDENTE, que subtraem 50 pontos cada
ENTÃO o score do pet chega a 0 e não fica negativo
```

---

## PBI `Listagem de pets ordenada por score`

```
CENÁRIO 1 - Tutor vê apenas os próprios pets
DADO dois tutores, cada um com seus pets
QUANDO o tutor A chamar GET /api/pets
ENTÃO o sistema retorna 200 somente com os pets do tutor A

CENÁRIO 2 - Ordenação por score
DADO um tutor com pets de scores diferentes
QUANDO ele listar os pets e em seguida chamar GET /api/pets?sort=healthScore,asc
ENTÃO o sistema retorna 200 com os pets do menor para o maior score
```

---

# 2. CASOS DE TESTE

**14 casos em 5 suítes.**
Criação: card do PBI → três pontos → **Add Test** → digitar o título → Enter.
Roteiro: Test Plans → três pontos no caso → **Edit test case** → preencher **Steps** →
preencher **Parameter values** → **Save & Close**.

Onde o valor do parâmetro for `ID da Luna`, `ID do Thor` ou `ID da Mel`, use o número
anotado na tabela da seção 0.

| Suíte (PBI no Azure) | Casos |
|----------------------|-------|
| Autenticacao e autorizacao com Spring Security | 4 |
| CRUD de pets na API | 4 |
| Recalculo automatico do score por evento | 2 |
| Score Engine v1 | 2 |
| Listagem de pets ordenada por score | 2 |

---

## SUÍTE 1 — Autenticação e autorização com Spring Security

### [200] Login com credenciais válidas retorna token

| N | Action | Expected result |
|---|--------|-----------------|
| 1 | Abrir no Swagger o endpoint POST /api/tutores/login e clicar em Try it out | Campo de corpo da requisição liberado para edição |
| 2 | Preencher o corpo com e-mail @emailTutor e senha @senha | Corpo exibido com as credenciais informadas |
| 3 | Clicar em Execute | Resposta com status 200 |
| 4 | Conferir o corpo da resposta | Campo token preenchido, com três blocos separados por ponto, e campo perfil igual a TUTOR |

**Parameter values:** emailTutor = `tutor.a@clyvo.com` · senha = `Senha@123`

---

### [401] Login com senha incorreta é recusado

| N | Action | Expected result |
|---|--------|-----------------|
| 1 | Abrir no Swagger o endpoint POST /api/tutores/login e clicar em Try it out | Campo de corpo da requisição liberado para edição |
| 2 | Preencher o corpo com e-mail @emailTutor e senha @senhaErrada | Corpo exibido com as credenciais informadas |
| 3 | Clicar em Execute | Resposta com status 401 |
| 4 | Conferir o corpo da resposta | Nenhum token retornado e a mensagem não revela se o e-mail existe |

**Parameter values:** emailTutor = `tutor.a@clyvo.com` · senhaErrada = `SenhaErrada999`

> **Resultado previsto: falha no passo 3** (a API devolve 400). O passo 4 passa — a mensagem
> é genérica e não há token. Marque X no passo 3, check no passo 4, e abra o Bug 1 (seção 3).

---

### [401] Requisição sem token é recusada

| N | Action | Expected result |
|---|--------|-----------------|
| 1 | No Swagger, clicar em Authorize, depois em Logout, e fechar a janela | Nenhum token configurado, cadeados abertos |
| 2 | Abrir GET /api/pets, clicar em Try it out e em Execute | Resposta com status 401 |
| 3 | Conferir o corpo da resposta | Nenhum dado de pet retornado |

**Parameter values:** este caso não usa dados de entrada.

---

### [403] Tutor acessa rota exclusiva de veterinário e é bloqueado

| N | Action | Expected result |
|---|--------|-----------------|
| 1 | Fazer login em POST /api/tutores/login com @emailTutor e @senha e copiar o token | Status 200 e token obtido |
| 2 | Clicar em Authorize, colar o token e confirmar | Swagger autorizado, cadeados fechados |
| 3 | Abrir GET /api/tutores, clicar em Try it out e em Execute | Resposta com status 403 |
| 4 | Conferir o corpo da resposta | Mensagem "Você não tem permissão para acessar este recurso." e nenhuma lista de tutores |

**Parameter values:** emailTutor = `tutor.a@clyvo.com` · senha = `Senha@123`

---

## SUÍTE 2 — CRUD de pets na API

### [201] Pet válido é cadastrado

| N | Action | Expected result |
|---|--------|-----------------|
| 1 | Fazer login como @emailTutor com a senha @senha e autorizar o Swagger com o token | Status 200 e Swagger autorizado |
| 2 | Abrir POST /api/pets e preencher nome @nomePet, espécie @especie, idade @idade e peso @peso | Corpo exibido com os dados do pet |
| 3 | Clicar em Execute | Status 201, id gerado e healthScore igual a 100 |
| 4 | Abrir GET /api/pets e clicar em Execute | O pet @nomePet aparece na lista com o tutorId do tutor autenticado |

**Parameter values:** emailTutor = `tutor.a@clyvo.com` · senha = `Senha@123` · nomePet = `Bob` · especie = `Cachorro` · idade = `4` · peso = `10.5`

---

### [400] Pet sem nome é recusado

| N | Action | Expected result |
|---|--------|-----------------|
| 1 | Fazer login como @emailTutor com a senha @senha e autorizar o Swagger com o token | Status 200 e Swagger autorizado |
| 2 | Abrir POST /api/pets e preencher espécie @especie, idade @idade e peso @peso, sem o campo nome | Corpo exibido sem o campo nome |
| 3 | Clicar em Execute | Resposta com status 400 |
| 4 | Conferir o corpo da resposta | Lista de erros com campo "nome" e mensagem "O nome do pet é obrigatório" |

**Parameter values:** emailTutor = `tutor.a@clyvo.com` · senha = `Senha@123` · especie = `Gato` · idade = `2` · peso = `3.5`

---

### [403] Tutor acessa pet de outro tutor e é bloqueado

| N | Action | Expected result |
|---|--------|-----------------|
| 1 | Fazer login como @emailTutorB com a senha @senha e autorizar o Swagger com o token | Status 200 e Swagger autorizado |
| 2 | Abrir GET /api/pets/{id} e informar o id @idPetTutorA | Campo id preenchido |
| 3 | Clicar em Execute | Resposta com status 403 |
| 4 | Conferir o corpo da resposta | Mensagem "Este pet não pertence ao tutor autenticado." e nenhum dado do pet |

**Parameter values:** emailTutorB = `tutor.b@clyvo.com` · senha = `Senha@123` · idPetTutorA = `ID da Luna`

> Este caso valida a correção da falha de controle de acesso apontada no feedback da Sprint 3.

---

### [404] Pet inexistente retorna não encontrado

| N | Action | Expected result |
|---|--------|-----------------|
| 1 | Fazer login como @emailTutor com a senha @senha e autorizar o Swagger com o token | Status 200 e Swagger autorizado |
| 2 | Abrir GET /api/pets/{id}, informar o id @idInexistente e clicar em Execute | Resposta com status 404 |
| 3 | Conferir o corpo da resposta | Mensagem "Pet não encontrado com o ID: @idInexistente", sem detalhes técnicos do erro |

**Parameter values:** emailTutor = `tutor.a@clyvo.com` · senha = `Senha@123` · idInexistente = `999999`

---

## SUÍTE 3 — Recálculo automático do score por evento

### [201] Evento de doença reduz o score do pet

| N | Action | Expected result |
|---|--------|-----------------|
| 1 | Fazer login em POST /api/veterinarios/login com @emailVet e @senha e autorizar o Swagger | Status 200 e Swagger autorizado |
| 2 | Abrir GET /api/pets/{id}, informar @idPet, executar e anotar o healthScore | Status 200 e healthScore exibido (100 na primeira execução) |
| 3 | Abrir POST /api/eventos com petId @idPet, tipoEvento @tipoEvento, descricao @descricao e dataEvento @dataEvento e executar | Status 201 e novoHealthScore igual ao valor anotado menos 15 |
| 4 | Abrir GET /api/pets/{id} com @idPet e executar | healthScore igual ao novoHealthScore do passo 3 |
| 5 | Abrir GET /api/eventos/pet/{petId} com @idPet e executar | O evento @tipoEvento aparece na lista |

**Parameter values:** emailVet = `vet.teste@clyvo.com` · senha = `Senha@123` · idPet = `ID do Thor` · tipoEvento = `DOENCA_LEVE` · descricao = `Otite leve` · dataEvento = `2026-11-01`

---

### [400] Evento sem data é recusado

| N | Action | Expected result |
|---|--------|-----------------|
| 1 | Fazer login em POST /api/veterinarios/login com @emailVet e @senha e autorizar o Swagger | Status 200 e Swagger autorizado |
| 2 | Abrir GET /api/pets/{id}, informar @idPet, executar e anotar o healthScore | Status 200 e healthScore exibido |
| 3 | Abrir POST /api/eventos com petId @idPet, tipoEvento @tipoEvento e descricao @descricao, sem dataEvento, e executar | Status 400 e lista de erros apontando o campo dataEvento |
| 4 | Abrir GET /api/pets/{id} com @idPet e executar | healthScore igual ao anotado no passo 2 |

**Parameter values:** emailVet = `vet.teste@clyvo.com` · senha = `Senha@123` · idPet = `ID do Thor` · tipoEvento = `VACINA` · descricao = `V10 anual`

---

## SUÍTE 4 — Score Engine v1

### [201] Vacina em pet com score 100 mantém o score em 100

| N | Action | Expected result |
|---|--------|-----------------|
| 1 | Fazer login em POST /api/veterinarios/login com @emailVet e @senha e autorizar o Swagger | Status 200 e Swagger autorizado |
| 2 | Abrir GET /api/pets/{id}, informar @idPet e executar | Status 200 e healthScore igual a 100 |
| 3 | Abrir POST /api/eventos com petId @idPet, tipoEvento @tipoEvento, descricao @descricao e dataEvento @dataEvento e executar | Status 201 e novoHealthScore igual a 100 |
| 4 | Abrir GET /api/pets/{id} com @idPet e executar | healthScore continua 100, sem ultrapassar o limite |

**Parameter values:** emailVet = `vet.teste@clyvo.com` · senha = `Senha@123` · idPet = `ID da Luna` · tipoEvento = `VACINA` · descricao = `V10 anual` · dataEvento = `2026-11-01`

---

### [201] Eventos graves não deixam o score negativo

| N | Action | Expected result |
|---|--------|-----------------|
| 1 | Fazer login em POST /api/veterinarios/login com @emailVet e @senha e autorizar o Swagger | Status 200 e Swagger autorizado |
| 2 | Abrir POST /api/eventos com petId @idPet, tipoEvento @tipoEvento, descricao @descricao e dataEvento @dataEvento e executar | Status 201 e novoHealthScore igual ao score anterior menos 50, nunca abaixo de 0 |
| 3 | Executar novamente a mesma requisição | Status 201 e novoHealthScore igual a 0 |
| 4 | Executar a mesma requisição pela terceira vez | Status 201 e novoHealthScore continua 0 |
| 5 | Abrir GET /api/pets/{id} com @idPet e executar | healthScore igual a 0, sem valor negativo |

**Parameter values:** emailVet = `vet.teste@clyvo.com` · senha = `Senha@123` · idPet = `ID da Mel` · tipoEvento = `ACIDENTE` · descricao = `Atropelamento` · dataEvento = `2026-11-01`

---

## SUÍTE 5 — Listagem de pets ordenada por score

### [200] Tutor lista apenas os próprios pets

| N | Action | Expected result |
|---|--------|-----------------|
| 1 | Fazer login como @emailTutorA com a senha @senha e autorizar o Swagger com o token | Status 200 e Swagger autorizado |
| 2 | Abrir GET /api/pets e executar | Status 200 com a lista de pets |
| 3 | Conferir o tutorId de cada pet da lista | Todos os pets têm o tutorId do tutor A |
| 4 | Procurar na lista o pet @nomePetTutorB | O pet @nomePetTutorB não aparece |

**Parameter values:** emailTutorA = `tutor.a@clyvo.com` · senha = `Senha@123` · nomePetTutorB = `Mel`

---

### [200] Listagem ordenada por score crescente

| N | Action | Expected result |
|---|--------|-----------------|
| 1 | Fazer login como @emailTutorA com a senha @senha e autorizar o Swagger com o token | Status 200 e Swagger autorizado |
| 2 | Abrir POST /api/eventos e registrar um evento @tipoEvento para o pet @idPetMenorScore, com descricao @descricao e dataEvento @dataEvento | Status 201 e score do pet reduzido |
| 3 | Abrir GET /api/pets sem preencher parâmetros e executar | Status 200 com os pets em ordem alfabética |
| 4 | Abrir GET /api/pets, adicionar no parâmetro sort o valor @ordenacao e executar | Status 200 |
| 5 | Conferir a ordem dos pets na resposta do passo 4 | O pet @idPetMenorScore aparece primeiro e cada healthScore é maior ou igual ao anterior |

**Parameter values:** emailTutorA = `tutor.a@clyvo.com` · senha = `Senha@123` · tipoEvento = `DOENCA_LEVE` · idPetMenorScore = `ID do Thor` · descricao = `Gastrite` · dataEvento = `2026-11-01` · ordenacao = `healthScore,asc`

> **Resultado previsto: falha no passo 5.** A resposta vem na mesma ordem alfabética do passo 3.
> Marque X no passo 5 e abra o Bug 2 (seção 3).

---

# 3. BUGS

No Test Runner, no passo que falhar:

1. Marcar o **X vermelho** no passo exato que falhou.
2. No campo COMMENT, escrever o que aconteceu de fato, com o valor observado.
3. Clicar em **Create bug**, anexar o print da resposta do Swagger e salvar.

---

## Bug 1

**Título:** `[401 esperado] Login com senha incorreta retorna 400`

```
Caso de teste: [401] Login com senha incorreta é recusado — passo 3
Resultado esperado: 401 Unauthorized, sem token
Resultado obtido: 400 Bad Request com o corpo "Erro: E-mail ou senha inválidos."
Impacto: o cliente não consegue distinguir credencial inválida de erro de
validação; contraria a semântica HTTP de autenticação.
Severidade: 3 - Medium
Causa: TutorService.autenticar e o login de VeterinarioService lançam
RuntimeException, que o TratadorDeErros converte em 400.
Correção sugerida: lançar new ResponseStatusException(HttpStatus.UNAUTHORIZED,
"E-mail ou senha inválidos."), que o TratadorDeErros já converte em 401.
```

Comentário no passo: `Esperado 401. Retornou 400 com "Erro: E-mail ou senha inválidos."`

---

## Bug 2

**Título:** `Ordenação de GET /api/pets é ignorada após uma listagem anterior`

```
Caso de teste: [200] Listagem ordenada por score crescente — passo 5
Resultado esperado: pets do menor para o maior healthScore
Resultado obtido: pets na mesma ordem alfabética da listagem do passo 3
Impacto: o painel da clínica pode exibir os pets fora da ordem de risco,
que é o objetivo do PBI.
Severidade: 2 - High
Causa: a chave do cache listaDePets em PetService.listar considera usuário,
nome, página e tamanho, mas não a ordenação. A segunda chamada recebe o
resultado guardado da primeira.
Correção sugerida: incluir + '_' + #paginacao.sort na chave do @Cacheable.
```

Comentário no passo: `Esperado Thor (menor score) primeiro. Lista veio em ordem alfabética, igual ao passo 3.`

---

## Depois dos bugs

Com os bugs abertos, a equipe de Java aplica as correções sugeridas. Se der tempo antes da
entrega, reexecutem os dois casos: eles passam a ser aprovados e os bugs podem ser fechados.
Antes de reexecutar, **tirem print dos dois gráficos** — eles mostram as falhas encontradas.

Se não der tempo de corrigir, a entrega continua completa: casos executados, 2 bugs abertos
com evidência e gráficos.

---

# 4. GRÁFICOS

Na suíte, aba **Chart** → **New** → **New test result chart**:

| Gráfico | Chart Type | Group by |
|---------|-----------|----------|
| 1 | Pie | Outcome |
| 2 | Pie | Run by |

---

# 5. CHECKLIST

- [ ] Licença Basic + Test Plans ativa para quem escreve os casos
- [ ] Critérios de aceite dos 5 PBIs em DADO / QUANDO / ENTÃO
- [ ] 14 casos criados pelo card do PBI (Add Test)
- [ ] Todo título começa por [status code]
- [ ] Todo passo com Action e Expected result
- [ ] Dados de entrada em parâmetro @, nunca escritos fixos no passo
- [ ] Save & Close em todos os 14
- [ ] 14 casos executados no Test Runner, passo a passo, com a API rodando
- [ ] Bug 1 e Bug 2 abertos com comentário e print
- [ ] Gráfico Pie por Outcome
- [ ] Gráfico Pie por Run by
- [ ] Print dos gráficos antes de qualquer reexecução
- [ ] Contador de testes visível no card de cada PBI no board
