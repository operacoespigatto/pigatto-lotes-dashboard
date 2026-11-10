# Painel Comercial — Produtos a Vencer (Grupo Pigatto)

Painel **aberto (sem senha)** com os lotes que vencem nos próximos 90 dias e
o risco de cada um vencer sem ser vendido, publicado no GitHub Pages e
atualizado automaticamente todo dia via GitHub Actions, puxando os dados
direto da API do Omie.

> **Dados sensíveis ficam de fora.** Como o painel é público, clientes com
> contato (dados pessoais, LGPD), CMC e perda em R$ não são publicados. O
> script já sabe gerá-los (`INCLUIR_DADOS_SENSIVEIS` em
> `scripts/fetch_omie_data.py`), mas só podem voltar com aprovação formal da
> diretoria; a página mostra essas seções automaticamente quando os dados
> existem.
>
> **Complementar aos e-mails de alerta.** Os rascunhos quinzenais de e-mail
> continuam sendo gerados pela tarefa agendada do Claude. A política comercial
> de descontos fica só no e-mail interno, não neste painel.

## Estrutura do repositório

```
.
├── .github/workflows/update-dashboard.yml   # roda 1x por dia (e sob demanda)
├── scripts/
│   ├── fetch_omie_data.py                   # busca dados no Omie e gera docs/data.json
│   └── requirements.txt
└── docs/
    ├── index.html                           # o dashboard em si (HTML/CSS/JS puro, sem build)
    └── data.json                            # dados gerados pelo workflow
```

## Passo a passo para publicar

1. **Crie o repositório no GitHub** (pode ser público) e envie todo este
   conteúdo para ele (`git init`, `git add .`, `git commit`, `git remote add
   origin ...`, `git push`).

2. **Cadastre as credenciais do Omie como Secrets do repositório** — nunca
   deixe a chave em um arquivo versionado:
   - Vá em **Settings → Secrets and variables → Actions → New repository
     secret**.
   - Crie `OMIE_APP_KEY` com o App Key da integração "OMIE — Grupo Pigatto".
   - Crie `OMIE_APP_SECRET` com o App Secret correspondente.

3. **Habilite permissão de escrita para o GitHub Actions** (necessária para o
   workflow conseguir fazer commit do `docs/data.json` atualizado):
   - **Settings → Actions → General → Workflow permissions** → selecione
     **"Read and write permissions"** → Salvar.

4. **Habilite o GitHub Pages**:
   - **Settings → Pages → Build and deployment → Source**: "Deploy from a
     branch".
   - **Branch**: `main` (ou a branch padrão do seu repositório), pasta
     **`/docs`**.
   - Salve. O GitHub mostra a URL pública (algo como
     `https://SEU-USUARIO.github.io/SEU-REPOSITORIO/`) em alguns minutos.

5. **Rode o workflow manualmente uma vez** antes de esperar pelo agendamento
   automático: aba **Actions → Atualizar Dashboard de Lotes a Vencer → Run
   workflow**. Confira os logs e abra o painel.

6. Depois disso, o workflow roda sozinho **todo dia antes das 08:00 (horário
   de Brasília)** — agendado para 07:15 porque o GitHub atrasa o início em
   15–25 min. Também pode ser disparado manualmente a qualquer momento pela
   mesma tela de Actions. Páginas que ficam abertas (ex.: TV) buscam os dados
   novos sozinhas a cada 30 min.

## O que este dashboard mostra (e o que não mostra)

Mostra, agregado e atualizado automaticamente:
- 6 indicadores (lotes a vencer em 90 dias, urgência ≤30 dias, perda
  provável, risco parcial, sem giro em 12 meses, vende a tempo);
- tabela de risco por família, com a perda estimada de cada uma;
- uma tabela com todos os lotes (produto, família, lote, validade, dias para
  vencer, saldo, venda média por dia, dias para vender, saldo em risco e
  classificação de risco), com busca e filtros por família, prazo e risco.

**Como o risco é calculado:** para cada produto, a venda média diária vem dos
pedidos faturados e não cancelados dos últimos 12 meses. Supondo que o lote que
vence antes sai antes, o script projeta quanto do saldo de cada lote ainda
sobra na data de validade ("saldo em risco"). Classificação: *Perda provável*
(sobra ≥ 50% do saldo), *Risco parcial* (sobra algo), *Vende a tempo*, *Sem
giro* (nenhuma venda em 12 meses).

**Lotes já vencidos não aparecem no painel**: ficam só no e-mail interno
de vencidos, tratado pela operação. É uma projeção pela média: não
prevê sazonalidade nem pedidos pontuais.

## Pontos de atenção antes de confiar 100% nos números

- A busca de **lotes** (`ListarLotes`) e de **família do produto**
  (`ConsultarProduto`) replica exatamente a lógica já validada e em produção
  na tarefa agendada de e-mail (mesma janela de 90 dias, mesmo critério de
  saldo, mesmos buckets de urgência) — alto grau de confiança.
- A busca de **vendas dos últimos 12 meses** (`ListarPedidos`, endpoint
  `produtos/pedido`) foi conferida contra a conta real em 14/10/2026 e conta
  só pedidos faturados e não cancelados. Se essa busca falhar, o workflow fica
  vermelho e o `data.json` anterior é mantido, para não publicar risco errado.
## Segurança

- A chave do Omie nunca aparece em nenhum arquivo deste repositório — só
  existe como Secret do GitHub, acessível apenas durante a execução do
  workflow.
- O painel é público: não publique nele dados pessoais de clientes nem
  custos sem aprovação formal da diretoria.

## Rotina de alerta por e-mail

A definição da tarefa agendada que gera os rascunhos quinzenais de alerta de vencimento (Gmail) está versionada em [`rotina/`](rotina/README.md).
