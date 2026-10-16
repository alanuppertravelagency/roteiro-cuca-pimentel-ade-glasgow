// Roda todo dia (agendada no vercel.json) e envia por WhatsApp e/ou
// e-mail os itens do dia e os lembretes da planilha que tenham
// iso_date igual à data de hoje. Formato de função serverless da Vercel.
//
// Configuração necessária (Environment Variables no Vercel):
//   SHEET_CSV_URL, RECIPIENT_WHATSAPP, RECIPIENT_EMAIL,
//   TWILIO_ACCOUNT_SID, TWILIO_AUTH_TOKEN, TWILIO_WHATSAPP_FROM,
//   RESEND_API_KEY, RESEND_FROM

function parseCsv(text) {
  const rows = [];
  let row = [], field = "", inQuotes = false;
  for (let i = 0; i < text.length; i++) {
    const c = text[i];
    if (inQuotes) {
      if (c === '"' && text[i + 1] === '"') { field += '"'; i++; }
      else if (c === '"') { inQuotes = false; }
      else { field += c; }
    } else {
      if (c === '"') inQuotes = true;
      else if (c === ',') { row.push(field); field = ""; }
      else if (c === '\n' || c === '\r') {
        if (c === '\r' && text[i + 1] === '\n') i++;
        row.push(field); field = "";
        if (row.some(v => v.trim() !== "")) rows.push(row);
        row = [];
      } else field += c;
    }
  }
  if (field !== "" || row.length) { row.push(field); rows.push(row); }
  const headers = rows.shift().map(h => h.trim());
  return rows.map(r => Object.fromEntries(headers.map((h, i) => [h, (r[i] || "").trim()])));
}

function buildMessage(todayRows) {
  const first = todayRows[0];
  const lines = [`Bom dia! Roteiro de hoje (${first.day_theme}):`, ""];

  todayRows
    .filter(r => r.type !== "reminder")
    .forEach(r => lines.push(`${r.time ? r.time + " · " : ""}${r.title}`));

  const reminders = todayRows.filter(r => r.type === "reminder" && r.title);
  if (reminders.length) {
    lines.push("", "Lembretes:");
    reminders.forEach(r => lines.push(`- ${r.title}`));
  }

  return lines.join("\n");
}

async function sendWhatsapp(message) {
  const { TWILIO_ACCOUNT_SID, TWILIO_AUTH_TOKEN, TWILIO_WHATSAPP_FROM, RECIPIENT_WHATSAPP } = process.env;
  if (!TWILIO_ACCOUNT_SID || !TWILIO_AUTH_TOKEN || !TWILIO_WHATSAPP_FROM || !RECIPIENT_WHATSAPP) return;

  const auth = Buffer.from(`${TWILIO_ACCOUNT_SID}:${TWILIO_AUTH_TOKEN}`).toString("base64");
  await fetch(`https://api.twilio.com/2010-04-01/Accounts/${TWILIO_ACCOUNT_SID}/Messages.json`, {
    method: "POST",
    headers: {
      Authorization: `Basic ${auth}`,
      "Content-Type": "application/x-www-form-urlencoded"
    },
    body: new URLSearchParams({
      From: TWILIO_WHATSAPP_FROM,
      To: RECIPIENT_WHATSAPP,
      Body: message
    })
  });
}

async function sendEmail(message, subject) {
  const { RESEND_API_KEY, RESEND_FROM, RECIPIENT_EMAIL } = process.env;
  if (!RESEND_API_KEY || !RESEND_FROM || !RECIPIENT_EMAIL) return;

  await fetch("https://api.resend.com/emails", {
    method: "POST",
    headers: {
      Authorization: `Bearer ${RESEND_API_KEY}`,
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      from: RESEND_FROM,
      to: RECIPIENT_EMAIL,
      subject,
      text: message
    })
  });
}

module.exports = async (req, res) => {
  const { SHEET_CSV_URL } = process.env;
  if (!SHEET_CSV_URL) {
    return res.status(500).send("SHEET_CSV_URL não configurada.");
  }

  const sheetRes = await fetch(SHEET_CSV_URL);
  const rows = parseCsv(await sheetRes.text());

  const todayIso = new Date().toISOString().slice(0, 10);
  const todayRows = rows.filter(r => r.iso_date === todayIso);

  if (!todayRows.length) {
    return res.status(200).send("Sem itens de roteiro para hoje.");
  }

  const message = buildMessage(todayRows);
  await Promise.all([
    sendWhatsapp(message),
    sendEmail(message, `Roteiro de hoje · ${todayRows[0].day_theme}`)
  ]);

  return res.status(200).send("Lembretes enviados.");
};
