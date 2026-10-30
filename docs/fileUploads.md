# File uploads

Several media methods in this library accept the sealed `TelegramFile` type, so you can send existing files and upload new ones without constructing multipart requests by hand. There are five variants: `ByFileId` reuses a file already on Telegram's servers, `ByUrl` asks Telegram to fetch a remote URL, `ByFile` reads a local `java.io.File`, `ByByteArray` uploads an in-memory blob, and `ByInputStream` streams bytes directly from an `InputStream` without buffering the entire file.

```kotlin
import com.github.kotlintelegrambot.entities.TelegramFile
import okhttp3.MediaType.Companion.toMediaType
import java.io.File

val byFileId = TelegramFile.ByFileId("AgACAgIAAxkBAAEC...")
val byUrl = TelegramFile.ByUrl("https://example.com/photo.jpg")
val byFile = TelegramFile.ByFile(File("/path/photo.jpg"))
val byBytes = TelegramFile.ByByteArray(fileBytes = photoBytes, filename = "photo.jpg")
val byStream = TelegramFile.ByInputStream(
    stream = httpResponse.body!!.byteStream(),
    filename = "photo.jpg",
    contentType = "image/jpeg".toMediaType(),
)
```

Every flavour goes through the same `sendPhoto` (or `sendAudio`, `sendVideo`, ...) call:

```kotlin
val bot = bot { token = BOT_API_TOKEN }
val chatId = ChatId.fromId(USER_CHAT_ID)

bot.sendPhoto(chatId, photo = byFileId)
bot.sendPhoto(chatId, photo = byUrl)
bot.sendPhoto(chatId, photo = byFile)
bot.sendPhoto(chatId, photo = byBytes, caption = "in-memory blob")
bot.sendPhoto(chatId, photo = byStream, caption = "streamed upload")
```

All five `TelegramFile` variants are supported by `sendPhoto`, `sendAudio`, `sendVideo`, `sendDocument`, `sendVoice` and `sendAnimation`. Other methods currently have narrower APIs: `sendSticker` takes a file ID or local file, `sendVideoNote` accepts only `TelegramFile.ByFile` or `ByFileId`, and `sendMediaGroup` / `editMessageMedia` use `InputMedia` wrappers without `ByInputStream` support. See [#361](https://github.com/kotlin-telegram-bot/kotlin-telegram-bot/issues/361) for the remaining streaming-upload work.

A small but important detail: when you build a `sendMediaGroup` out of `TelegramFile.ByByteArray` parts, give each part a **unique filename**. Telegram references the attached parts by filename in the multipart body, so two parts called `photo.jpg` will collide and the second one will silently overwrite the first.

For all the details about which formats and size limits Telegram accepts for each kind of file, see the official docs:

* https://core.telegram.org/bots/api#sending-files
