const express = require('express');
const app = express();
app.use(express.json());

// === CONFIG MAX CLEAN ===
const VERIFY_TOKEN = 'maxclean123';
const PHONE_ID = process.env.PHONE_ID || '1376841848844620';
const TOKEN = process.env.TOKEN; // Tu token EAAgL30...
const PORT = process.env.PORT || 3000;

// Verificación webhook Meta
app.get('/webhook', (req, res) => {
  const mode = req.query['hub.mode'];
  const token = req.query['hub.verify_token'];
  const challenge = req.query['hub.challenge'];
  
  if (mode === 'subscribe' && token === VERIFY_TOKEN) {
    console.log('WEBHOOK VERIFICADO!');
    res.status(200).send(challenge);
  } else {
    res.sendStatus(403);
  }
});

// Recibir mensajes y contestar Max Clean
app.post('/webhook', async (req, res) => {
  try {
    const entry = req.body.entry?.[0];
    const change = entry?.changes?.[0];
    const message = change?.value?.messages?.[0];
    
    if (message) {
      const from = message.from; // número del cliente
      const text = message.text?.body?.toLowerCase() || '';
      
      console.log(`Mensaje de ${from}: ${text}`);

      // Mensaje de mayoreo Max Clean
      let reply = `¡Hola! 👋 Soy el asistente de *Max Clean* 🧼✨

*MAYOREO 12 PIEZAS:*
• 1kg = *$16* c/u
• 2kg = *$22* c/u

✅ Envío GRATIS a todo México
✅ Desinfectante y jabón premium que sí deja aroma

¿Cuántas piezas necesitas? Mándame tu código postal para cotizarte el envío 🚚`;

      // Si preguntan por precio, menú, etc - mismo mensaje
      if (text.includes('precio') || text.includes('mayoreo') || text.includes('costo') || text.includes('hola') || text.includes('info')) {
        // usa el mensaje de arriba
      }

      // Enviar respuesta por API WhatsApp
      await fetch(`https://graph.facebook.com/v22.0/${PHONE_ID}/messages`, {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${TOKEN}`,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          messaging_product: 'whatsapp',
          to: from,
          type: 'text',
          text: { body: reply }
        })
      });
    }
    
    res.sendStatus(200);
  } catch (e) {
    console.error(e);
    res.sendStatus(200);
  }
});

app.get('/', (req, res) => {
  res.send('Bot Max Clean Activo ✅ - Webhook: /webhook');
});

app.listen(PORT, () => console.log(`Bot Max Clean corriendo en puerto ${PORT}`));
