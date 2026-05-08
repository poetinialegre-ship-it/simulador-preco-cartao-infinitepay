# Simulador de Preço Pelo Cartão (InfinitePay) — CLAUDE.md

Documento de contexto e diretrizes do projeto. Mantenha este arquivo atualizado para que qualquer instância do Claude (Cowork, Claude Code, etc.) possa retomar o trabalho com contexto completo.

---

## 1. Visão geral

Calculadora HTML autocontida que simula o valor das parcelas e o total cobrado no cartão de crédito quando o vendedor usa o **modo "Repasse de Taxas"** (cliente paga as taxas, vendedor recebe o valor cheio), aplicando as taxas da **InfinitePay (Link de Pagamento, recebimento em 1 dia útil)**.

O usuário preenche o valor da venda → aparece a tabela de 1x a 12x com o valor da parcela e o total que o cliente paga no cartão, pronto para enviar ao cliente.

## 2. Contexto de negócio (importante)

- **O usuário NÃO cobra pela InfinitePay.** Ele cobra pelo **Asaas**.
- A InfinitePay é usada apenas como **referência de taxas** — o usuário gosta da tabela de taxas dela e quer aplicar a mesma lógica de repasse nas suas cobranças via Asaas.
- Por isso, **nenhum texto dos resumos pode mencionar "InfinitePay" ou "Link InfinitePay"** como meio de pagamento. A marca aparece apenas no cabeçalho da própria calculadora, como referência das taxas.
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
| 6x       | 9,67% | 12x      | 16,66% |

Observe o salto de 6x → 7x (9,67% → 12,59%), que reflete a mudança de faixa interna do plano.

## 4. Fórmula de cálculo (gross-up)

A InfinitePay aplica gross-up no modo repasse: o cliente paga um valor maior, calculado para que **após o desconto da taxa, o vendedor receba exatamente o valor da venda**.

```
total_cliente = valor_venda / (1 - taxa)
parcela       = total_cliente / n
```

Exemplo: venda de R$ 100,00 em 12x à taxa de 16,66% → total cliente = 100 / 0,8334 ≈ R$ 119,99 → 12 parcelas de R$ 10,00 cada → vendedor recebe R$ 100,00 líquidos.

**Não é simples adição** (× 1 + taxa). Isso foi confirmado pela própria UI do app, que mostra "Você recebe: R$ 100,00" no modo repasse.

## 5. Decisões de UI já fechadas

- **Layout em tabela**, não em cards/quadrados. Colunas: Parc. • Valor da parcela • Total no cartão • Taxa.
- Linhas de **1x e 12x destacadas** em verde-limão (cor da InfinitePay).
- Tema visual InfinitePay: preto + verde-limão (`#0a0a0a` e `#d2fc73`).
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

- Se as taxas da InfinitePay mudarem, atualize o objeto `TAXAS` no `<script>` do HTML e a tabela da seção 3 deste arquivo.
- Para verificar as taxas atuais: logar em `app.infinitepay.io` → "Cobrar" (Nova cobrança) → digitar qualquer valor → "Continuar" → "Parcelamento e taxas". A lista de "Ou você assume as parcelas" mostra os percentuais por parcela.
- Se o plano de recebimento mudar (de D+1 para "na hora"), as taxas serão maiores — re-extraia.
- O arquivo está em pasta com OneDrive, então qualquer salvamento já sincroniza para a nuvem do usuário.

## 9. Histórico das decisões nessa conversa

1. Usuário pediu uma calculadora baseada nas taxas do Link de Pagamento da InfinitePay, similar à que já existia para o Asaas (`D:\OneDrive\01 - CLAUDE\Projetos\Calculadora de Prestação ASAAS\calculadora-asaas.html`).
2. Confirmamos: recebimento **D+1**, modo **repasse de taxas para o cliente**, **taxas fixas** no HTML (sem campos editáveis).
3. Taxas extraídas direto da conta logada do usuário no Chrome (via Claude in Chrome MCP) para garantir valores reais — não confiar em listas genéricas de blogs.
4. Layout migrado de **cards** para **tabela** (Parc. / Valor da parcela / Total / Taxa) por preferência do usuário.
5. Removidas todas as menções a "Link InfinitePay" / "Pagamento via InfinitePay" nos resumos copiáveis, pois o usuário cobra pelo **Asaas** — a InfinitePay é só fonte das taxas.
6. Adicionada especificação "no cartão de crédito" e a lista de bandeiras aceitas pelo Asaas (Visa, Mastercard, Elo, Amex, Hipercard, Diners) em ambos os resumos.

## 10. Diretrizes de comportamento para o Claude

- **Não reintroduza menções à InfinitePay** nos textos copiáveis dos resumos. Os resumos vão para clientes finais que pagam via Asaas.
- **Não troque a fórmula gross-up** (`valor / (1 - taxa)`) por adição simples — isso quebraria o pressuposto "vendedor recebe o valor cheio".
- **Não mude as bandeiras** sem antes confirmar com o usuário — a lista está alinhada às bandeiras aceitas pelo Asaas.
- Se o usuário pedir para adicionar débito/Pix, lembre que essas operações têm taxas diferentes na InfinitePay (não estão na tabela atual).
- Sempre que mexer no HTML, valide o cálculo com um caso conhecido: R$ 100 em 12x → R$ 10,00/parcela, total R$ 119,99.
