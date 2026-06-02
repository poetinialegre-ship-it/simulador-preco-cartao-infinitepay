# Simulador de Preço Pelo Cartão (InfinitePay) — CLAUDE.md

Documento de contexto e diretrizes do projeto. Mantenha este arquivo atualizado para que qualquer instância do Claude (Cowork, Claude Code, etc.) possa retomar o trabalho com contexto completo.

---

## 1. Visão geral

Calculadora HTML autocontida que simula o valor das parcelas e o total cobrado no cartão de crédito quando o vendedor usa o **modo "Repasse de Taxas"** (cliente paga as taxas, vendedor recebe o valor cheio), aplicando as taxas da **InfinitePay (Link de Pagamento, recebimento em 1 dia útil)**.

O usuário preenche o valor da venda → aparece a tabela de 1x a 12x com o valor da parcela e o total que o cliente paga no cartão, pronto para enviar ao cliente.

## 2. Contexto de negócio (importante)

- **O usuário NÃO cobra pela InfinitePay.** Ele cobra pelo **Asaas**.
- A InfinitePay é usada apenas como **referência de taxas** — o usuário gosta da tabela de taxas dela e quer aplicar a mesma lógica de repasse nas suas cobranças via Asaas.
- Por isso, **a página foi descaracterizada da marca InfinitePay** (sessão 2026-06-01): nem os resumos copiáveis, nem o cabeçalho, nem o `<title>`, nem o rodapé mencionam "InfinitePay". A paleta de cores também foi trocada pra não lembrar a marca. O único lugar onde "infinitepay" ainda aparece é no **nome do repositório/URL pública**, que não é renomeado pra não quebrar o link já compartilhado.
- As bandeiras aceitas devem ser as **bandeiras aceitas pelo Asaas**: Visa, Mastercard, Elo, American Express, Hipercard e Diners Club.

## 3. Taxas oficiais (extraídas da conta do usuário em 08/05/2026)

Plano: **Link de Pagamento • Recebimento em 1 dia útil (D+1) • Repassar taxas para o cliente**.

| Parcelas | Taxa  | Parcelas | Taxa   |
|---------:|------:|---------:|-------:|
| 1x       | 4,20% | 7x       | 12,59% |
| 2x       | 6,09% | 8x       | 13,42% |
| 3x       | 7,01% | 9x       | 14,25% |
| 4x       | 7,91% | 10x      | 15,06% |
| 5x       | 8,80% | 11x      | 15,87% |
| 6x       | 9,67% | 12x      | 16,6667%\* |

\* **12x diverge da taxa oficial InfinitePay (16,66%)** desde 2026-06-01. Foi ajustada para `16.6667` (≈ 1/6) para que R$ 10.000 em 12x dê **R$ 1.000,00/parcela** e **R$ 12.000,00 total** exatos — pedido do usuário pra simplificar a comunicação com o cliente. Display arredonda pra "16,67%". As taxas de 1x a 11x continuam as oficiais.

Observe o salto de 6x → 7x (9,67% → 12,59%), que reflete a mudança de faixa interna do plano.

## 4. Fórmula de cálculo (gross-up)

A InfinitePay aplica gross-up no modo repasse: o cliente paga um valor maior, calculado para que **após o desconto da taxa, o vendedor receba exatamente o valor da venda**.

```
total_cliente = valor_venda / (1 - taxa)
parcela       = total_cliente / n
```

Exemplo: venda de R$ 10.000,00 em 12x à taxa de 16,6667% → total cliente = 10.000 / 0,833333 ≈ R$ 12.000,00 → 12 parcelas de R$ 1.000,00 cada → vendedor recebe R$ 10.000,00 líquidos.

**Não é simples adição** (× 1 + taxa). Isso foi confirmado pela própria UI do app, que mostra "Você recebe: R$ 100,00" no modo repasse.

## 5. Decisões de UI já fechadas

- **Layout em tabela**, não em cards/quadrados. Colunas: Parc. • Valor da parcela • Total no cartão • Taxa.
- Linhas de **1x e 12x destacadas** em âmbar dourado.
- Tema visual: **slate-navy + âmbar dourado** (`--c-dark: #0f172a`, `--c-accent: #fbbf24`, `--c-accent-dark: #f59e0b`). Variáveis CSS prefixadas com `--c-` (não `--ip-`). Versão anterior era preto + verde-limão (`#0a0a0a` / `#d2fc73`) — foi trocada em 2026-06-01 pra desvincular visualmente da InfinitePay.
- Input de valor da venda com máscara BRL e prefixo "R$".
- Botões para **copiar resumo** (texto plano) e **copiar formato WhatsApp** (com `*negrito*` e `_itálico_`).
- Pílulas informativas no topo do cartão de input: "Recebe em 1 dia útil", "Cliente paga em até 12x", "Você recebe o valor cheio".

