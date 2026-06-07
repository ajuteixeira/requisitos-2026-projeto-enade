# Especificação de Requisitos Funcionais

## Caso de Uso (CDU) - (Sistema ENADE Comentado - SEC)

## Histórico de Versões

| Data       | Versão | Descrição                                                                      | Autor   |
| ---------- | ------ | ------------------------------------------------------------------------------ | ------- |
| 18/05/2026 | 1.0    | Criação inicial do caso de uso com base no requisito F1.1 da Visão da Demanda. | Juliana |
| 27/05/2026 | 1.1    | Inclusão da especificação de Interface Visual e outros ajustes.                | Juliana |
| 07/06/2026 | 1.2    | Ajustes gerais.                                                                | Juliana |

## 1. Nome do Caso de Uso

Realizar simulados cronometrados

<img src="../Imagens/Diagrama-CDU-1-RealizarSimulado_v2.jpg" width="60%">

## 2. Objetivo

Permitir ao aluno concluinte responder a blocos de questões aderentes ao padrão oficial do INEP dentro de um limite de tempo controlado, sob uma interface acessível e de alta performance, garantindo o salvamento automático do progresso e o cálculo exato de suas métricas estatísticas de desempenho.

## 3. Tipo de Caso de Uso

Concreto.

## 4. Atores

### 4.1 Primário

| Ator             | Descrição                                                                |
| ---------------- | ------------------------------------------------------------------------ |
| Aluno Concluinte | Realiza o simulado, seleciona alternativas e gerencia seu tempo de prova |

### 4.2 Secundário

| Ator                | Descrição                                                                                  |
| ------------------- | ------------------------------------------------------------------------------------------ |
| SECBackEndAPI       | Responsável por fornecer as questões, validar o tempo e processar os cálculos estatísticos |
| PostgreSQL (SEC_BD) | Responsável pela persistência e integridade dos dados e logs de auditoria                  |

## 5. Precondições

| Código | Descrição                                                                                                         |
| ------ | ----------------------------------------------------------------------------------------------------------------- |
| PRE01  | O Aluno Concluinte deve estar previamente autenticado no sistema (F4.1 / RNF-017)                                 |
| PRE02  | O Aluno Concluinte deve ter aceitado explicitamente o Termo de Consentimento da LGPD no primeiro acesso (RNF-019) |
| PRE03  | Devem existir questões válidas indexadas e cadastradas no banco de dados para o curso do aluno (RN1 / F4.8)       |

## 6. Fluxo Principal

### P1. Solicitar início do simulado

O Aluno Concluinte acessa a plataforma web por meio de um navegador compatível (`RNF-008`), visualiza o ambiente com estética limpa (`RNF-029`) e aciona o comando para iniciar um novo simulado cronometrado.

### P2. Inicializar caderno de questões e cronômetro

A solução processa a requisição via tráfego seguro HTTPS (`RNF-013`) em formato JSON (`RNF-035`), monta o caderno de questões personalizado para o curso, inicializa a contagem regressiva e renderiza a interface em menos de 3 segundos (`RNF-002`).

### P3. Apresentar questão componentizada

A solução renderiza a questão em tela utilizando componentes visuais reutilizáveis (`RNF-031`) e respeitando as diretrizes de acessibilidade WCAG 2.1 (`RNF-027`). O carregamento completo do texto base, imagens de suporte, enunciado e asserções leva menos de 1,5 segundos (`RNF-001`).

### P4. Registrar resposta de forma incremental

O Aluno Concluinte analisa a questão, seleciona uma das alternativas e clica no comando de navegação. A solução armazena a resposta em repouso de forma incremental no banco e localmente (`RNF-011`), assegurando que o consumo de memória RAM no cliente não ultrapasse 150MB (`RNF-005`).

> _Nota: O passo P4 se repete de forma cíclica até que todas as questões do simulado tenham sido navegadas._

### P5. Solicitar encerramento do simulado

O Aluno Concluinte conclui as respostas e aciona o comando de finalização. A solução interrompe temporariamente o cronômetro e exibe uma caixa de diálogo (Modal) solicitando a confirmação da ação crítica (`RNF-026`).

### P6. Computar resultados com precisão estatística

O Aluno Concluinte confirma a finalização. A solução encerra o cronômetro em definitivo, calcula a proporção exata de acertos por área com base nas fórmulas matemáticas do ENADE (`RNF-039`), persiste o histórico definitivo no banco (`PostgreSQL`) e apresenta a tela de visão geral do simulado.

## 7. Fluxos Alternativos

### A1. Aplicação de filtros customizados de acervo

#### A1.1. No passo P1 do fluxo principal, o Aluno Concluinte opta por parametrizar o simulado utilizando filtros de seleção (`F1.2`).

#### A1.2. O Aluno Concluinte define os critérios desejados (Formação Geral ou Conhecimento Específico) seguindo as taxonomias oficiais do INEP (`RNF-042`).

#### A1.3. A solução valida os filtros informados, monta o caderno restrito e retorna ao fluxo principal no passo P2.

## 8. Fluxos de Exceção

