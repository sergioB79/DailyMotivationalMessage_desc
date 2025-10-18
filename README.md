# 💬 Cria o teu Gerador Automático de Mensagens Diárias (Google + OpenAI)
### por [Sérgio Batalha](https://github.com/sergioB79)

Mensagens reais, personalizadas e diferentes todos os dias.  
Corre sozinho — até com o computador desligado ☁️  

---

## ⚡ O que é isto?

Um pequeno projeto em **Google Apps Script** que:

1️⃣ Lê o teu perfil (um ficheiro JSON no Google Drive);  
2️⃣ Usa a tua chave da **OpenAI (GPT-5)** para criar uma mensagem única;  
3️⃣ Guarda a mensagem no teu Drive;  
4️⃣ E envia-ta por **email — de manhã e à noite.**

---

## 🧠 Exemplo de mensagem recebida

> “A noite não é fuga, é o lugar onde regressas a ti.  
> Deixa o dia pousar sem ruído: o cansaço não acusa, testemunha.  
> Suaviza o foco sem o perder; atenção ampla, intenção simples.  
> Senta-te no teu centro: soberania interior, clara e silenciosa.  
> Hoje, dorme como um ato de soberania.”  
>  
> 📁 *Guardado em:* `MotivationDaily`

---

## ⚙️ O que precisas

