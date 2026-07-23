package com.example.fileconverter

import android.content.ContentResolver
import android.net.Uri
import okhttp3.MediaType
import okhttp3.RequestBody
import okio.Buffer
import okio.BufferedSink
import okio.ForwardingSink
import okio.Sink
import okio.buffer

/**
 * RequestBody يقرأ من Uri عبر ContentResolver ويبلغ عن نسبة التقدم أثناء الرفع،
 * مما يمكّننا من تحديث شريط التقدم في الواجهة أثناء رفع الملف إلى السيرفر.
 */
class ProgressRequestBody(
    private val contentResolver: ContentResolver,
    private val uri: Uri,
    private val contentType: MediaType?,
    private val contentLength: Long,
    private val onProgress: (percent: Int) -> Unit
) : RequestBody() {

    override fun contentType(): MediaType? = contentType

    override fun contentLength(): Long = contentLength

    override fun writeTo(sink: BufferedSink) {
        val inputStream = contentResolver.openInputStream(uri)
            ?: throw IllegalStateException("تعذر فتح الملف المحدد")

        inputStream.use { input ->
            val buffer = ByteArray(8192)
            var totalRead = 0L
            var read: Int
            while (input.read(buffer).also { read = it } != -1) {
                sink.write(buffer, 0, read)
                totalRead += read
                if (contentLength > 0) {
                    val percent = ((totalRead * 100) / contentLength).toInt().coerceIn(0, 100)
                    onProgress(percent)
                }
            }
        }
    }
}
