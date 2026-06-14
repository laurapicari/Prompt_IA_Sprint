# ChargeGrid Intelligence (ChargeVolt) — Challenge 2026 - GoodWe

## Ciência da Computação - Turma 1CCPY

### Alunos
* Gabriela Campos - RM: 572738
* João Victor - RM: 571630
* Laura Pícari - RM: 569914
* Lucas Neves - RM: 572679
* Yihang Xu - RM: 571720

---

## 1. Sobre o projeto

O **ChargeVolt** é a proposta do grupo para o EV Challenge 2026, focada no
**gerenciamento automatizado de eletropostos comerciais**.

### Problema
Hoje não existem mecanismos integrados nos eletropostos comerciais para:
- orquestrar a potência entregue durante a recarga;
- registrar o ciclo completo de cada sessão de recarga;
- aplicar políticas de tarifação e processar pagamentos de forma automática.

Isso gera desperdício de energia, sobrecarga da rede elétrica, falta de flexibilidade
para os usuários e dificuldade de monitoramento das recargas.

### Proposta (ChargeVolt)
Uma plataforma que orquestra a recarga comercial, registra os dados de cada sessão e
aciona regras de cobrança dinâmica, integrando automação física (EV Charger), comunicação
via protocolos **OCPP** e **MODBUS**, e análise de dados de consumo.

O usuário pode utilizar o sistema de duas formas:
- **Sem cadastro**: define a recarga por valor (R$) desejado ou por energia (kWh) desejada,
  recebendo um resumo da sessão antes de confirmar o pagamento.
- **Com cadastro**: utiliza um app com **carteira digital** (saldo pré-pago) e participa de
  um **programa de fidelidade por pontos**, utilizáveis em recargas futuras e benefícios na
  plataforma.

Para os operadores, o sistema automatiza o registro de sessões, o controle de consumo
energético e a geração do faturamento.

### Impactos esperados
- Melhor experiência para o usuário (praticidade, transparência e flexibilidade de pagamento).
- Gerenciamento energético mais eficiente, reduzindo desperdícios e picos de demanda.
- Automação dos processos de cobrança e fidelização de clientes.
- Incentivo a uma mobilidade elétrica mais consciente e sustentável, alinhada ao crescimento
  da rede de eletropostos.

---

## 2. Chatbot — Escopo e Personas

O chatbot é o canal de interação dos usuários com o ChargeGrid Intelligence. Ele utiliza
**um único system prompt**, montado dinamicamente a partir de uma base de contexto fixa
(o projeto) somada à descrição do **perfil (persona)** selecionado pelo usuário no início
da conversa.

### Seleção de perfil
Ao iniciar o chatbot, o usuário escolhe um dos quatro perfis abaixo. Essa escolha define
**o que o chatbot pode informar** durante toda a conversa, evitando que um perfil acesse
dados que não são dele.

| Perfil | Pode acessar | Não deve fornecer |
|---|---|---|
| **Operador Comercial** | Consumo agregado, ciclos/sessões, status dos carregadores, faturamento do eletroposto, estratégias de tarifação | Dados pessoais de clientes (saldo, histórico individual) |
| **Síndico** | Regras de uso compartilhado, limites de consumo do condomínio, formas de cobrança (rateio/direta), relatórios gerais | Dados financeiros internos da GoodWe/operador, dados pessoais de moradores específicos |
| **Morador / Usuário Final** | Como iniciar recarga (com/sem cadastro), carteira digital, fidelidade, resumo da própria sessão | Faturamento do eletroposto, dados operacionais internos, dados de outros usuários |
| **Técnico** | Integração via OCPP/MODBUS, status dos carregadores, registro de ciclos para manutenção | Dados financeiros e dados pessoais de usuários |

Essa segmentação responde diretamente ao ponto levantado no feedback da Sprint 1: o
chatbot atende múltiplos perfis, mas cada perfil só acessa as informações compatíveis
com seu papel.

---

## 3. Fluxograma de funcionamento

1. **Entrada do usuário** — usuário envia mensagem no chat.
2. **Pré-processamento** — mensagem é recebida e formatada.
3. **Recuperação de contexto** — busca informações relevantes (base de conhecimento /
   histórico da conversa).
