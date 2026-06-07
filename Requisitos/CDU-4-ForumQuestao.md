# Especificação de Requisitos Funcionais

## Caso de Uso (CDU) - (Sistema ENADE Comentado - SEC)

## Histórico de Versões

| Data       | Versão | Descrição                                                                                          | Autor         |
| ---------- | ------- | -------------------------------------------------------------------------------------------------- | ------------- |
| 28/05/2026 | 1.0     | Criação inicial do caso de uso com base nos requisitos F2.3 e F3.2 da Visão da Demanda e na RN3.   | Juliana        |
| 07/06/2026 | 1.1     | Ajustes gerais.   | Juliana        |


## 1. Nome do Caso de Uso

Acessar fórum de discussão por questão

## 2. Objetivo

Permitir que o Aluno Concluinte interaja, tire dúvidas e debata em um espaço colaborativo sobre questões específicas que ele já tenha respondido em simulados, e possibilitar ao Professor atuar como tutor, respondendo a esses questionamentos exclusivamente nas questões que ele mesmo cadastrou na plataforma.

## 3. Tipo de Caso de Uso

Concreto.

## 4. Atores

### 4.1 Primário

| Ator | Descrição |
|---|---|
| Aluno Concluinte | Acessa o fórum de discussões para ler interações e postar dúvidas sobre as questões que já respondeu |
| Professor | Atua como tutor especializado, respondendo e orientando os alunos nos fóruns das questões criadas por ele |

### 4.2 Secundário

| Ator | Descrição |
|---|---|
| SECBackEndAPI | Responsável por validar as regras de escopo de acesso ao fórum, processar e servir as postagens em formato JSON |
| PostgreSQL (SEC_BD) | Armazena o histórico imutável das postagens, respostas, sinalizações e os vínculos de autoria de questões |

## 5. Precondições

| Código | Descrição |
|---|---|
| PRE01 | Ambos os atores devem estar previamente autenticados no sistema com uma sessão ativa (`F4.1` / `RNF-017`) |
| PRE02 | **[Para o Aluno Concluinte]**: Deve ter obrigatoriamente submetido e respondido à respectiva questão em um simulado anterior (`RN3` / `POS03` do CDU Realizar Simulados) |
| PRE03 | **[Para o Professor]**: Deve ser formalmente o autor/cadastrador original da questão vinculada ao fórum de discussão solicitado (`RN3`) |

## 6. Fluxo Principal

### P1. Solicitar acesso ao fórum da questão

O ator (Aluno ou Professor) navega pela plataforma e aciona o comando "Acessar Fórum" (o Aluno a partir da tela de revisão de suas provas; o Professor a partir do seu painel de gerenciamento de itens cadastrados).

### P2. Validar perfil e regras de escopo de conteúdo

A API backend intercepta a requisição segura HTTPS (`RNF-013`), decodifica o token JWT e valida as travas de negócio (`RN3`):
* Se o ator for Aluno, o sistema confirma se a questão está marcada como "Respondida" no histórico dele.
* Se o ator for Professor, o sistema confirma se o ID do professor coincide com o criador da questão.

### P3. Carregar o histórico de discussões do item

Após a validação bem-sucedida, o backend extrai a árvore de comentários do PostgreSQL. A interface em React monta a tela renderizando o enunciado resumido da questão e a listagem cronológica de tópicos em menos de 3 segundos (`RNF-002`).

### P4. Visualizar tópicos e asserções tutoriais

O ator lê as postagens organizadas de maneira limpa (`RNF-029`) e com total suporte a tecnologias assistivas (`RNF-027`). Os comentários enviados por professores ganham uma estilização de destaque visual indicando a autoria tutorial oficial.

### P5. Postar uma nova mensagem

O ator digita uma nova dúvida, comentário ou esclarecimento técnico na caixa de texto enriquecido e aciona o comando "Enviar Comentário".

### P6. Persistir e sincronizar a interação

A solução processa a mensagem text-only via JSON (`RNF-035`), sanitiza strings contra injeções maliciosas, persiste de forma permanente no banco de dados (`PostgreSQL`) e atualiza instantaneamente a linha do tempo do fórum sem necessidade de recarga de página.

## 7. Fluxos Alternativos

### A1. Professor responde a uma dúvida específica de aluno

#### A1.1. No passo P5 do fluxo principal, o Professor opta por responder diretamente a um comentário postado por um Aluno Concluinte.
#### A1.2. O Professor clica no comando "Responder" posicionado logo abaixo do card do aluno discente.
#### A1.3. O sistema cria um recuo visual (Thread/Aninhamento) atrelando a resposta do Professor à dúvida inicial e retorna ao passo P6.

## 8. Fluxos de Exceção

### E1. Tentativa de acesso forçado ou não autorizado (Violação de Escopo)

#### E1.1. No passo P2, a validação do backend identifica que o Aluno tenta acessar via URL direta o fórum de uma questão não respondida, OU que o Professor tenta acessar o fórum de uma questão cadastrada por um colega terceiro.
#### E1.2. O sistema bloqueia a execução da consulta no PostgreSQL para proteger a integridade dos dados e regras de negócio.
#### E1.3. A interface apresenta uma mensagem de erro restritiva (`MSG001`) e redireciona o usuário ao seu respectivo painel principal.

### E2. Falha de envio por timeout ou oscilação de internet

#### E2.1. No decorrer do passo P6, o usuário perde comunicação ativa com o servidor web durante o envio da mensagem.
#### E2.2. O sistema aborta o carregamento infinito do botão de envio e retém o texto digitado pelo usuário na caixa de escrita localmente (`RNF-011`).
#### E2.3. A interface exibe um alerta de indisponibilidade momentânea (`MSG002`), mantendo o espaço habilitado para nova tentativa assim que o sinal HTTPS se reestabelecer.