## 6. Formato dos resumos (texto copiado)

Os dois formatos seguem essa estrutura mínima — **sem mencionar InfinitePay**:

**Formato simples:**

```
OPÇÕES DE PARCELAMENTO NO CARTÃO DE CRÉDITO
Bandeiras aceitas: Visa, Mastercard, Elo, American Express, Hipercard e Diners Club.
────────────────────────────────────────
Valor da compra: R$ <valor>
────────────────────────────────────────
 1x de R$ <parc>  →  Total: R$ <total>
 ...
12x de R$ <parc>  →  Total: R$ <total>
────────────────────────────────────────
```

**Formato WhatsApp:**

```
*💳 OPÇÕES DE PARCELAMENTO NO CARTÃO DE CRÉDITO*
_Bandeiras aceitas: Visa, Mastercard, Elo, American Express, Hipercard e Diners Club._

*Valor da compra:* R$ <valor>

1x de R$ <parc>  _(total R$ <total>)_
...
12x de R$ <parc>  _(total R$ <total>)_
```

## 7. Estrutura de arquivos

```
Simulador de Preço Pelo Cartão (InfinitePay)/
├── index.html      ← arquivo único, autocontido (servido pelo GitHub Pages)
├── README.md       ← descrição pública
└── CLAUDE.md       ← este documento
```

Publicado em: https://poetinialegre-ship-it.github.io/simulador-preco-cartao-infinitepay/

