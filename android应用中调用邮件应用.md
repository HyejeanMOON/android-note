虽然国内APP调用邮件应用情况很少，但是国外还是有一些应用会使用调用邮件应用的方式实现(~~其实是因为赶不上工期，用这个功能来凑数~~)。

调用邮件应用的方式还是老规矩的Intent，用的是ACTION_SENDTO的Flag。

整体代码如下：

```kotlin
class MailUtil(val context: Context) {

    /**
     *  发送邮件
     */
    fun sendMail() {
        // 制作邮件相关内容
        val address = "对方邮件地址"
        val subject = "邮件标题"  
        var message = "邮件内容"


        // 用intent来调用邮件
        // 邮件内容，包含对方邮件地址，主题以及邮件内容
        val mailContent =
            "mailto:$address?subject=$subject&body=${message.replace("\n", "</br>")}"
        val intent = Intent(Intent.ACTION_SENDTO)
        // 用Uri来解析邮件内容，并放入intent中
        intent.data = Uri.parse(mailContent)

        context?.also {
            it.startActivity(intent)
        }
    }
}
```

会发现上面邮件内容中有一段是关于`\n`替换成`</br>`字符串的代码。 
原因是邮件内容是以html形式来展示的，所以`\n`字符串不能是内容换行，
所以如果引用的内容中有`\n`字符串，则需要把它换成`</br>`。