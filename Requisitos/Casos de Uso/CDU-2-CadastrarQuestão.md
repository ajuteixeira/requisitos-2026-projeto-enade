# Especificação de Requisitos Funcionais

## Caso de Uso (CDU) - (Sistema ENADE Comentado - SEC)

## Histórico de Versões

| Data       | Versão | Descrição                                                                       | Autor   |
| ---------- | ------ | ------------------------------------------------------------------------------- | ------- |
| 26/05/2026 | 1.1    | Criação inicial do caso de uso com base no requisito F4.5 da Visão da Demanda.  | Juliana |
| 27/05/2026 | 1.2    | Inclusão da parte de identidade visual e outros ajustes.                        | Juliana |
| 07/06/2026 | 1.3    | Inclusão do Fluxo Alternativo A2 (Salvar Rascunho da Questão) e ajustes gerais. | Juliana |

## 1. Nome do Caso de Uso

Cadastro de questões por área e curso

<img src="../Imagens/Diagrama-CDU-2-CadastrarQuestão_v2.jpg" width="60%">

## 2. Objetivo

Permitir ao professor incluir e editar questões no acervo do sistema que sejam aderentes ao padrão oficial do INEP, vinculadas automaticamente ao seu respectivo curso de cadastro e restritas à subcategoria de área/tema selecionada, garantindo o preenchimento dos campos obrigatórios, a segurança contra injeção de dados e gerando logs imutáveis para fins de auditoria.

## 3. Tipo de Caso de Uso

Concreto.

## 4. Atores

### 4.1 Primário

| Ator      | Descrição                                                                                                 |
| --------- | --------------------------------------------------------------------------------------------------------- |
| Professor | Docente especialista que alimenta o banco de questões e realiza a manutenção de itens da sua área e curso |

### 4.2 Secundário

| Ator                | Descrição                                                                                                      |
| ------------------- | -------------------------------------------------------------------------------------------------------------- |
| SECBackEndAPI       | API Node.js responsável por validar as permissões via token JWT, sanitizar entradas e processar as requisições |
| PostgreSQL (SEC_BD) | SGBD Relacional responsável pela persistência das questões e gravação da trilha de auditoria                   |

## 5. Precondições

| Código | Descrição                                                                                                                       |
| ------ | ------------------------------------------------------------------------------------------------------------------------------- |
| PRE01  | O Professor deve estar autenticado no sistema com token JWT válido e perfil ativo de controle de acesso (F4.1 / RNF-017)        |
| PRE02  | O Professor deve possuir o aceite explícito do Termo de Consentimento da LGPD registrado no primeiro acesso (RNF-019)           |
| PRE03  | O Professor deve possuir um vínculo acadêmico pré-configurado com um curso e área específicos na carga inicial (F4.8 / RNF-021) |

## 6. Fluxo Principal

### P1. Solicitar acesso ao módulo de cadastro

O Professor acessa a plataforma web por meio de um navegador compatível (`RNF-008`), visualizando uma interface limpa com logotipos institucionais claros (`RNF-023` / `RNF-029`), e aciona o comando para cadastrar uma nova questão.

### P2. Exibir formulário de cadastro estruturado

A API Node.js na AWS processa a requisição via tráfego criptografado HTTPS TLS 1.3 (`RNF-013`), validando o token JWT do usuário. A solução renderiza a tela do formulário estruturado em React no cliente em menos de 3 segundos (`RNF-002`), **apresentando o campo "Curso" automaticamente preenchido e bloqueado para alterações**, trazendo apenas as subcategorias de área/tema associadas a este curso (`RNF-040`).

### P3. Preencher dados da questão

O Professor insere as informações obrigatórias utilizando componentes de UI reutilizáveis (`RNF-031`) que respeitam a acessibilidade WCAG 2.1 (`RNF-027`). Ele seleciona a **Subcategoria de Área/Tema** desejada (dentro das opções filtradas para o seu curso), define o Componente de Avaliação (`RNF-042`), preenche o texto base, adiciona imagens de suporte se necessário (`RNF-004`), o enunciado, as asserções e a justificativa técnica explicativa.

### P4. Solicitar gravação da nova questão

O Professor aciona o comando para salvar. O sistema intercepta o envio dos dados estruturados em formato JSON (`RNF-035`). A API backend valida o token JWT, sanitiza as entradas contra SQL Injection e XSS (`RNF-015`) e garante que a questão respeite estritamente a área e o curso vinculados ao perfil do professor autenticado (`RN2`).

### P5. Registrar questão e trilha de auditoria

