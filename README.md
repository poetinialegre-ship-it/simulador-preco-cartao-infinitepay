# Simulador de Preço Pelo Cartão (InfinitePay)

Calculadora HTML autocontida para simular o valor das parcelas e o total cobrado no cartão de crédito quando se aplica o **modo "Repasse de Taxas"** com as taxas oficiais do Link de Pagamento da **InfinitePay** (recebimento em 1 dia útil).

> ⚠️ **Atenção**: a cobrança é feita via Asaas. A InfinitePay é referência apenas para as taxas. Veja [CLAUDE.md](./CLAUDE.md) para o contexto completo.

## Acesse online

🔗 **https://poetinialegre-ship-it.github.io/simulador-preco-cartao-infinitepay/**

## Como usar

Abra o link acima (ou rode `index.html` localmente), digite o valor da venda e a tabela de 1x a 12x aparece automaticamente. Use os botões para copiar o resumo e mandar pro cliente (texto plano ou WhatsApp).

## Fórmula

```
total_cliente = valor_venda / (1 - taxa)
parcela        = total_cliente / n
```

Gross-up para que o vendedor receba sempre o valor cheio.

## Bandeiras aceitas

Visa, Mastercard, Elo, American Express, Hipercard e Diners Club (alinhadas ao Asaas).

## Documentação

- [`CLAUDE.md`](./CLAUDE.md) — diretrizes completas do projeto, contexto de negócio, taxas, decisões de UI e histórico.
- [`index.html`](./index.html) — arquivo único, autocontido (HTML + CSS + JS).

## Atualizando depois

Sempre que mudar algo, dentro da pasta:

```powershell
git add .
git commit -m "descrição da mudança"
git push
```

O GitHub Pages atualiza sozinho em ~30s.
