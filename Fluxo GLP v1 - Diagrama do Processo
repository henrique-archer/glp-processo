# Sistema de Requisição de Recarga de GLP

Este repositório documenta o **processo de requisição, autorização, entrega e fechamento mensal**
das recargas de gás GLP, conforme fluxo operacional do Almoxarifado.

---

## 📌 Visão Geral do Processo

O processo é dividido em quatro módulos principais:

1. Cadastros
2. Requisição
3. Entrega
4. Fechamento Mensal

Os fluxos abaixo servem como **base para o desenvolvimento do sistema**, definição de status,
regras de negócio e auditoria.

---

## 🔁 Fluxograma do Processo (por Módulos)

flowchart LR
  %% === Estilos ===
  classDef status fill:#E0F7FA,stroke:#00838F,stroke-width:1px,color:#004D40;
  classDef action fill:#F1F8E9,stroke:#33691E,stroke-width:1px,color:#1B5E20;
  classDef decision fill:#FFF8E1,stroke:#F9A825,stroke-width:1px,color:#6D4C41,stroke-dasharray: 5 5;
  classDef doc fill:#F3E5F5,stroke:#6A1B9A,color:#4A148C;

  %% === Módulo 1: Cadastros ===
  subgraph M1["Módulo 1 · Cadastros"]
    A1(("Início"))
    A2[/"Cadastrar Controle"/]:::action
    A3["Definir saldo inicial"]:::action
    A4(("Fim"))
    A1 --> A2 --> A3 --> A4
  end

  %% === Módulo 2: Requisição ===
  subgraph M2["Módulo 2 · Requisição"]
    B1(("Início"))
    B2["Identificar necessidade de recarga de gás"]:::action
    B3["Preencher formulário"]:::action
    B4[["Status: solicitada"]]:::status
    B5["Analisar dados da solicitação"]:::action
    B6{"Dados corretos?"}:::decision
    B7["Solicitar correção ao requisitante\n(registrar mensagem)"]:::action
    B8[["Status: pendente_correcao"]]:::status
    B9["Requisitante corrige e reenvia"]:::action
    B10[["Status: em_analise"]]:::status
    B11["Verificar saldo do contrato\n(por tipo de carga e período)"]:::action
    B12{"Existe saldo contratual?"}:::decision
    B13["Cancelar solicitação"]:::action
    B14[["Status: cancelada"]]:::status
    B15["Autorizar fornecimento"]:::action
    B16["Gerar nº de autorização\n(RG-AAAA-#####)"]:::action
    B17["Registrar autorização no sistema"]:::action
    B18[["Status: autorizada"]]:::status
    B19["Encaminhar para entrega\n(notificar fornecedor)"]:::action
    B20[["Status: entrega_pendente"]]:::status

    B1 --> B2 --> B3 --> B4 --> B5 --> B6
    B6 -- "Não" --> B7 --> B8 --> B9 --> B10 --> B5
    B6 -- "Sim" --> B11 --> B12
    B12 -- "Não" --> B13 --> B14
    B12 -- "Sim" --> B15 --> B16 --> B17 --> B18 --> B19 --> B20
  end

  %% === Módulo 3: Entrega ===
  subgraph M3["Módulo 3 · Entrega"]
    C0[["Entrada: entrega_pendente"]]:::status
    C1["Executar entrega"]:::action
    C2["Fornecedor realiza entrega"]:::action
    C3["Recebedor confirma recebimento"]:::action
    C4["Coletar assinatura/recibo de entrega\n(upload de PDF/foto)"]:::doc
    C5["Registrar entrega no sistema\n(quantidade efetivamente entregue)"]:::action
    C6{"Entrega realizada?"}:::decision
    C7["Registrar motivo de insucesso"]:::action
    C8{"Reagendar entrega?"}:::decision
    C9[["Status: entrega_pendente"]]:::status
    C10["Cancelar requisição"]:::action
    C11[["Status: cancelada"]]:::status
    C12[["Status: entregue"]]:::status
    C13["Fornecedor envia documentação\ndo fechamento mensal"]:::doc
    C14[["Status: em_fechamento"]]:::status

    C0 --> C1 --> C2 --> C3 --> C4 --> C5 --> C6
    C6 -- "Não" --> C7 --> C8
    C8 -- "Sim" --> C9
    C8 -- "Não" --> C10 --> C11
    C6 -- "Sim" --> C12 --> C13 --> C14
  end

  %% === Módulo 4: Fechamento Mensal ===
  subgraph M4["Módulo 4 · Fechamento Mensal"]
    D0[["Entrada: em_fechamento"]]:::status
    D1["Executar conferência documental"]:::action
    D2["Comparar: autorizações registradas × faturas apresentadas"]:::action
    D3{"Conferência OK?"}:::decision
    D4["Solicitar correção ao fornecedor"]:::action
    D5["Registrar divergência"]:::action
    D6["Fornecedor reenvia documentação"]:::doc
    D7["Gerar encaminhamento para pagamento"]:::action
    D8[["Status: finalizada"]]:::status
    D9(("Fim"))

    D0 --> D1 --> D2 --> D3
    D3 -- "Não" --> D4 --> D5 --> D6 --> D1
    D3 -- "Sim" --> D7 --> D8 --> D9
  end

  %% === Ligações entre módulos ===
  A4 -.-> B1
  B20 --> C0
  C14 --> D0

  ---

## 🔄 Estados da Requisição

stateDiagram-v2
  [*] --> rascunho

  rascunho --> solicitada: Enviar formulário
  solicitada --> em_analise: Triagem/Análise
  em_analise --> pendente_correcao: Devolver com ajustes
  pendente_correcao --> em_analise: Requisitante corrige e reenvia
  em_analise --> autorizada: Aprovação/Autorizar fornecimento
  autorizada --> entrega_pendente: Encaminhar ao fornecedor

  entrega_pendente --> entregue: Entrega registrada (recibo)
  entrega_pendente --> cancelada: Insucesso/Cancelamento (motivo)

  entregue --> em_fechamento: Recebida documentação mensal
  em_fechamento --> finalizada: Conferência OK / Encaminhar p/ pagamento
  em_fechamento --> pendente_correcao: Divergência documental

  %% Transições excepcionais
  solicitada --> cancelada: Sem saldo contratual
  em_analise --> cancelada: Sem saldo / Erro crítico
  [*] --> cancelada: Abortada
