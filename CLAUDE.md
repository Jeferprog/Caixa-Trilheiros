# Livro Caixa — Associação Trilheiros Tudo Loko

Ferramenta web para a contabilidade de uma associação sem fins lucrativos:
importa extrato bancário em **OFX**, categoriza as transações e gera
**demonstrativo de receitas e despesas** (mensal e anual) e **Livro Diário e
Razão** em partidas dobradas.

> Este arquivo é o briefing do projeto. O Claude Code o lê automaticamente ao
> abrir esta pasta.

## Arquivo principal
- `livro-caixa-trilheiros.html` — aplicação inteira num único arquivo (HTML +
  CSS + JavaScript vanilla). **Sem etapa de build, sem dependências externas.**
  Roda com duplo-clique no navegador ou publicado no GitHub Pages.

## Restrições de arquitetura (importantes)
- **Vanilla JS**, sem frameworks nem CDNs — precisa funcionar offline e no GitHub Pages.
- **Não usar `localStorage`/`sessionStorage`** enquanto for pré-visualizado como
  artefato do Claude (falha lá). Estado fica em memória; a persistência das
  categorias/regras é feita por **exportar/importar um JSON de configuração**.
  (Ao publicar no GitHub Pages, `localStorage` passa a ser uma opção viável.)
- Sem `<form>` com submit; usar handlers de evento (`onclick`, `onchange`).

## O que já está implementado
- **Leitura de OFX** nos dois formatos brasileiros: 1.x (SGML, tags sem
  fechamento) e 2.x (XML). Detecção de codificação (windows-1252 / UTF-8) e
  deduplicação por `FITID`.
- **Categorização** automática por regras de palavra-chave (editáveis) +
  ajuste manual por transação. Categorias padrão pensadas para associação.
- **Demonstrativo mensal e anual** (regime de caixa): receitas e despesas por
  categoria, resultado do período e saldo acumulado. Campo de saldo inicial com
  conferência contra o saldo informado no OFX (`LEDGERBAL`).
- **Livro Diário e Razão** (partidas dobradas) — ver decisões abaixo.
- **Exportações**: CSV (transações, demonstrativo, diário, razão) com separador
  `;` e vírgula decimal (Excel pt-BR), além de Imprimir/PDF. Salvar/carregar
  configuração (categorias + regras) em JSON.

## Decisões contábeis (não alterar sem motivo)
- Regime de **caixa**: tudo deriva do que passou na conta bancária. **Não é**
  Balanço Patrimonial — esse é responsabilidade do contador.
- Dupla entrada: **entrada** → D `1.1.1.01 Banco Conta Movimento` / C conta de
  receita; **saída** → D conta de despesa / C Banco.
- Plano de contas gerado a partir das categorias: receitas em `3.xx`, despesas
  em `4.xx`, banco em `1.1.1.01`. Transações "Não classificado" caem em
  `3.99 Receitas a classificar` / `4.99 Despesas a classificar` para os livros
  nunca deixarem de fechar.
- Livro Diário/Razão filtráveis por **exercício** (ano). O saldo de abertura do
  banco de um exercício = saldo inicial global + soma dos anos anteriores.
- **Lançamentos de encerramento do exercício** (zerar receitas/despesas contra o
  resultado) foram deixados de fora de propósito — quem faz é o contador.

## Contexto fiscal (por que a ferramenta existe assim)
- Associação sem fins lucrativos, **isenta** de IRPJ/CSLL.
- Teve **rendimentos de aplicação financeira** com **IRRF retido na fonte**; para
  entidade isenta essa retenção é **definitiva** (sem imposto adicional).
- A declaração que reporta esses valores é a **ECF**; estes livros são a base.

## Próximos passos sugeridos (backlog)
1. **Balancete de verificação** — tabela final com todas as contas + total de
   débitos e créditos, provando que fecham.
2. **Persistência** entre sessões via `localStorage` (após publicar no GitHub Pages).
3. **Regras específicas do banco** da associação (ajustar palavras-chave conforme
   como o banco escreve tarifas, rendimentos, PIX de mensalidade etc.).
4. Exportação em **XLSX** (SheetJS) além de CSV.
5. Opção de gerar os **lançamentos de encerramento** do exercício.

## Como testar rápido
Abrir o HTML no navegador e arrastar um `.ofx`. Formato mínimo de um lançamento:
```
<STMTTRN>
<TRNTYPE>CREDIT
<DTPOSTED>20250105
<TRNAMT>50.00
<FITID>ID001
<MEMO>MENSALIDADE FULANO
</STMTTRN>
```