A solução grava permanentemente a nova questão no banco PostgreSQL (`RNF-046`) em menos de 1,5 segundos (`RNF-001`). Simultaneamente, o sistema gera um log de auditoria imutável (`RNF-016` / `RN5`) vinculando o ID do usuário (extraído do token) e o timestamp do evento, exibindo uma notificação de sucesso na interface responsiva (`RNF-007`).

## 7. Fluxos Alternativos

### A1. Edição de questão existente

- **A1.1.** No passo P1 do fluxo principal, o Professor opta por gerenciar o acervo e seleciona uma questão de sua autoria para alteração.
- **A1.2.** O sistema valida os privilégios RBAC por meio do token JWT do professor (`RNF-017` / `RN2`) e carrega os dados atuais da questão em tela em menos de 1,5 segundos (`RNF-001`), mantendo o campo do Curso preenchido e completamente bloqueado.
- **A1.3.** O Professor altera a subcategoria ou os campos pedagógicos necessários e prossegue para o passo P4 do fluxo principal.

### A2. Salvar rascunho da questão

- **A2.1.** No passo P3 do fluxo principal, o Professor decide interromper a elaboração da questão (por falta de dados ou tempo) e aciona o botão **"Salvar Rascunho"**.
- **A2.2.** O sistema intercepta o envio e valida a autenticação (JWT), o escopo do curso (`RN2`) e sanitiza os dados informados até o momento (`RNF-015`), **ignorando intencionalmente as regras de validação de obrigatoriedade** dos campos pedagógicos estruturados do INEP.
- **A2.3.** A API persiste os dados parciais no banco de dados (`RNF-046`) atrelando a eles o status **"Incompleto"** ou **"Rascunho"**.
- **A2.4.** O sistema gera a respectiva trilha de auditoria da ação (`RNF-016`) e exibe a mensagem de sucesso correspondente em tela (`MSG002`), permitindo que o professor retome a edição futuramente através do fluxo A1.

## 8. Fluxos de Exceção

### E1. Estrutura padrão do INEP incompleta

- **E1.1.** No passo P4, o sistema detecta a ausência de campos obrigatórios exigidos pela estrutura do exame (`RN1` / `RNF-040`).
- **E1.2.** A solução bloqueia a persistência no banco SEC_BD para garantir a integridade do acervo.
- **E1.3.** O sistema exibe mensagens de erro claras, apontando visualmente o campo exato que precisa de correção (`RNF-028`), retornando ao passo P3.

### E2. Tentativa de violação de escopo de curso/área

- **E2.1.** No passo P4, a validação lógica do backend detecta que o identificador do curso enviado na requisição diverge da área de atuação configurada de forma fixa no token JWT do professor (`RN2` / `RNF-017`).
- **E2.2.** A API Node.js bloqueia imediatamente a operação e rejeita a gravação.
- **E2.3.** O sistema renderiza um alerta de violação de privilégios e retorna ao estado anterior sem modificar o banco de dados.

### E3. Queda de conexão ou instabilidade de rede

- **E3.1.** Durante o envio do formulário (passo P4), o sistema detecta uma perda de comunicação activa com os servidores da AWS Cloud (`RNF-011`).
- **E3.2.** A interface em React impede a perda do texto digitado pelo professor, mantendo os dados intactos no formulário local para proteção contra erros (`RNF-026` / `RNF-030`).
- **E3.3.** Uma notificação amigável de erro de rede é exibida na interface responsiva (`RNF-007` / `RNF-028`), permitindo reenvio assim que a conexão TLS 1.3 for reestabelecida.

### E4. Exclusão acidental de rascunhos de questões

- **E4.1.** Durante a manutenção de itens (Fluxo Alternativo A1), o Professor clica por engano no comando de exclusão.
- **E4.2.** O sistema intercepta o comando e impede a remoção imediata, exibindo obrigatoriamente um modal de confirmação de ação crítica (`RNF-026`).
- **E4.3.** Se o usuário cancelar, a questão é mantida intacta; se confirmar, a exclusão é processada e registrada na trilha de auditoria (`RNF-016`).

## 9. Pós-condições

| Código | Descrição                                                                                                                                 |
| ------ | ----------------------------------------------------------------------------------------------------------------------------------------- |
| POS01  | A questão é persistida de forma segura no PostgreSQL associada automaticamente às taxonomias oficiais do INEP/MEC (`RNF-042` / `RNF-046`) |
| POS02  | Uma trilha de auditoria imutável é gravada nativamente no banco de logs para fins de não repúdio (`RNF-016`)                              |
| POS03  | A questão torna-se disponível para visualização imediata da Coordenação de Curso no painel de acompanhamento (`F4.6`)                     |