1️⃣ Uma conta Google (Drive + Gmail)  
2️⃣ Uma **API Key** da [OpenAI](https://platform.openai.com/account/api-keys)  
3️⃣ 10 minutos e este guia

---

## 🧭 PASSO 1 — Criar o projeto no Google Apps Script

- Vai a 👉 [https://script.google.com/home](https://script.google.com/home)  
- Clica em **+ Novo Projeto**  
- Dá-lhe um nome (ex.: `Mensagens Diárias com IA`)

---

## 🧾 PASSO 2 — Criar o teu perfil de estilo no Drive

Cria um ficheiro chamado **`meu_perfil.json`** com o conteúdo:

{
  "identity": {
    "name": "Insere o teu nome aqui",
    "language": "Português (Portugal)",
    "tone": "direto, energético, filosófico"
  },
  "custom_prompts": {
    "morning_message": "Cria uma mensagem curta e motivacional, sem clichés, para despertar foco e ação.",
    "evening_message": "Cria uma reflexão calma e filosófica para fechar o dia, com presença e simplicidade."
  }
}

📁 Faz upload para o Google Drive.
📋 Abre o ficheiro → copia o ID (a parte entre /d/ e /view no link).

🔐 PASSO 3 — Guardar a tua API Key no projeto
No Apps Script:

Vai a Extensões → Propriedades do projeto → Propriedades do Script

Adiciona:
OPENAI_API_KEY = a_tua_chave_da_openai

💻 PASSO 4 — Colar o código
Copia o conteúdo de Code.gs para o teu projeto no Apps Script.

⚙️ Troca:

const fileId = 'COLOCA_AQUI_O_ID_DO_TEU_JSON';
const destinatario = 'TEU_EMAIL_AQUI';

⏰ PASSO 5 — Criar os relógios automáticos
No menu esquerdo do Apps Script:
1️⃣ Clica em ⏰ Acionadores (Triggers)
2️⃣ Clica em + Adicionar acionador

Função	Hora	Descrição
morningTrigger	10:00 – 11:00	Envia a motivação da manhã
eveningTrigger	23:00 – 00:00	Envia a reflexão da noite

Autoriza os acessos (Drive + Gmail + API externa).

🎯 PRONTO!
☀️ Às 10h → motivação diária criada por IA
🌙 Às 23h → reflexão noturna personalizada
💾 Tudo guardado no teu Drive
📧 Tudo enviado por email
💤 Mesmo com o computador desligado

💬 Dicas Rápidas
⚠️ Se vires “O Google não verificou este app” → Avançado → Ir para o projeto → Permitir
📭 Se não chega email → verifica Spam e autorizações
⏹ Para parar → vai a Acionadores → Apagar todos

⚡ Resumo Final
1 ficheiro JSON	Define o teu estilo e tom
1 script Apps Script	Faz tudo sozinho
2 triggers	Manhã e Noite
Resultado	Mensagens reais, diferentes e criadas só para ti

💸 Custo por chamada API: ~0,01€ (GPT-5)
🧩 100% automático e pessoal
☁️ 100% Google-native

💡 Expande a tua automação
Esta estrutura tem tudo o que precisas para criar projetos ainda mais poderosos:

🗞️ Recolher automaticamente notícias e posts de redes sociais
🧠 Gerar resumos e análise de sentimento
🤖 Enviar resultados para um bot de trading, dashboard ou assistente virtual

E o melhor?
🚀 Tudo com Google Apps Script — sem pagar um cêntimo.
O que no Zapier custa os olhos da cara 💸, aqui é gratuito, flexível e personalizável.

Automação + APIs + Lógica = resultados que parecem magia.

👨‍💻 Autor
Sérgio Batalha

“Automate your inspiration. Discipline is the bridge between dream and reality.”

---

## 🧱 **Code.gs (versão limpa para o repositório)**

```javascript
/**
 * Gerador Automático de Mensagens Diárias com GPT
 * Autor: Sérgio Batalha
 */

function sendDailyMessage(type) {
  const fileId = 'COLOCA_AQUI_O_ID_DO_TEU_JSON'; // ID do teu perfil JSON no Drive
  const profile = loadProfileFromDrive(fileId);

  const hoje = new Date().toISOString().slice(0, 10);
  const folderName = 'MotivationDaily';
  const folders = DriveApp.getFoldersByName(folderName);
  const folder = folders.hasNext() ? folders.next() : DriveApp.createFolder(folderName);

  const prompt = (type === 'morning')
    ? profile.custom_prompts.morning_message
    : profile.custom_prompts.evening_message;

  const fullPrompt = `
Perfil:
${JSON.stringify(profile.identity)}

Estilo e tom:
${JSON.stringify(profile.identity.tone)}

Instrução:
${prompt}
`;

  // ---- Chamada à OpenAI ----
  const apiKey = PropertiesService.getScriptProperties().getProperty('OPENAI_API_KEY');
  const payload = {
    model: "gpt-5",
    messages: [
      { role: "system", content: "És um mentor pessoal que escreve mensagens diárias únicas e personalizadas." },
      { role: "user", content: fullPrompt }
    ]
  };

  const response = UrlFetchApp.fetch("https://api.openai.com/v1/chat/completions", {
    method: "post",
    headers: {
      "Content-Type": "application/json",
      "Authorization": "Bearer " + apiKey
    },
    payload: JSON.stringify(payload)
  });

  const result = JSON.parse(response.getContentText());
  const message = result.choices?.[0]?.message?.content || "⚠️ Erro ao gerar mensagem.";

  // ---- Guardar no Drive ----
  const filename = (type === 'morning')
    ? `Motivation_${hoje}.md`
    : `Reflection_${hoje}.md`;

  const file = folder.createFile(filename, message, MimeType.PLAIN_TEXT);
  Logger.log('✅ Ficheiro criado: ' + file.getName());

  // ---- Enviar Email ----
  const destinatario = 'TEU_EMAIL_AQUI';
  const assunto = (type === 'morning' ? '☀️ Motivação Diária' : '🌙 Reflexão da Noite') + ' — ' + hoje;
  const corpo = `${message}\n\n📁 Guardado em: ${folder.getName()}`;

  MailApp.sendEmail(destinatario, assunto, corpo);
  Logger.log('📧 Email enviado: ' + assunto);
}

/**
 * Lê JSON de perfil no Drive
 */
function loadProfileFromDrive(fileId) {
  const url = `https://www.googleapis.com/drive/v3/files/${fileId}?alt=media`;
  const response = UrlFetchApp.fetch(url, {
    headers: { Authorization: 'Bearer ' + ScriptApp.getOAuthToken() }
  });
  return JSON.parse(response.getContentText());
}

/**
 * Acionadores automáticos (Triggers)
 */
function morningTrigger() {
  sendDailyMessage('morning');
}

function eveningTrigger() {
  sendDailyMessage('evening');
}