## 9. Pós-condições

| Código | Descrição |
|---|---|
| POS01 | O novo comentário/resposta é indexado de forma permanente no PostgreSQL associado ao ID único da questão |

## 10. Requisitos Não Funcionais

| Código | Requisito |
|---|---|
| RNF01 | A renderização completa da linha do tempo do fórum e carregamento das mensagens não deve exceder 3 segundos (`RNF-002`) |
| RNF02 | O consumo de memória RAM da aba do fórum de discussão em React não deve ultrapassar 150MB no cliente (`RNF-005`) |
| RNF03 | O layout das discussões deve se adaptar perfeitamente a dispositivos móveis a partir de 360px de largura (`RNF-007`) |
| RNF04 | O isolamento de acesso determinado pela Regra de Negócio 3 (RN3) deve ser validado estritamente no lado do servidor (`RNF-017`) |
| RNF05 | Toda a árvore de comentários deve conter marcas e descritores HTML acessíveis (WCAG 2.1) para leitura de tela (`RNF-027`) |
| RNF06 | O design deve mitigar poluição visual, mantendo avatares discretos e separação nítida entre tópicos (`RNF-029`) |
| RNF07 | Comunicação baseada na transmissão leve de pacotes JSON para garantir rapidez sob redes móveis (`RNF-035`) |

## 11. Ponto de Extensão

Não se aplica.

## 12. Frequência de Utilização

Alta. Utilizado continuamente por alunos após a consolidação de simulados para sanar dúvidas recorrentes e por professores nos períodos semanais de tutoria acadêmica.

# 13. Interface Visual

## IV1. Tela de Fórum de Discussões por Item Acadêmico

### Layout da Tela

Interface limpa focada em conversação e legibilidade de textos longos. Apresenta o cabeçalho descritivo da questão fixado no topo, seguido por uma linha do tempo vertical de comentários aninhados e, na base da página, a caixa de inserção de texto.

---

## 13.1 Campos da Interface

| Campo | Tipo/Formato | Obrigatório | Descrição | Regra de Negócio / RNF |
|---|---|---|---|---|
| Identificador da Questão | Rótulo / Texto | Sim | Código numérico e área ENADE da questão consultada | Exemplo: "Fórum - Item #4102 [Conhecimento Específico]" |
| Mini Enunciado | Texto (Reduzido) | Sim | Primeiras linhas do texto base/enunciado para contextualização | Apenas leitura para guiar o ator |
| Feed de Comentários | Componente Lista | Não | Stream vertical exibindo avatares, nomes, perfis e textos | Suporte completo a leitores de tela (`RNF-027`) |
| Badge "Tutor" | Elemento Visual | Não | Selo colorido aplicado apenas ao card de postagem do Professor | Identifica respostas oficiais da mentoria acadêmica |
| Caixa de Mensagem | Área de Texto | Sim | Campo de digitação de texto simples ou Markdown básico | Limite máximo de 2000 caracteres por postagem |
| Botão "Enviar Comentário"| Botão | Sim | Submete a mensagem digitada para validação e persistência | Salva de forma incremental e segura (`RNF-013`) |
| Botão "Voltar" | Botão | Sim | Retorna o usuário à sua tela de origem (Simulados ou Itens) | Não descarta o progresso de leitura da página |

---

## 13.2 Navegabilidade

| Ação | Resultado |
|---|---|
| Clicar em "Acessar Fórum" na revisão do simulado | O sistema valida se o aluno respondeu à questão e abre a tela carregando as postagens em até 3 segundos (`RNF-002`) |
| Professor acessar fórum de questão alheia | O sistema intercepta o redirecionamento, barra a requisição no backend e mostra aviso crítico (`E1`) |
| Digitar texto e clicar em "Enviar Comentário" | A mensagem é enviada via API, limpa o campo de texto local e plota a nova linha na base do feed em tempo real |
| Clicar em "Responder" num comentário discente | O cursor foca automaticamente na caixa de texto mapeando aquela ID como nó-pai do novo comentário (`A1`) |

---

## 13.3 Mensagens Previstas

| Código | Mensagem |
|---|---|
| MSG001 | Acesso Negado. Você não possui permissões regulamentares para visualizar o fórum desta questão (RN3). |
| MSG002 | Não foi possível enviar sua mensagem. Verifique sua conexão de rede e tente novamente. |
| MSG003 | Publicando seu comentário... |
| MSG004 | Nenhuma discussão iniciada para esta questão ainda. Seja o primeiro a expor suas dúvidas! |

---

## 13.4 Componentes Visuais

| Área | Componente |
|---|---|
| Topo fixado (Header) | Barra cinza claro com tipografia em negrito contendo os metadados da questão e botão minimalista de retorno |
| Linha do tempo (Timeline) | Lista encadeada com cards brancos de bordas arredondadas. O card do Professor possui borda verde-esmeralda distintiva e tag "Tutor" |
| Bloco de entrada (Input Area) | Caixa cinza-escura posicionada na base contendo o editor de texto adaptável (Textarea auto-expandível) e botão de envio lateral |

---

## 14. Observações

Não se aplica.

## 15. Referências

* Visão da Demanda (VD) - Seção 6 (Funcionalidades F2.3, F3.2 e F4.1).
* Regras de Negócio da Solução (RN) - Identificador RN3 (Vínculo de Fórum e Isolamento de Escopo por Autoria/Resposta).
* Especificação de Requisitos Não Funcionais (RNF) - Itens RNF-002, RNF-005, RNF-007, RNF-011, RNF-013, RNF-017, RNF-027, RNF-029 e RNF-035.
* Diagrama de Caso de Uso (`../Imagens/diagrama_casos_de_uso_v3.png`).
