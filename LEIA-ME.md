# Roteiro de viagem — versão Vercel

Mesmo app, adaptado para publicar no Vercel em vez do Netlify (enquanto o problema de créditos do Netlify não se resolve).

## O que mudou em relação à versão Netlify

- As functions ficam em `api/` (não `netlify/functions/`)
- `TRAVEL_TIME_FUNCTION_URL` no index.html já aponta para `/api/travel-time`
- O agendamento diário dos lembretes está no `vercel.json` (em vez do `netlify.toml`)
- Não existe arquivo equivalente ao `netlify.toml` — o Vercel detecta a estrutura sozinho

## Passo a passo

1. **Importar a planilha e configurar o link**: mesmo processo de sempre — importe o `modelo-planilha.csv` no Google Sheets, publique como CSV, e cole o link no `SHEET_CSV_URL` do `index.html`.
2. **Subir os arquivos desta pasta pro GitHub** (pode ser um repositório novo, ou substituir os arquivos do repositório que você já criou).
3. **Criar conta no Vercel**: acesse [vercel.com](https://vercel.com) e entre com sua conta do GitHub.
4. **Importar o projeto**: clique em "Add New" → "Project", selecione o repositório, e clique em "Deploy". Não precisa mudar nenhuma configuração de build.
5. **Tornar público**: no plano gratuito do Vercel, os projetos já ficam públicos por padrão (ao contrário do Netlify) — não tem esse passo extra.

## Ativando deslocamento e lembretes (opcional)

No painel do Vercel: Project → Settings → Environment Variables. Adicione as mesmas variáveis de antes:
- `GOOGLE_MAPS_API_KEY` (deslocamento)
- `SHEET_CSV_URL`, `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`, `TWILIO_WHATSAPP_FROM`, `RECIPIENT_WHATSAPP` e/ou `RESEND_API_KEY`, `RESEND_FROM`, `RECIPIENT_EMAIL` (lembretes)

Depois de adicionar variáveis, é preciso fazer um novo deploy para elas passarem a valer (Deployments → menu "..." do último deploy → Redeploy).

**Nota sobre o cron job**: no plano gratuito (Hobby) do Vercel, jobs agendados rodam no máximo 1x por dia — o que já é exatamente o que a gente precisa (lembrete diário pela manhã).