### E1. Queda abrupta de conexão de internet

#### E1.1. Durante a navegação ou submissão de respostas (passos P3 ou P4), a solução detecta instabilidade técnica ou perda de comunicação activa com o servidor AWS.

#### E1.2. A solução aciona os mecanismos de tolerância a falhas, congelando imediatamente o cronômetro regressivo em tela para não punir o aluno com encerramento forçado indevido (`RNF-011` / `RNF-037`).

#### E1.3. A solução salva localmente a última resposta assinalada via _LocalStorage_ (`RNF-011`) e exibe uma notificação de erro clara na interface responsiva (`RNF-028`).

#### E1.4. Assim que a conexão HTTPS (TLS 1.3) é restabelecida, o estado do simulado é validado junto ao servidor e o Aluno Concluinte retoma o simulado sem perda de progresso.

### E2. Esgotamento do tempo limite estabelecido (Timeout)

#### E2.1. No decorrer do passo P4, o cronômetro regressivo atinge o tempo zerado antes do encerramento manual.

#### E2.2. A solução congela os campos de seleção da tela para impedir modificações de respostas fora do prazo.

#### E2.3. A solução recolhe automaticamente as alternativas salvas no banco até aquele momento e avança compulsoriamente para o processamento de notas e estatísticas descrito no passo P6.

## 9. Pós-condições

| Código | Descrição                                                                                                            |
| ------ | -------------------------------------------------------------------------------------------------------------------- |
| POS01  | A tentativa de simulado e suas respostas são registradas de forma permanente no PostgreSQL                           |
| POS02  | Os painéis estatísticos individuais do aluno e os relatórios gerenciais das turmas são atualizados sincronizadamente |
| POS03  | O Aluno Concluinte ganha privilégio de acesso para interagir nos Fóruns por Questão correspondentes ao simulado      |

## 10. Requisitos Não Funcionais

| Código | Requisito                                                                                                                            |
| ------ | ------------------------------------------------------------------------------------------------------------------------------------ |
| RNF01  | O início e a renderização da interface do simulado não devem exceder 3 segundos após o comando (RNF-002)                             |
| RNF02  | O carregamento do texto base, imagens e alternativas de uma questão deve levar menos de 1,5 segundos (RNF-001)                       |
| RNF03  | A infraestrutura deve suportar um mínimo de 500 usuários simultâneos realizando testes sem degradação do tempo de resposta (RNF-003) |
| RNF04  | A aplicação web em React não deve consumir mais de 150MB de memória RAM no navegador da máquina do cliente (RNF-005)                 |
| RNF05  | A interface deve ser 100% responsiva, adaptando-se a resoluções a partir de 360px de largura (RNF-007)                               |
| RNF06  | Em caso de perda de conexão, o sistema deve salvar o progresso localmente e congelar o tempo sem punir o aluno (RNF-011 / RNF-037)   |
| RNF07  | Todo o tráfego de dados do simulado deve obrigatoriamente utilizar criptografia por meio do protocolo HTTPS TLS 1.3 (RNF-013)        |
| RNF08  | O sistema deve exibir obrigatoriamente uma caixa de diálogo de confirmação antes de o aluno finalizar o simulado (RNF-026)           |
| RNF09  | A interface deve seguir as diretrizes básicas da WCAG 2.1, oferecendo suporte a alto contraste e leitores de tela (RNF-027)          |
| RNF10  | O design de interface deve adotar uma estética limpa, minimizando distrações visuais nas telas de simulados (RNF-029)                |

## 11. Ponto de Extensão

## PE1. Consultar Gabarito Detalhado e Comentado

Permite direcionar o Aluno Concluinte para a funcionalidade de exibição de gabarito comentado (F2.1) imediatamente após a consolidação dos resultados do simulado.

## 12. Frequência de Utilização

Alta. Volume concentrado massivamente em períodos de avaliações institucionais e nos meses que antecedem o cronograma oficial de aplicação do ENADE.

# 13. Interface Visual

## IV1. Tela de Execução de Simulado Cronometrado

### Layout da Tela

Interface de uso exclusivo do Aluno Concluinte focada em alta performance e legibilidade, exibindo blocos de questões sequenciais, mapa de navegação de respostas e cronômetro regressivo persistente.

---

## 13.1 Campos da Interface

