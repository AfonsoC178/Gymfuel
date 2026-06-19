# GymFuel — Integração Stripe (via Supabase Edge Function)

## 📂 Estrutura

```
index.html                                       ← Substitui o do teu repo
supabase/
  functions/
    stripe-checkout/
      index.ts                                   ← Edge Function (deploy)
  migrations/
    001_payment_transactions.sql                 ← (Opcional) Tabela transações
```

## 🚀 Como aplicar

### 1) Secret no Supabase
Vai a **Supabase → Project Settings → Edge Functions → Secrets** e confirma:
- `STRIPE_SECRET_KEY` = `sk_live_...` (ou `sk_test_...`)

### 2) (Opcional) Criar tabela de transações
No Supabase → SQL Editor, cola e executa o `001_payment_transactions.sql`.

### 3) Fazer deploy da Edge Function
Tens 2 opções:

**Via Supabase CLI:**
```bash
supabase functions deploy stripe-checkout --no-verify-jwt
```

**Via Supabase Dashboard:**
- Vai a Edge Functions → escolhe `stripe-checkout`
- Cola o conteúdo de `supabase/functions/stripe-checkout/index.ts`
- Em "Verify JWT" → **desmarca** (ou define a função como pública)
- Deploy

### 4) Substituir o index.html no Vercel
- Substitui o `index.html` do teu repositório GitHub pelo desta pasta
- Faz commit + push → o Vercel deploy automaticamente

## ✅ Como funciona

1. Cliente clica em "FINALIZAR COMPRA" no carrinho
2. Preenche dados de envio (passo 1)
3. Clica em "PAGAR €X.XX →" (passo 2)
4. O frontend chama a Edge Function `stripe-checkout` com os IDs dos produtos + quantidades
5. A Edge Function vai buscar os **preços REAIS** à BD (impossível manipular pelo browser)
6. Cria sessão Stripe Checkout com `line_items` (cada produto + quantidade)
7. Redireciona para o ambiente seguro do Stripe
8. Após pagamento → volta a `gymfuel.store/?stripe_session_id=...`
9. Frontend faz polling à Edge Function (GET) para confirmar `payment_status='paid'`
10. Grava a encomenda nas tabelas `encomendas` + `itens_encomenda`
11. Mostra "Encomenda confirmada!" e limpa o carrinho

## 🧪 Testar antes de produção

Usa o **modo de teste do Stripe**:
- Secret key: `sk_test_...`
- Cartão de teste: `4242 4242 4242 4242` · qualquer data futura · qualquer CVC

## 🔒 Segurança

- ✅ Preços vêm sempre da BD (nunca do browser)
- ✅ Service role só usada dentro da Edge Function
- ✅ Chave Stripe nunca exposta no frontend
- ✅ HTTPS obrigatório no redirecionamento