Tudo (HTML + CSS + JS) está em **um único arquivo HTML**, sem dependências externas. Pode ser aberto direto no navegador (file://) ou hospedado em qualquer lugar estático.

## 8. Como manter / evoluir

- Se as taxas da InfinitePay mudarem, atualize o objeto `TAXAS` no `<script>` do `index.html` e a tabela da seção 3 deste arquivo. **Cuidado:** 12x não é a taxa oficial — é customizada em 16,6667% (vide seção 3). Não sobrescreva sem checar.
- Para verificar as taxas atuais: logar em `app.infinitepay.io` → "Cobrar" (Nova cobrança) → digitar qualquer valor → "Continuar" → "Parcelamento e taxas". A lista de "Ou você assume as parcelas" mostra os percentuais por parcela.
- Se o plano de recebimento mudar (de D+1 para "na hora"), as taxas serão maiores — re-extraia.
- O arquivo está em pasta com OneDrive, então qualquer salvamento já sincroniza para a nuvem do usuário.
- **Para republicar a versão online:** dentro da pasta, `git add . && git commit -m "..." && git push`. O GitHub Pages republica em ~30s. Detalhes na seção 11.

## 9. Histórico das decisões nessa conversa

1. Usuário pediu uma calculadora baseada nas taxas do Link de Pagamento da InfinitePay, similar à que já existia para o Asaas (`D:\OneDrive\01 - CLAUDE\Projetos\Calculadora de Prestação ASAAS\calculadora-asaas.html`).
2. Confirmamos: recebimento **D+1**, modo **repasse de taxas para o cliente**, **taxas fixas** no HTML (sem campos editáveis).
3. Taxas extraídas direto da conta logada do usuário no Chrome (via Claude in Chrome MCP) para garantir valores reais — não confiar em listas genéricas de blogs.
4. Layout migrado de **cards** para **tabela** (Parc. / Valor da parcela / Total / Taxa) por preferência do usuário.
5. Removidas todas as menções a "Link InfinitePay" / "Pagamento via InfinitePay" nos resumos copiáveis, pois o usuário cobra pelo **Asaas** — a InfinitePay é só fonte das taxas.
6. Adicionada especificação "no cartão de crédito" e a lista de bandeiras aceitas pelo Asaas (Visa, Mastercard, Elo, Amex, Hipercard, Diners) em ambos os resumos.
7. **Publicação no GitHub Pages (08/05/2026)** — usuário pediu para hospedar online pra poder mandar o link pros clientes. Decisões tomadas:
   - Conta: `poetinialegre-ship-it` (única logada no `gh`, plano Free).
   - Repo: `simulador-preco-cartao-infinitepay`, **público** (Pages em repo privado exigiria GitHub Pro — não justifica).
   - Branch `main`, source path `/` (root), build legacy.
   - Renomeado `calculadora-infinitepay.html` → `index.html` para a URL ficar limpa (`/<repo>/` em vez de `/<repo>/calculadora-infinitepay.html`). Conteúdo do HTML não foi alterado.
   - URL final: https://poetinialegre-ship-it.github.io/simulador-preco-cartao-infinitepay/
8. Removida pasta `.git/` corrompida que tinha sobrado de uma sandbox anterior (Cowork) que travou no meio de um `git init` — `git status` falhava com "fatal: not a git repository". Foi recriado do zero antes do push.
9. Removido `PUBLICAR-NO-GITHUB.md` — era um guia manual em PowerShell pra publicação que ficou obsoleto após o deploy automatizado. Instruções de update foram pro README.md.
10. **Sessão 2026-06-01 — descaracterização da marca InfinitePay e ajuste do 12x:**
    - Removido o badge "InfinitePay" do cabeçalho da página.
    - Removido "InfinitePay" do `<title>` da aba (agora "Simulador de Parcelamento no Cartão").
    - Removido do rodapé (agora "Taxas com repasse para o cliente (Plano D+1).").
    - Paleta visual trocada de preto + verde-limão (`#0a0a0a` / `#d2fc73`) para **slate-navy + âmbar dourado** (`#0f172a` / `#fbbf24`). Variáveis CSS renomeadas de `--ip-*` para `--c-*` (e `--c-black`/`--c-lime` viraram `--c-dark`/`--c-accent`).
    - **Taxa de 12x ajustada de 16,66% (oficial InfinitePay) para 16,6667%** para que R$ 10.000 em 12x dê exatamente R$ 1.000,00/parcela. Display arredondado mostra "16,67%". Taxas de 1x a 11x permanecem oficiais.
    - O nome do repositório/URL pública (`simulador-preco-cartao-infinitepay`) **não foi renomeado** porque já há links compartilhados em circulação.

## 10. Diretrizes de comportamento para o Claude

- **Sempre, ao final de uma atualização** (no `index.html`, fluxo, taxa, layout, deploy, qualquer coisa), **pergunte explicitamente ao usuário se ele quer que o `CLAUDE.md` seja atualizado** com as mudanças. Não atualize silenciosamente nem espere ele lembrar — a pergunta tem que partir de você. O objetivo é que este arquivo nunca fique defasado em relação ao estado real do projeto. Pergunte mesmo que a mudança pareça pequena; é melhor um "não precisa" do usuário do que perder contexto.
- **Não reintroduza menções à InfinitePay** nos textos copiáveis dos resumos nem em elementos visíveis da página (cabeçalho, `<title>`, rodapé, badge). Os resumos vão para clientes finais que pagam via Asaas, e a página foi descaracterizada da marca em 2026-06-01.
- **Não troque a fórmula gross-up** (`valor / (1 - taxa)`) por adição simples — isso quebraria o pressuposto "vendedor recebe o valor cheio".
- **Não mude as bandeiras** sem antes confirmar com o usuário — a lista está alinhada às bandeiras aceitas pelo Asaas.
- Se o usuário pedir para adicionar débito/Pix, lembre que essas operações têm taxas diferentes na InfinitePay (não estão na tabela atual).
- Sempre que mexer no HTML, valide o cálculo com um caso conhecido: **R$ 10.000 em 12x → R$ 1.000,00/parcela, total R$ 12.000,00** (com `TAXAS[12] = 16.6667`). Caso secundário: R$ 100 em 12x → R$ 10,00/parcela, total R$ 120,00.
- O arquivo principal **agora se chama `index.html`** (não `calculadora-infinitepay.html`) — necessário pro GitHub Pages servir na raiz do repo. Não renomeie de volta.
- Toda mudança no `index.html` deve ser commitada e pushada (vide seção 11) — caso contrário, a versão online fica desatualizada em relação à local.

## 11. Deploy / hospedagem

- **Plataforma:** GitHub Pages (gratuito, repo público).
- **Repo:** https://github.com/poetinialegre-ship-it/simulador-preco-cartao-infinitepay
- **URL pública:** https://poetinialegre-ship-it.github.io/simulador-preco-cartao-infinitepay/
- **Source:** branch `main`, path `/` (root). O Pages serve `index.html` automaticamente.
- **Build:** legacy (sem Jekyll customizado — só serve os arquivos como estão).
- **Atualizar versão online:** commit + push na `main` → Pages republica em ~30s.
- **Comandos úteis:**
  - `gh repo view poetinialegre-ship-it/simulador-preco-cartao-infinitepay --web` — abre o repo no navegador.
  - `gh api repos/poetinialegre-ship-it/simulador-preco-cartao-infinitepay/pages --jq .status` — checa status do build (`built`, `building`, `errored`).
  - `gh api repos/poetinialegre-ship-it/simulador-preco-cartao-infinitepay/pages/builds/latest` — detalhes do último build (útil quando dá erro).
- **Reinstalando o ambiente em outra máquina:**
  ```powershell
  gh auth login                                                            # se ainda não logou
  gh repo clone poetinialegre-ship-it/simulador-preco-cartao-infinitepay   # clona o repo
  cd simulador-preco-cartao-infinitepay
  ```
- **Por que Pages e não outro lugar:** custo zero, sem servidor, e o repo já fica versionado. A cobrança continua sendo feita pelo Asaas — Pages só hospeda a calculadora estática (vide seção 2).
