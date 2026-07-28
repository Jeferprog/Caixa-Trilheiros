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
  Exceção controlada: a URL do backup no Google Sheets é lembrada via
  `localStorage` **protegido por `try/catch`** (inerte no artefato, útil no Pages).
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
- **Balancete de verificação** (na aba Diário/Razão, filtrável por exercício):
  todas as contas com débitos, créditos e saldo devedor/credor, provando que
  Σdébitos = Σcréditos. O saldo de abertura do banco aparece à parte, com nota
  de que a contrapartida patrimonial fica a cargo do contador.
- **Backup na nuvem (Google Sheets)**: salva transações + categorias/regras numa
  planilha via um *Apps Script Web App* que o usuário publica na própria conta
  (a página só faz `fetch`; nenhum SDK/CDN é carregado). Botões Salvar/Carregar
  na aba "Categorias e regras", com passo a passo e o código do script embutido.
- **Exportações**: CSV (transações, demonstrativo, diário, razão, balancete) com
  separador `;` e vírgula decimal (Excel pt-BR), além de Imprimir/PDF.
  Salvar/carregar configuração (categorias + regras) em JSON.

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
1. **Regras específicas do banco** da associação (ajustar palavras-chave conforme
   como o banco escreve tarifas, rendimentos, PIX de mensalidade etc.).
2. **Persistência completa** entre sessões via `localStorage` após publicar no
   GitHub Pages (hoje só a URL do Sheets é lembrada; transações/config ficam em
   memória e persistem via Sheets ou JSON).
3. Exportação em **XLSX** (SheetJS) além de CSV.
4. Opção de gerar os **lançamentos de encerramento** do exercício.
5. **Auto-salvar no Sheets** após cada mudança (hoje é manual, via botão).

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
