
const express = require("express");

const app = express();
app.use(express.json());

const VERIFY_TOKEN = process.env.VERIFY_TOKEN;
const ACCESS_TOKEN = process.env.WHATSAPP_TOKEN;
const PHONE_NUMBER_ID = process.env.PHONE_NUMBER_ID;
const GRAPH_VERSION = process.env.GRAPH_VERSION || "v23.0";

app.get("/", (req, res) => {
  res.send("Sahee WhatsApp AI Bot is running!");
});

// WhatsApp webhook verification
app.get("/webhook", (req, res) => {
  const mode = req.query["hub.mode"];
  const token = req.query["hub.verify_token"];
  const challenge = req.query["hub.challenge"];

  if (mode === "subscribe" && token === VERIFY_TOKEN) {
    return res.status(200).send(challenge);
  }

  res.sendStatus(403);
});

// Receive WhatsApp messages
app.post("/webhook", async (req, res) => {
  res.sendStatus(200);

  try {
    const entry = req.body.entry?.[0];
    const change = entry?.changes?.[0];
    const message = change?.value?.messages?.[0];

    if (!message) return;

    const from = message.from;

    let reply = "Vanakkam! ❤️ Naan Sahee AI Assistant.";

    if (message.type === "text") {
      const text = message.text.body;
      reply = `Vanakkam! ❤️ Neenga sonnadhu: ${text}`;
    } else if (message.type === "video") {
      reply = "Ungal video vandhuvittadhu! Video processing feature seekiram varum. ❤️";
    } else if (message.type === "audio") {
      reply = "Ungal audio vandhuvittadhu! ❤️";
    }

    await fetch(
      `https://graph.facebook.com/${GRAPH_VERSION}/${PHONE_NUMBER_ID}/messages`,
      {
        method: "POST",
        headers: {
          Authorization: `Bearer ${ACCESS_TOKEN}`,
          "Content-Type": "application/json"
        },
        body: JSON.stringify({
          messaging_product: "whatsapp",
          recipient_type: "individual",
          to: from,
          type: "text",
          text: { body: reply }
        })
      }
    );

  } catch (error) {
    console.error("Bot error:", error);
  }
});

const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  console.log(`Sahee Bot running on port ${PORT}`);
});

{
  "name": "sahee-whatsapp-ai",
  "version": "1.0.0",
  "description": "Sahee WhatsApp AI Bot",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.21.2"
  }
}
