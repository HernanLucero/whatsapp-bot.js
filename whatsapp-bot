/**

- ╔══════════════════════════════════════════════╗
- ║     BOT GERENCIADOR DE COMUNIDAD WHATSAPP    ║
- ║     Powered by whatsapp-web.js               ║
- ╚══════════════════════════════════════════════╝
- 
- INSTALACIÓN:
- npm install whatsapp-web.js qrcode-terminal
- 
- USO:
- node whatsapp-bot.js
- Escanea el QR con tu WhatsApp → Listo!
  */

const { Client, LocalAuth, MessageMedia } = require(‘whatsapp-web.js’);
const qrcode = require(‘qrcode-terminal’);

// ──────────────────────────────────────────────
// CONFIGURACIÓN DEL BOT
// ──────────────────────────────────────────────
const CONFIG = {
nombre_comunidad: ‘🌟 Mi Comunidad’,
prefijo_comandos: ‘!’,
admins: [
// Agrega los números de teléfono de los admins (formato: 521234567890@c.us)
// Ejemplo: ‘5215512345678@c.us’
],

// Mensaje de bienvenida para nuevos miembros
bienvenida: (nombre) => `
╔══════════════════════════╗
║  ¡BIENVENID@ AL GRUPO!  ║
╚══════════════════════════╝

Hola *${nombre}* 👋

Nos alegra que te unas a nuestra comunidad.

📌 *Antes de empezar, lee las reglas:*
Escribe *!reglas* para verlas.

📋 *¿Qué puedo hacer?*
Escribe *!ayuda* para ver todos los comandos.

¡Esperamos que disfrutes la comunidad! 🎉
`.trim(),

// Reglas de la comunidad
reglas: `
📜 *REGLAS DE LA COMUNIDAD*

1️⃣ *Respeto* — Trata a todos con respeto y amabilidad.
2️⃣ *Sin spam* — No envíes publicidad no solicitada.
3️⃣ *Tema* — Mantén las conversaciones relevantes al grupo.
4️⃣ *Sin contenido ofensivo* — Nada de violencia, odio o discriminación.
5️⃣ *Privacidad* — No compartas datos personales de otros miembros.
6️⃣ *Admins* — Sus decisiones son finales. Respétalas.

⚠️ El incumplimiento puede resultar en expulsión.
`.trim(),

// Información del grupo
info: `
ℹ️ *INFORMACIÓN DE LA COMUNIDAD*

🏷️ *Nombre:* Mi Comunidad
📅 *Creada:* 2024
👥 *Tipo:* Comunidad abierta
📍 *Idioma:* Español

🔗 *Redes sociales:*
• Instagram: @micomunidad
• Web: www.micomunidad.com

💬 *¿Tienes dudas?* Escríbele a un admin.
`.trim(),

// Mensaje de ayuda
ayuda: `
🤖 *COMANDOS DISPONIBLES*

📌 *Información*
• `!ayuda` — Muestra este menú
• `!reglas` — Reglas del grupo
• `!info` — Información de la comunidad

📊 *Encuestas*
• `!encuesta “Pregunta” op1 | op2 | op3` — Crea una encuesta
• `!votar [id] [opción]` — Vota en una encuesta activa
• `!resultados [id]` — Ver resultados de una encuesta

👑 *Solo admins*
• `!anuncio [texto]` — Enviar anuncio destacado
• `!cerrar [id]` — Cerrar una encuesta

Escribe el comando y te respondo al instante 🚀
`.trim(),
};

// ──────────────────────────────────────────────
// ESTADO GLOBAL DE ENCUESTAS
// ──────────────────────────────────────────────
const encuestas = new Map();
let encuestaIdCounter = 1;

function crearEncuesta(pregunta, opciones, autorId, chatId) {
const id = encuestaIdCounter++;
encuestas.set(id, {
id,
pregunta,
opciones,
votos: opciones.reduce((acc, op) => ({ …acc, [op]: [] }), {}),
autorId,
chatId,
activa: true,
fechaCreacion: new Date(),
});
return id;
}