## 10. Requisitos Não Funcionais

| Código | Requisito                                                                                                             |
| ------ | --------------------------------------------------------------------------------------------------------------------- |
| RNF01  | O tempo de resposta para salvar ou carregar uma questão editável deve ser inferior a 1,5 segundos (`RNF-001`)         |
| RNF02  | O carregamento inicial do formulário de cadastro limpo não deve ultrapassar 3 segundos (`RNF-002`)                    |
| RNF03  | O ecossistema de banco de dados deve ser dimensionado para comportar até 50.000 questões com mídias (`RNF-004`)       |
| RNF04  | A interface do formulário deve ser adaptável para telas a partir de 360px até desktops em 1080p (`RNF-007`)           |
| RNF05  | Todo o tráfego do formulário de cadastro deve utilizar criptografia segura via protocolo HTTPS TLS 1.3 (`RNF-013`)    |
| RNF06  | Prevenção ativa contra ataques de SQL Injection e XSS nos campos de texto da questão (`RNF-015`)                      |
| RNF07  | Exibição de caixas de diálogo de confirmação (Modais) antes da exclusão de qualquer item (`RNF-026`)                  |
| RNF08  | Interface em React aderente às diretrizes de acessibilidade WCAG 2.1 e à Lei nº 13.146/2015 (`RNF-027` / `RNF-043`)   |
| RNF09  | Mensagens de erro de validação de campos amigáveis e explícitas, evitando códigos técnicos (`RNF-028`)                |
| RNF10  | Arquitetura de software totalmente desacoplada e baseada na stack React, Node.js e PostgreSQL (`RNF-030` / `RNF-046`) |

## 11. Ponto de Extensão

### PE1. Inicialização de Fórum de Discussão por Questão

Imediatamente após a gravação da questão com sucesso no acervo (`POS01`), o sistema estende a operação para gerar automaticamente o Fórum de Discussão correspondente (`F2.2`), aplicando nativamente os filtros de acesso por curso descritos nas regras de negócio (`RN3`) e exibindo os termos de conduta ética obrigatórios (`RNF-041`).

## 12. Frequência de Utilização

Média. Alimentação contínua pelos docentes ao longo dos semestres, intensificada em janelas que antecedem avaliações simuladas programadas pelas coordenações.

## 13. Interface Visual

### IV1. Tela de Cadastro e Edição de Questões Educacionais

#### Layout da Tela

Interface de uso exclusivo da equipe docente para inserção, formatação estruturada de itens de avaliação e revisão de justificativas técnicas, utilizando o token de sessão ativa e exibindo o curso de atuação de forma estática.

---

### 13.1 Campos da Interface

| Campo                      | Tipo/Formato      | Obrigatório | Descrição                                                     | Regra de Negócio                                                                |
| -------------------------- | ----------------- | ----------- | ------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| Token de Autorização       | JWT / Oculto      | Sim         | Token de identificação com perfil do usuário                  | Deve possuir privilégio de Professor ou Admin (`RNF-017`)                       |
| Curso Atual                | Texto (Bloqueado) | Sim         | Exibe o nome do curso do professor logado                     | Campo preenchido via backend de forma imutável (`RN2`)                          |
| Subcategoria de Área/Tema  | Lista de Seleção  | Sim         | Subcategoria ou eixo temático da avaliação                    | Exibe apenas as subcategorias ativas vinculadas ao curso do docente (`RNF-042`) |
| Componente de Avaliação    | Radio Button      | Sim         | Classificação entre Formação Geral ou Conhecimento Específico | Padrão INEP obrigatório (`RNF-042`)                                             |
| Campo de Texto Base        | Rich Text / HTML  | Sim         | Contextualização inicial, trechos de textos ou cenários       | Permite formatação textual limpa                                                |
| Upload de Imagem           | Arquivo (PNG/JPG) | Não         | Imagem, gráfico ou mapa de suporte técnico                    | Tamanho máximo controlado pelo servidor                                         |
| Enunciado da Questão       | Texto Longo       | Sim         | Pergunta ou comando direcionado ao aluno                      | Campo obrigatório                                                               |
| Alternativa A              | Texto             | Sim         | Opção de resposta de múltipla escolha A                       | Campo estruturado obrigatório                                                   |
| Alternativa B              | Texto             | Sim         | Opção de resposta de múltipla escolha B                       | Campo estruturado obrigatório                                                   |
| Alternativa C              | Texto             | Sim         | Opção de resposta de múltipla escolha C                       | Campo estruturado obrigatório                                                   |
| Alternativa D              | Texto             | Sim         | Opção de resposta de múltipla escolha D                       | Campo estruturado obrigatório                                                   |
| Alternativa E              | Texto             | Sim         | Opção de resposta de múltipla escolha E                       | Campo estruturado obrigatório                                                   |
| Gabarito Correto           | Radio Group       | Sim         | Selecionador de qual alternativa (A a E) é a correta          | Apenas uma alternativa pode ser marcada                                         |
| Justificativa / Comentário | Texto Longo       | Sim         | Explicação técnica e pedagógica detalhada do gabarito         | Obrigatório para questões ativas (`RNF-040`)                                    |
| Botão "Salvar Questão"     | Botão             | Sim         | Valida os dados e submete para ativação no acervo             | Envia em JSON estruturado via HTTPS                                             |
| Botão "Salvar Rascunho"    | Botão             | Não         | Grava o progresso atual sem exigir validações INEP completas  | Salva com status "Incompleto"                                                   |
| Botão "Excluir Rascunho"   | Botão             | Não         | Remove permanentemente o rascunho de questão                  | Dispara Modal de confirmação (`RNF-026`)                                        |