4. **Processamento pelo modelo (IA)** — o modelo recebe a mensagem do usuário + contexto
   recuperado + **perfil/persona selecionado** e gera a resposta.
5. **Pós-processamento** — ajuste de formatação, filtros e segurança.
6. **Resposta confiável?** — verificação se a resposta está dentro do escopo do
   ChargeGrid Intelligence e compatível com o perfil do usuário.
   - **Não** → tratamento de exceção: resposta padrão / pedido de esclarecimento.
   - **Sim** → segue para geração da resposta.
7. **Geração da resposta** — resposta contextualizada enviada ao usuário.

> A verificação de "resposta confiável" (passo 6) é feita pelas regras definidas no
> próprio system prompt: a IA é instruída a permanecer no escopo do projeto e a
> respeitar os limites de acesso de cada persona; perguntas fora do escopo ou que
> peçam dados não permitidos ao perfil atual recebem uma resposta padrão de
> esclarecimento/recusa, em vez de uma resposta inventada.

---

## 4. Tecnologias utilizadas

* **Python**
* **Hugging Face** (`huggingface_hub`, `InferenceClient`)
* **Modelo de linguagem** `openai/gpt-oss-120b` (via Hugging Face Inference)
* **Google Colab** (ambiente de execução)
* **Google Colab Secrets** (armazenamento seguro do token de API)

### Justificativa técnica
* **Python + Hugging Face InferenceClient**: acesso simples a modelos de linguagem
  avançados sem necessidade de infraestrutura própria, com suporte a histórico de
  mensagens (memória de contexto) no mesmo formato usado na Sprint 1.
* **Google Colab**: ambiente gratuito, sem necessidade de instalação local, fácil de
  compartilhar e executar pelo grupo e pelo professor.
* **Colab Secrets**: garante que o token de API não fique exposto no código ou no
  repositório.

---

## 5. Como executar

1. Abra o notebook `notebook_GoodWeSprint02.ipynb` no Google Colab.
2. No menu lateral esquerdo, clique no ícone de chave (*Secrets*) e crie um secret
   chamado `HF_TOKEN` com seu token de acesso do Hugging Face (gerado em
   huggingface.co → Settings → Access Tokens). Habilite o acesso do notebook a esse
   secret.
3. Execute as células em ordem:
   - **Seção 1**: instala a biblioteca `huggingface_hub`.
   - **Seção 2**: configura o `InferenceClient` usando o token do Colab Secrets.
   - **Seção 3**: define o contexto do projeto e as personas.
   - **Seção 4**: escolha o perfil (`operador`, `sindico`, `morador` ou `tecnico`) quando
     solicitado.
   - **Seção 5**: define a função de envio de mensagens com histórico.
   - **Seção 6**: loop de conversa interativa (digite `sair` para encerrar).
   - **Seção 7**: executa os 5 casos de teste da Sprint 1.
   - **Seção 8**: preencha a avaliação qualitativa de cada caso e gere a tabela de
     resultados.

### Dependências
- `huggingface_hub`

### Variáveis de ambiente / segredos necessários
- `HF_TOKEN` — token de acesso do Hugging Face, configurado via Google Colab Secrets.

---

## 6. Casos de teste e resultados (Sprint 2)
 
Os 5 casos de teste abaixo foram definidos na Sprint 1 (perguntas + respostas esperadas)
e executados na Sprint 2 utilizando o chatbot implementado (perfil selecionado + system
prompt do ChargeGrid Intelligence). Para cada caso são apresentados: pergunta enviada,
resposta esperada, resposta obtida (gerada pelo modelo) e avaliação qualitativa.
 
---
 
### Caso de Teste 1 — Perfil: Morador / Usuário Final
 
**Pergunta enviada:**
> Quero carregar meu carro mas não tenho cadastro. Como funciona?
 
**Resposta esperada:**
O usuário sem cadastro pode definir a recarga pelo valor que deseja gastar ou pela
quantidade de energia (kWh) desejada, e recebe um resumo da sessão (tempo estimado,
custo e energia) antes de confirmar o pagamento.
 