| Campo                      | Tipo/Formato         | Obrigatório | Descrição                                                           | Regra de Negócio / RNF                                |
| -------------------------- | -------------------- | ----------- | ------------------------------------------------------------------- | ----------------------------------------------------- |
| Cronômetro Regressivo      | Contador de Tempo    | Sim         | Tempo restante dinâmico no formato HH:MM:SS                         | Congela em quedas de rede (RN4 / RNF-037)             |
| Painel de Questões         | Grid Numérico        | Sim         | Atalhos visuais para saltar direto para qualquer questão do caderno | Muda de cor para indicar item respondido ou pendente  |
| Identificador do Item      | Rótulo / Texto       | Sim         | Código e numeração sequencial da questão atual                      | Exemplo: "Questão 04 - Componente Específico"         |
| Bloco de Texto Base        | Rich Text (Leitura)  | Sim         | Texto principal, charges, artigos ou cenários formulados            | Renderização rápida em tela (RNF-001)                 |
| Imagem Decorativa/Suporte  | Imagem / Gráfico     | Não         | Ilustrações técnicas acopladas à contextualização                   | Suporte a zoom e descrição de tela (RNF-027)          |
| Enunciado da Avaliação     | Texto (Leitura)      | Sim         | Pergunta ou comando central da questão                              | Alinhado à estrutura oficial (RN1)                    |
| Alternativa A              | Radio Button / Texto | Sim         | Texto descritivo da asserção A                                      | Seleção exclusiva e salvamento automático             |
| Alternativa B              | Radio Button / Texto | Sim         | Texto descritivo da asserção B                                      | Seleção exclusiva e salvamento automático             |
| Alternativa C              | Radio Button / Texto | Sim         | Texto descritivo da asserção C                                      | Seleção exclusiva e salvamento automático             |
| Alternativa D              | Radio Button / Texto | Sim         | Texto descritivo da asserção D                                      | Seleção exclusiva e salvamento automático             |
| Alternativa E              | Radio Button / Texto | Sim         | Texto descritivo da asserção E                                      | Seleção exclusiva e salvamento automático             |
| Botão "Próximo"            | Botão                | Não         | Retorna ao item imediatamente anterior do caderno                   | Desabilitado se o usuário estiver na primeira questão |
| Botão "Anterior"           | Botão                | Não         | Grava a resposta e avança para a próxima questão                    | Salva incrementalmente (RNF-011)                      |
| Botão "Finalizar Simulado" | Botão                | Sim         | Conclui a prova e envia o caderno de respostas para a API           | Abre caixa de diálogo obrigatória (RNF-026)           |

---

## 13.2 Navegabilidade

| Ação                                            | Resultado                                                                                                                       |
| ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| Marcar alternativa e clicar em "Próximo"        | O sistema registra a opção de forma incremental (banco/local), muda a cor do item no painel e exibe a próxima questão (RNF-011) |
| Clicar em qualquer número do Painel de Questões | A interface realiza uma transição suave direta para a questão selecionada, mantendo o tempo rodando                             |
| Clicar em "Finalizar Simulado"                  | O cronômetro é congelado em segundo plano e uma caixa de diálogo em Modal surge cobrando a confirmação                          |
| Confirmar encerramento no Modal                 | O sistema interrompe o ciclo, processa as notas e métricas via backend e redireciona para a tela de resumo (P6)                 |
| O tempo zerar no Cronômetro                     | O sistema bloqueia imediatamente os campos de rádio e envia as respostas salvas compulsoriamente (E2)                           |

---

## 13.3 Mensagens Previstas

| Código | Mensagem                                                                                                         |
| ------ | ---------------------------------------------------------------------------------------------------------------- |
| MSG001 | Atenção! Você possui questões pendentes de resposta. Deseja realmente finalizar o simulado?                      |
| MSG002 | Conexão de rede interrompida. Seu progresso foi guardado localmente e o cronômetro está pausado preventivamente. |
| MSG003 | Sinal restabelecido! Sincronizando respostas com o servidor...                                                   |
| MSG004 | Tempo limite esgotado. Suas respostas foram computadas e consolidadas automaticamente.                           |
| MSG005 | Simulado concluído com sucesso. Gerando suas métricas de desempenho estatístico...                               |

---

## 13.4 Componentes Visuais

| Área                        | Componente                                                                                                                 |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| Barra de Topo Fixa (Header) | Barra escura contendo o Cronômetro em fonte digital de alto contraste e o botão vermelho "Finalizar Simulado"              |
| Menu Lateral Esquerdo       | Gaveta retrátil (Sidebar) contendo o grid numérico das questões com badges coloridos (Cinza: Pendente, Verde: Respondido)  |
| Corpo Central da Página     | Card branco contendo o Texto Base estruturado em HTML limpo, imagem centralizada e Enunciado em destaque                   |
| Bloco de Asserções          | Lista vertical de blocos com efeito hover que cobrem toda a largura útil, facilitando o clique em telas sensíveis ao toque |
| Barra de Rodapé             | Botões minimalistas posicionados nas extremidades esquerda ("Anterior") e direita ("Próxima")                              |

---

## 14. Observações

Não se aplica.

## 15. Referências

- Visão da Demanda (VD) - Seção 6 (Funcionalidades F1.1, F1.2, F2.1, F3.1 e F4.1).
- Regras de Negócio da Solução (RN) - Identificadores RN1 (Estrutura INEP), RN3 (Vínculo de Fórum) e RN4 (Congelamento de tempo).
- Especificação de Requisitos Não Funcionais (RNF) - Itens RNF-001, RNF-002, RNF-003, RNF-005, RNF-007, RNF-011, RNF-013, RNF-017, RNF-019, RNF-026, RNF-027, RNF-029, RNF-037 e RNF-039.
- Diagrama de Caso de Uso (`../Imagens/diagrama_casos_de_uso_v3.png`).