function votar(encuestaId, userId, opcionIndex) {
const enc = encuestas.get(encuestaId);
if (!enc) return { error: ‘❌ Encuesta no encontrada.’ };
if (!enc.activa) return { error: ‘❌ Esta encuesta ya está cerrada.’ };

const opcion = enc.opciones[opcionIndex - 1];
if (!opcion) return { error: `❌ Opción inválida. Elige entre 1 y ${enc.opciones.length}.` };

// Eliminar voto anterior si existe
for (const key in enc.votos) {
enc.votos[key] = enc.votos[key].filter((id) => id !== userId);
}
enc.votos[opcion].push(userId);
return { ok: true, opcion };
}

function resultadosEncuesta(encuestaId) {
const enc = encuestas.get(encuestaId);
if (!enc) return null;

const totalVotos = Object.values(enc.votos).reduce((sum, arr) => sum + arr.length, 0);
let txt = `📊 *${enc.pregunta}*\n`;
txt += enc.activa ? ‘🟢 Encuesta activa\n\n’ : ‘🔴 Encuesta cerrada\n\n’;

enc.opciones.forEach((op, i) => {
const count = enc.votos[op].length;
const pct = totalVotos > 0 ? Math.round((count / totalVotos) * 100) : 0;
const barra = ‘█’.repeat(Math.floor(pct / 10)) + ‘░’.repeat(10 - Math.floor(pct / 10));
txt += `${i + 1}. *${op}*\n   ${barra} ${count} votos (${pct}%)\n`;
});

txt += `\n👥 Total de votos: ${totalVotos}`;
return txt;
}

// ──────────────────────────────────────────────
// INICIALIZAR CLIENTE WHATSAPP
// ──────────────────────────────────────────────
const client = new Client({
authStrategy: new LocalAuth({ clientId: ‘comunidad-bot’ }),
puppeteer: {
headless: true,
args: [’–no-sandbox’, ‘–disable-setuid-sandbox’],
},
});

// ──────────────────────────────────────────────
// EVENTOS
// ──────────────────────────────────────────────

// Mostrar QR para autenticación
client.on(‘qr’, (qr) => {
console.log(’\n🔐 Escanea este QR con tu WhatsApp:\n’);
qrcode.generate(qr, { small: true });
});

// Bot listo
client.on(‘ready’, () => {
console.log(’\n✅ Bot conectado y listo!’);
console.log(`🤖 Gerenciando: ${CONFIG.nombre_comunidad}`);
console.log(‘━’.repeat(40));
});

// Bienvenida automática a nuevos miembros
client.on(‘group_join’, async (notification) => {
try {
const chat = await notification.getChat();
const contact = await notification.getContact();
const nombre = contact.pushname || contact.number;

```
await chat.sendMessage(CONFIG.bienvenida(nombre));
console.log(`👋 Nuevo miembro bienvenido: ${nombre}`);
```

} catch (err) {
console.error(‘Error en bienvenida:’, err.message);
}
});