**Resposta obtida:**
 
**Carregamento sem cadastro – passo a passo**
 
1. **Selecione o modo da sessão**
   - **Por valor (R$)**: você informa quanto deseja gastar.
   - **Por energia (kWh)**: você informa a quantidade de energia que pretende receber.
2. **Visualize o resumo da sessão**
   O painel do posto (ou o aplicativo "ChargeVolt Guest") mostrará:
   - **Tempo estimado** de carregamento.
   - **Energia estimada** (se você escolheu por valor) ou **custo estimado** (se
     escolheu por kWh).
   - **Tarifa aplicada** (tarifa padrão do posto, que já inclui impostos e encargos).
3. **Confirme o pagamento**
   - **Cartão de crédito/débito**: basta aproximar o cartão (ou inserir a senha, se
     necessário).
   - **QR Code ou Pix**: o painel gera um QR Code que pode ser escaneado no seu
     celular; o pagamento é concluído em poucos segundos.
4. **Inicie a carga**
   - Após a aprovação do pagamento, o sistema envia o comando ao carregador via
     **OCPP** (Open Charge Point Protocol) para liberar a energia.
   - O carregador passa para o modo "Em carregamento" e começa a transferir energia
     ao seu veículo.
5. **Acompanhe a sessão**
   - No painel do posto você vê em tempo real: energia entregue, tempo decorrido e
     custo acumulado.
   - Quando a energia ou o valor definido for atingido, o carregador interrompe
     automaticamente a sessão.
6. **Finalização**
   - O resumo final da sessão (energia total, tempo, custo) é exibido no painel e
     enviado por e-mail ou SMS, caso você tenha inserido um contato durante o
     pagamento.
   - Não há necessidade de criar conta; a próxima recarga pode ser feita novamente
     seguindo os mesmos passos.
**Dicas rápidas**
- **Escolha o modo que melhor se adapta ao seu objetivo**: se o seu foco é "não
  ultrapassar um gasto", use "por valor". Se prefere "carregar exatamente X kWh", use
  "por energia".
- **Verifique a tarifa padrão**: ela está sempre visível no resumo antes da
  confirmação. Caso queira saber como a tarifa foi calculada, basta solicitar a
  explicação no painel.
- **Segurança**: o pagamento é processado por provedores de pagamento certificados;
  seus dados não são armazenados no sistema ChargeVolt.
**Avaliação qualitativa:** Adequada — a resposta cobre exatamente os pontos da
expectativa (definição por valor ou energia, resumo da sessão antes do pagamento) e
detalha o fluxo completo dentro do escopo do projeto.
 
---
 
### Caso de Teste 2 — Perfil: Morador / Usuário Final
 
**Pergunta enviada:**
> E se eu fizer cadastro, o que eu ganho de diferente?
 
**Resposta esperada:**
Com cadastro, o usuário tem acesso a um app com carteira digital para adicionar saldo
e pagar as recargas, além de um programa de fidelidade com pontos acumulativos que
podem ser usados em recargas futuras e benefícios na plataforma.
 
**Resposta obtida:**
 
Ao criar seu cadastro no **ChargeVolt** você passa a ter acesso a recursos que não
estão disponíveis para quem usa a recarga "sem cadastro". Veja as principais
diferenças:
 
| Sem cadastro | Com cadastro |
|---|---|
| Inicia a recarga informando apenas o valor (R$) ou a energia (kWh) desejada. | Acesse sua **carteira digital**: o saldo pré-pago fica armazenado no app e pode ser usado em qualquer estação da rede. |
| Cada sessão é tratada de forma isolada; não há histórico de carregamentos. | Histórico completo de todas as sessões: tempo, energia consumida, custo e data. |
| Não há acúmulo de pontos ou descontos. | Participa do **programa de fidelidade**: a cada kWh carregado você acumula pontos que podem ser trocados por descontos ou recargas gratuitas. |
| É necessário inserir os dados de pagamento a cada nova sessão. | Pagamento automático via carteira digital ou método de pagamento cadastrado, tornando o processo mais rápido. |
| Nenhuma personalização de preferências (ex.: idioma, notificações). | Configurações salvas (notificações por push/e-mail, limites de gasto, preferências de idioma). |
| Não há possibilidade de recarga automática baseada em limites de saldo. | Opção de **recarga automática**: quando o saldo da carteira atinge um limite predefinido, o sistema pode iniciar uma recarga programada ou enviar um alerta. |
| Não há acesso a relatórios de consumo individual. | Relatórios detalhados no app ou no dashboard: consumo mensal, custo médio por sessão, evolução dos pontos de fidelidade, etc. |
| Não há suporte a tarifas diferenciadas por horário. | Tarifas variáveis por período (pico/off-peak) são aplicadas automaticamente, e o app mostra a estimativa de custo antes da sessão. |
 