---

### 13.2 Navegabilidade

| Ação                                          | Resultado                                                                                                                                                         |
| --------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Acessar tela de cadastro                      | O sistema lê o token JWT, identifica o curso do usuário, preenche o rótulo "Curso Atual" de forma desabilitada e popula o combo de áreas com os subtemas do curso |
| Acionar "Salvar Questão" com campos em branco | O sistema bloqueia a requisição e sinaliza com destaque e texto o campo faltante (`RNF-028`)                                                                      |
| Acionar "Salvar Rascunho"                     | O sistema armazena os dados preenchidos sem cobrar regras de obrigatoriedade e exibe mensagem de rascunho salvo                                                   |
| Clicar em "Excluir Rascunho"                  | O sistema abre uma caixa de diálogo em Modal cobrando a confirmação da ação                                                                                       |
| Confirmar exclusão no Modal                   | O sistema deleta o registro, atualiza os logs de auditoria e limpa os campos da tela                                                                              |

---

### 13.3 Mensagens Previstas

| Código | Mensagem                                                                         |
| ------ | -------------------------------------------------------------------------------- |
| MSG001 | Questão cadastrada com sucesso.                                                  |
| MSG002 | Rascunho de questão salvo com sucesso.                                           |
| MSG003 | Atenção! O campo [Nome do Campo] é obrigatório.                                  |
| MSG004 | Tem certeza que deseja excluir este rascunho? Esta ação não poderá ser desfeita. |

---

### 13.4 Componentes Visuais

| Área               | Componente                                                                                                      |
| ------------------ | --------------------------------------------------------------------------------------------------------------- |
| Topo da Página     | Breadcrumbs de navegação, Rótulo estático do Curso do usuário e Seletor Dropdown para Subcategoria de Área/Tema |
| Seção Central      | Editores de texto rico (Rich Text Form) para Texto Base, Enunciado e Comentários                                |
| Bloco de Mídia     | Componente drag-and-drop para upload e visualização prévia da imagem de suporte                                 |
| Bloco de Asserções | Grupo alinhado de inputs de texto acoplados a seletores do tipo Radio Button                                    |
| Rodapé de Comandos | Barra flutuante de ações com botões (Salvar Questão, Salvar Rascunho, Excluir Rascunho)                         |

---

## 14. Observações

Não se aplica.

## 15. Referências

- Visão da Demanda (VD) - Seção 5 (Persona 5.3) e Seção 6 (Funcionalidades F4.1, F4.5, F4.6 e F4.8).
- Regras de Negócio da Solução (RN) - Identificadores RN1 (Estrutura INEP), RN2 (Limitação por área/curso), RN3 (Vínculo de Fórum) e RN5 (Trilha de auditoria).
- Especificação de Requisitos Não Funcionais (RNF) - Itens RNF-001, RNF-002, RNF-004, RNF-007, RNF-008, RNF-011, RNF-013, RNF-015, RNF-016, RNF-017, RNF-019, RNF-021, RNF-023, RNF-026, RNF-027, RNF-028, RNF-029, RNF-030, RNF-031, RNF-035, RNF-040, RNF-041, RNF-042, RNF-043 e RNF-046.
- Diagrama de Caso de Uso (`../Imagens/diagrama_casos_de_uso_v3.png`).
