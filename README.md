const makeWASocket = require("@whiskeysockets/baileys").default
const { useMultiFileAuthState } = require("@whiskeysockets/baileys")
const P = require("pino")
const youtubedl = require("yt-dlp-exec")
const readline = require("readline")

const rl = readline.createInterface({ input: process.stdin, output: process.stdout })
const question = (text) => new Promise((resolve) => rl.question(text, resolve))

const BOT_NAME = "King Binu Boy - MD V2 ✨"

async function startBot() {
  const { state, saveCreds } = await useMultiFileAuthState('auth')
  const sock = makeWASocket({ logger: P({ level: "silent" }), auth: state })
  sock.ev.on("creds.update", saveCreds)

  if (!sock.authState.creds.registered) {
    const phoneNumber = await question("Number eka (9477...): ")
    const code = await sock.requestPairingCode(phoneNumber.trim())
    console.log(`\n👑 PAIRING CODE: ${code} - ${BOT_NAME}\n`)
  }
  sock.ev.on("connection.update", (u) => { if(u.connection === "open") { console.log(`✅ ${BOT_NAME} Connected! 👑`); rl.close() } })

  sock.ev.on("messages.upsert", async ({ messages }) => {
    const msg = messages[0]
    if (!msg.message || msg.key.fromMe) return
    const text = msg.message.conversation || msg.message.extendedTextMessage?.text || ""
    const jid = msg.key.remoteJid
    
    if (!text.startsWith(".")) {
        if (text.startsWith("http")) {
            await sock.sendMessage(jid, { text: `⏳ Video eka gannawa... ${BOT_NAME} 👑` })
            try {
                await youtubedl(text, { output: "video.mp4", format: "mp4/best" })
                await sock.sendMessage(jid, { video: { url: "./video.mp4" }, caption: `Menn video eka! 👑\n*${BOT_NAME}*` })
            } catch { await sock.sendMessage(jid, { text: "Download karanna baha 🥹" }) }
        }
        return
    }

    const lower = text.toLowerCase()
    const args = text.split(" ").slice(1).join(" ")

    if (lower === ".menu" || lower === ".help" || lower === ".hi" || lower === ".start" || lower === ".king") {
      await sock.sendMessage(jid, { text: `👑 *${BOT_NAME}* 👑
✨🥹 MD Version 2 ✨

*Commands:*
🎵 *.song [name]*
🎥 *.tiktok [link]*
🔍 *.image [name]*
📚 *.pastpaper [subject]*
📖 *.edu [question]*
🎬 *Link dammama auto download*

_© Powered by King Binu Boy_ 👑` })
      return
    }

    if (lower.startsWith(".song ") || lower.startsWith(".play ")) {
      if(!args) return sock.sendMessage(jid, {text: `Use: .song flow g 👑`})
      await sock.sendMessage(jid, { text: `🎵 "${args}" hoyanawa... - ${BOT_NAME} 👑` })
      try {
        await youtubedl(`ytsearch1:${args}`, { output: "song.mp3", extractAudio: true, audioFormat: "mp3" })
        await sock.sendMessage(jid, { audio: { url: "./song.mp3" }, mimetype: "audio/mpeg" })
        await sock.sendMessage(jid, { text: `🎧 *${args}* - Downloaded by ${BOT_NAME} 👑✨` })
      } catch { await sock.sendMessage(jid, { text: "Song eka ganna baha 🥹" }) }
    }

    else if (lower.startsWith(".tiktok ")) {
      await sock.sendMessage(jid, { text: `⏳ TikTok gannawa... ${BOT_NAME} 👑` })
      try {
        await youtubedl(args, { output: "tiktok.mp4", format: "mp4" })
        await sock.sendMessage(jid, { video: { url: "./tiktok.mp4" }, caption: `Menn TikTok eka! 👑\n${BOT_NAME} ✨🥹` })
      } catch { await sock.sendMessage(jid, { text: "Link eka check karanna 🥹" }) }
    }

    else if (lower.startsWith(".image ") || lower.startsWith(".img ")) {
      if(!args) return sock.sendMessage(jid, {text: `Use: .image king 👑`})
      await sock.sendMessage(jid, { image: { url: `https://source.unsplash.com/600x800/?${encodeURIComponent(args)}` }, caption: `🔍 ${args}\n*${BOT_NAME}* 👑✨` })
    }

    else if (lower.startsWith(".pastpaper ")) {
      await sock.sendMessage(jid, { text: `📚 *${args} Past Paper* - ${BOT_NAME} 👑\n\n🔗 e-thaksalawa.moe.gov.lk\n🔗 pastpapers.lk` })
    }

    else if (lower.startsWith(".edu ")) {
      await sock.sendMessage(jid, { text: `📖 *${args}* - ${BOT_NAME} 👑\n\nSoon AI add karamu! ✨🥹` })
    }

  })
}
startBot()