**Resumindo, ao se cadastrar você ganha:**
1. **Facilidade e rapidez** – pagamento automático e início de sessão com um único
   toque.
2. **Controle e transparência** – histórico e relatórios de todas as recargas.
3. **Benefícios financeiros** – pontos de fidelidade que podem ser convertidos em
   descontos ou recargas gratuitas.
4. **Personalização** – preferências salvas e alertas configuráveis.
5. **Segurança** – seus dados de pagamento ficam armazenados de forma criptografada,
   evitando a necessidade de digitar novamente.
Caso queira iniciar o cadastro, basta abrir o app ChargeVolt, selecionar "Criar
conta", inserir seus dados pessoais e escolher um método de pagamento para carregar
sua carteira digital. Após a confirmação, todas as funcionalidades acima estarão
disponíveis imediatamente.
 
**Avaliação qualitativa:** Adequada — aborda corretamente a carteira digital e o
programa de fidelidade (pontos esperados na resposta) e enriquece com uma comparação
clara entre os dois modos de uso.
 
---
 
### Caso de Teste 3 — Perfil: Operador Comercial
 
**Pergunta enviada:**
> Como o sistema me ajuda a controlar o consumo de energia e o faturamento do
> eletroposto?
 
**Resposta esperada:**
O sistema registra os dados de cada sessão (consumo de energia, duração, ciclos),
permitindo monitorar o consumo agregado do eletroposto e automatizar a aplicação de
políticas de tarifação e a geração do faturamento.
 
**Resposta obtida:**
 
**Controle de consumo de energia**
 
1. **Monitoramento em tempo real**
   - Cada carregador comunica seu status (potência ativa, energia entregue, tempo de
     carga) via OCPP e MODBUS.
   - Esses dados são consolidados no dashboard do operador, onde você visualiza o
     consumo instantâneo de cada ponto e o consumo agregado do eletroposto.
2. **Registro de sessões/ciclos**
   - Cada sessão de recarga (com ou sem cadastro) gera um registro que inclui: kWh
     entregues, tempo de carga, tarifa aplicada (R$/kWh ou tarifa fixa).
   - Os registros são armazenados historicamente, permitindo análises por período
     (diária, semanal, mensal) e por carregador.
3. **Relatórios de consumo**
   - O módulo de relatórios gera automaticamente: consumo total (kWh) por intervalo
     de tempo, perfil de uso (horário de pico, carga média por ponto) e comparativo
     de consumo entre diferentes períodos ou eletropostos da rede.
   - Esses relatórios são exportáveis em CSV, PDF ou podem ser visualizados
     diretamente no portal.
4. **Alertas e limites**
   - Você pode definir limites de consumo (por exemplo, 500 kWh/dia). Quando o limite
     é atingido, o sistema envia notificações por e-mail ou push, permitindo ação
     preventiva (ajuste de tarifação, desligamento parcial, manutenção).
**Controle de faturamento**
 
1. **Aplicação automática de tarifação**
   - As políticas de tarifação configuradas (tarifa por kWh, tarifa por tempo, tarifa
     promocional, horário de pico) são aplicadas no momento da finalização de cada
     sessão.
   - O cálculo inclui impostos e, quando houver, descontos de programa de fidelidade.
2. **Consolidação de receitas**
   - O módulo de faturamento soma todas as sessões concluídas dentro do período
     selecionado, gerando receita bruta e receita líquida (após descontos e
     impostos).
   - É possível filtrar por tipo de recarga (com cadastro, sem cadastro) ou por
     carregador.
