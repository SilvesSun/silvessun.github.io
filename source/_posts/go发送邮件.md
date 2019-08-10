---
title: go发送邮件
date: 2018-06-27 11:42:42
tags: 
- 邮件
- gomail
- smtp
categories:
- golang学习
---

![go图标](http://blogsk.oss-us-west-1.aliyuncs.com/golang.png)

今天来说说go中邮件服务相关的内容

我们知道,  简单邮件传送服务基于smtp协议. 但是smtp协议存在几个缺点:

- SMTP 不能传送可执行文件或者其他二进制对象；
- SMTP 只能传送7位的 ASCII 码，许多国家非英文的文字将无法传送；
- SMTP 服务器会拒绝超过一定长度的邮件；
- 某些 SMTP 的实现并没有完全按照 SMTP 的因特网标准。

为了弥补这些缺陷, 新增了MIME. MIME在smtp的基础上增加了邮件主体的结构，并定义了传送非 ASCII 码的编码规则. MIME消息由消息头和消息体两大部分组成

# MIME邮件的格式
## 邮件头
邮件头中包含了发件人, 收件人, 主题, 时间, MIME版本, 邮件内容的类型等重要信息. 每条信息称为一个域，由域名后加“: ”和信息内容构成，可以是一行，较长的也可以占用多行。域的首行必须“顶头”写，即左边不能有空白字符（空格和制表符）；续行则必须以空白字符打头，且第 一个空白字符不是信息本身固有的，解码时要过滤掉.

![](http://p3euxxfa8.bkt.clouddn.com//18-6-27/84540210.jpg)

## 邮件体
邮件体包含邮件的内容，它的类型由邮件头的“Content-Type”域指出。常见的简单类型有text/plain(纯文本)和text/html(超文本).

如果在邮件中要添加附件，必须定义multipart/mixed段；如果存在内嵌资源，至少要定义multipart/related段；如果纯文本与超文本共存，至少要定义multipart/alternative段

mixed段允许单个报文含有多个相互独立的子报文，每个子报文拥有自己的类型和编码。在 mixed 后面用到 boundary=BOUNDARY 关键字，定义分隔各部分子报文的分隔符，在邮件用利用 --BOUNDARY 进行分隔

![2018-06-27-12-11-33](http://p3euxxfa8.bkt.clouddn.com/2018-06-27-12-11-33.png)

# go中smtp接口
正如前文所述, smtp实现了简单的邮件传输协议. 一个简单的实例如下:

```go
// host := "smtp.qq.com:25"
func SendToMail(user, passwd, host, to, subject, body, mailtype string) error {
	hp := strings.Split(host, ":")
	fmt.Println(hp)
	auth := smtp.PlainAuth("", user, passwd, hp[0])
	var content_type string
	if mailtype == "html" {
		content_type = "Content-Type: text/" + mailtype + "; charset=UTF-8"
	} else {
		content_type = "Content-Type: text/plain" + "; charset=UTF-8"
	}

	msg := []byte("To: " + to + "\r\nFrom: " + user + "\r\nSubject: " +subject+ "\r\n" + content_type + "\r\n\r\n" + body)
	send_to := strings.Split(to, ";")
	err := smtp.SendMail(host, auth, user, send_to, msg)
	return err
}

```

这种邮件可以发送简单的文本邮件, 如果需要发送附件, 则需要构建 mime 消息

```go
func main() {

	to := "**********"
	cc := "xxxxxxxxxxxxx"
	mime := bytes.NewBuffer(nil)
	//设置邮件

	boundary := "---test"
	mime.WriteString("From: sk<"+USER+">\r\nTo: "+to+"\r\nCc:"+cc +"\r\nSubject: 附件测试\r\nMIME-Version: 1.0\r\n")
	mime.WriteString("Content-Type: multipart/mixed; boundary="+boundary+"\r\n\r\n")

	mime.WriteString("--"+boundary+"\r\n")    //自定义邮件内容分隔符

	//邮件正文
	html := "<p>邮件附件测试</p>"  //邮件正文
	mime.WriteString("Content-Type: text/html; charset=utf-8\r\n\r\n")  //text/html html text/plain 纯文本
	mime.WriteString(html)
	mime.WriteString("\r\n\r\n\r\n")

	//附件
	fileName := "log.txt"
	mime.WriteString("--"+boundary+"\r\n")
	mime.WriteString("Content-Type: application/txt\r\n")   //设置应用类型
	mime.WriteString("Content-Transfer-Encoding: base64\r\n")
	mime.WriteString("Content-Disposition: attachment; filename=\""+fileName+"\"")
	mime.WriteString("\r\n\r\n")

	//将文件转为base64

	//读取并编码文件内容

	//attaData, err := ioutil.ReadFile("../bapi/main.go")
	attaData, err := ioutil.ReadFile(fileName)
	if err != nil {
		fmt.Print(err)
	}



	b := make([]byte, base64.StdEncoding.EncodedLen(len(attaData)))
	base64.StdEncoding.Encode(b, attaData)
	mime.Write(b)
	mime.WriteString("\r\n")
	mime.WriteString("--"+boundary+"--")


	str3 := mime.String()
    auth := smtp.PlainAuth("", USER, PASSWORD, HOST)
    
    // 针对所有的接收者发送邮件
	errs := smtp.SendMail(SERVER_ADDR,auth,USER,[]string{to}, []byte(str3))
	smtp.SendMail(SERVER_ADDR,auth,USER,[]string{cc}, []byte(str3))
	if errs != nil {
		fmt.Println(errs)
	}else{
		fmt.Println("邮件发送成功!")
	}

}

```

# 使用gomail发送邮件
上面发送邮件的方式可行, 但是每次配置这么多选项还是挺复杂的. go中使用体验比较好的发送邮件的包有很多, 这里以gomail为例

```go
m := gomail.NewMessage()
m.SetAddressHeader("From", "********", "sk")  // 发件人
m.SetHeader("To",  // 收件人
    m.FormatAddress("xxxxxxxx", "xx"),
)
m.SetHeader("Cc", "******12@qq.com")
m.SetHeader("Subject", "Gomail")  // 主题
m.SetBody("text/html", "Hello mail</a>")  // 正文
m.Attach("log.txt")

d := gomail.NewPlainDialer(host, 25, user, passwd)  // 发送邮件服务器、端口、发件人账号、发件人密码
if err := d.DialAndSend(m); err != nil {
    panic(err)
}
```

这样来看是否简单很多了呢?