// Procesar mensajes / comandos
client.on(‘message’, async (msg) => {
try {
const body = msg.body.trim();
const chat = await msg.getChat();
const sender = msg.from;

```
// Solo responder en grupos
if (!chat.isGroup) return;

// Verificar si es un comando
if (!body.startsWith(CONFIG.prefijo_comandos)) return;

const partes = body.slice(1).split(' ');
const comando = partes[0].toLowerCase();
const args = partes.slice(1);

console.log(`📩 Comando: !${comando} | Autor: ${sender}`);

// ── COMANDOS PÚBLICOS ──────────────────────

if (comando === 'ayuda') {
  await msg.reply(CONFIG.ayuda);
}

else if (comando === 'reglas') {
  await msg.reply(CONFIG.reglas);
}

else if (comando === 'info') {
  await msg.reply(CONFIG.info);
}

// ── ENCUESTAS ─────────────────────────────

else if (comando === 'encuesta') {
  // Uso: !encuesta "¿Pregunta?" op1 | op2 | op3
  const textoCompleto = args.join(' ');
  const matchPregunta = textoCompleto.match(/^"([^"]+)"\s+(.+)$/);

  if (!matchPregunta) {
    await msg.reply(
      '❌ Formato incorrecto.\n\n' +
      'Uso: `!encuesta "Tu pregunta aquí" Opción 1 | Opción 2 | Opción 3`\n\n' +
      'Ejemplo:\n`!encuesta "¿Cuál es tu color favorito?" Rojo | Azul | Verde`'
    );
    return;
  }

  const pregunta = matchPregunta[1];
  const opciones = matchPregunta[2].split('|').map((o) => o.trim()).filter(Boolean);

  if (opciones.length < 2) {
    await msg.reply('❌ Necesitas al menos 2 opciones.');
    return;
  }
  if (opciones.length > 6) {
    await msg.reply('❌ Máximo 6 opciones por encuesta.');
    return;
  }

  const id = crearEncuesta(pregunta, opciones, sender, chat.id._serialized);
  let respuesta = `📊 *NUEVA ENCUESTA #${id}*\n\n*${pregunta}*\n\n`;
  opciones.forEach((op, i) => {
    respuesta += `${i + 1}️⃣ ${op}\n`;
  });
  respuesta += `\n✏️ Vota con: \`!votar ${id} [número]\``;
  respuesta += `\n📈 Resultados: \`!resultados ${id}\``;

  await chat.sendMessage(respuesta);
}

else if (comando === 'votar') {
  // Uso: !votar [id] [opción]
  const encuestaId = parseInt(args[0]);
  const opcion = parseInt(args[1]);

  if (isNaN(encuestaId) || isNaN(opcion)) {
    await msg.reply('❌ Uso correcto: `!votar [id_encuesta] [número_opción]`\nEjemplo: `!votar 1 2`');
    return;
  }

  const resultado = votar(encuestaId, sender, opcion);
  if (resultado.error) {
    await msg.reply(resultado.error);
  } else {
    await msg.reply(`✅ Voto registrado por *${resultado.opcion}* en encuesta #${encuestaId}`);
  }
}

else if (comando === 'resultados') {
  // Uso: !resultados [id]
  const encuestaId = parseInt(args[0]);
  if (isNaN(encuestaId)) {
    await msg.reply('❌ Uso correcto: `!resultados [id_encuesta]`\nEjemplo: `!resultados 1`');
    return;
  }

  const res = resultadosEncuesta(encuestaId);
  if (!res) {
    await msg.reply('❌ Encuesta no encontrada.');
  } else {
    await msg.reply(res);
  }
}

// ── COMANDOS DE ADMIN ─────────────────────

else if (comando === 'anuncio') {
  const esAdmin = CONFIG.admins.includes(sender) || chat.participants?.find(
    (p) => p.id._serialized === sender && p.isAdmin
  );

  if (!esAdmin) {
    await msg.reply('⛔ Solo los administradores pueden enviar anuncios.');
    return;
  }

  const texto = args.join(' ');
  if (!texto) {
    await msg.reply('❌ Escribe el texto del anuncio.\nUso: `!anuncio Tu mensaje aquí`');
    return;
  }

  await chat.sendMessage(
    `📢 *ANUNCIO IMPORTANTE*\n\n${texto}\n\n— Administración de ${CONFIG.nombre_comunidad}`
  );
}

else if (comando === 'cerrar') {
  const esAdmin = CONFIG.admins.includes(sender) || chat.participants?.find(
    (p) => p.id._serialized === sender && p.isAdmin
  );

  if (!esAdmin) {
    await msg.reply('⛔ Solo los administradores pueden cerrar encuestas.');
    return;
  }

  const encuestaId = parseInt(args[0]);
  const enc = encuestas.get(encuestaId);
  if (!enc) {
    await msg.reply('❌ Encuesta no encontrada.');
    return;
  }

  enc.activa = false;
  const res = resultadosEncuesta(encuestaId);
  await chat.sendMessage(`🔴 *Encuesta #${encuestaId} cerrada*\n\n${res}`);
}

// Comando desconocido
else {
  await msg.reply(`❓ Comando desconocido. Escribe \`!ayuda\` para ver los comandos disponibles.`);
}
```

} catch (err) {
console.error(‘Error procesando mensaje:’, err.message);
}
});

// ──────────────────────────────────────────────
// INICIAR BOT
// ──────────────────────────────────────────────
client.initialize();