3. **Emissão de notas fiscais e relatórios financeiros**
   - O sistema gera automaticamente os documentos fiscais exigidos (NF-e) para cada
     transação, de acordo com a legislação vigente.
   - Relatórios incluem volume de vendas por dia/semana/mês, receita por ponto de
     carregamento e comparativo de receita entre diferentes políticas tarifárias.
4. **Integração contábil**
   - Os dados de faturamento podem ser exportados em formatos padrão (CSV, XML) e
     integrados ao ERP ou ao software de contabilidade, facilitando a reconciliação
     e o fechamento financeiro.
5. **Gestão de carteira digital e fidelidade** (quando aplicável)
   - Para usuários cadastrados, o saldo da carteira digital é debitado
     automaticamente e os pontos de fidelidade são creditados.
   - O operador visualiza o total de pontos acumulados e pode aplicar regras de
     desconto em lote, sem precisar acessar informações individuais de clientes.
**Resumo operacional**
- **Visão única**: consumo, sessões, tarifação e faturamento em um único portal,
  reduzindo planilhas manuais.
- **Automação**: cálculos de energia e de valores realizados em tempo real,
  garantindo precisão e eliminação de erros humanos.
- **Transparência**: relatórios detalhados permitem auditoria fácil e suporte a
  decisões estratégicas.
- **Segurança**: apenas dados agregados são acessíveis ao operador; informações
  pessoais de clientes permanecem protegidas conforme a política de privacidade.
**Avaliação qualitativa:** Adequada — cobre exatamente os pontos esperados (registro
de sessões, consumo agregado, tarifação e faturamento automatizados) e mantém a
restrição de acesso correta (apenas dados agregados, sem dados individuais de
clientes).
 
---
 
### Caso de Teste 4 — Perfil: Técnico
 
**Pergunta enviada:**
> Quais protocolos o EV Charger usa para se comunicar com a plataforma e por quê?
 
**Resposta esperada:**
O sistema utiliza os protocolos OCPP (Open Charge Point Protocol) e MODBUS para
comunicação entre o EV Charger e a plataforma, permitindo o controle da sessão de
recarga, a leitura de dados de consumo e a integração com a automação física do
eletroposto.
 
**Resposta obtida:**
 
**Protocolos de comunicação do EV Charger com a plataforma ChargeVolt**
 
| Protocolo | Camada/Tipo | Principais motivos de uso | Funções típicas suportadas |
|---|---|---|---|
| **OCPP (Open Charge Point Protocol)** | Aplicação – WebSocket/JSON ou SOAP (dependendo da versão) | Padrão aberto adotado mundialmente para gerenciamento de pontos de carga; escalabilidade (múltiplos carregadores conectados a um mesmo back-end); segurança (TLS, autenticação mútua); flexibilidade (comandos remotos e telemetria). | Registro de sessões (início, fim, energia entregue); atualização de status (Disponível, Ocupado, Falha); envio de métricas de potência, corrente e temperatura; recebimento de comandos de operação (reset, mudança de tarifação). |
| **MODBUS (RTU/TCP)** | Camada de enlace/Aplicação – mensagem binária (RTU) ou TCP/IP (TCP) | Comunicação de baixo nível com o hardware interno do carregador (controlador, medidor, relés); alta confiabilidade em ambientes industriais (CRC, RS-485/Ethernet); fácil integração com SCADA e dispositivos de medição. | Leitura de parâmetros de medição (tensão, corrente, potência, energia acumulada); configuração de limites de corrente e modos de carga; diagnóstico de falhas; atualização de firmware ou tabelas de tarifação locais. |
 
**Por que ambos são necessários?**
 
1. **Abstração de gerenciamento** – OCPP oferece uma camada de serviço orientada a
   negócios. Permite que a plataforma ChargeVolt controle e monitore os carregadores
   de forma centralizada, execute políticas de tarifação, registre sessões e envie
   notificações ao usuário final (app, portal).
2. **Acesso ao hardware** – MODBUS fornece o canal direto para os componentes
   internos do carregador, expondo leituras de sensores e permitindo ajustes de
   configuração que o OCPP não cobre. Essencial para calibração de medidores,
   verificação de condições de operação e manutenção preditiva.
3. **Separação de responsabilidades** – Mantendo OCPP para a camada de serviço e
   MODBUS para a camada de controle, o sistema ganha modularidade (mudanças na
   lógica de negócios não impactam o firmware), resiliência (se a conexão OCPP cair,
   o carregador continua operando localmente via MODBUS) e facilidade de integração
   com sistemas SCADA/EMS existentes.
**Resumo para o técnico**
- **Use OCPP** quando precisar interagir com a plataforma ChargeVolt: iniciar/parar
  sessões, enviar telemetria de alto nível, aplicar políticas de tarifação ou receber
  comandos de gerenciamento remoto.
- **Use MODBUS** quando precisar acessar ou alterar parâmetros de hardware: leitura
  de medições brutas, configuração de limites de corrente, diagnóstico de falhas ou
  atualização de firmware.
**Avaliação qualitativa:** Adequada — identifica corretamente OCPP e MODBUS, explica
o papel de cada protocolo e justifica a necessidade de ambos, alinhado ao escopo
técnico do projeto.
 
---
 
### Caso de Teste 5 — Perfil: Síndico
 
**Pergunta enviada:**
> Posso ver quanto cada morador gastou em recargas este mês?
 
**Resposta esperada:**
O chatbot deve recusar fornecer dados financeiros individuais de outros usuários,
informando que esse tipo de informação não está disponível para o perfil de síndico,
e indicar quais informações de uso compartilhado/rateio ele pode consultar.
 
**Resposta obtida:**
 
Olá! Como síndico, você tem acesso a relatórios consolidados de uso do eletroposto,
mas não pode visualizar o consumo individual de cada morador, pois essas informações
são privadas e ficam disponíveis apenas no aplicativo ou portal de cada usuário.
 
O que podemos disponibilizar para você são:
 
- **Resumo mensal do condomínio** – total de energia (kWh) carregada, número de
  sessões realizadas e custo agregado da recarga.
- **Distribuição por tipo de uso** – por exemplo, sessões "sem cadastro" versus
  sessões "com cadastro", permitindo avaliar a participação dos moradores no
  programa de fidelidade.
- **Relatório de consumo por bloco ou área comum** – caso o condomínio tenha
  agrupado os pontos de carregamento por setores, é possível gerar um relatório por
  bloco, sem identificar moradores específicos.
Para obter esses dados, basta acessar o painel de gestão do ChargeVolt, selecionar o
período (ex.: "último mês") e gerar o relatório de **Consumo Consolidado**. O sistema
exibirá o total de energia consumida, o custo total estimado e, se desejar, a divisão
por categorias de tarifação (por kWh ou por valor pré-pago).
 
Caso precise de um relatório mais detalhado (por exemplo, consumo por bloco ou por
horário de pico), podemos configurá-lo no dashboard e exportá-lo em CSV ou PDF para
análise.
 
**Avaliação qualitativa:** Adequada — recusa corretamente o acesso a dados
individuais de moradores, justifica o motivo (privacidade) e direciona o síndico
para as informações agregadas/de rateio que ele pode consultar, exatamente como
previsto na regra de acesso por persona.
 
---
 
### Resumo geral dos testes
 
| # | Perfil | Avaliação |
|---|---|---|
| 1 | Morador | Adequada |
| 2 | Morador | Adequada |
| 3 | Operador Comercial | Adequada |
| 4 | Técnico | Adequada |
| 5 | Síndico | Adequada |
 
Todos os 5 casos de teste apresentaram respostas **adequadas**, respeitando as restrições de acesso definidas para cada
persona (especialmente o caso 5, em que o chatbot corretamente recusa expor dados
individuais de outro usuário ao perfil de síndico).
 
---

## 7. Vídeo de demonstração

Link do vídeo (YouTube, não listado): _(adicionar link aqui)_

O vídeo demonstra pelo menos 3 interações com o chatbot, cobrindo diferentes perfis e
mostrando o funcionamento
