---
layout:     post
title:      使用SpringMail发送带图片和附件的邮件
subtitle:   使用SpringMail发送带图片和附件的邮件
date:       2017-12-08
author:     ptpan
header-img: img/post-bg-pexels-photo-695299.jpeg
catalog: true
tags:
    - Spring Mail
    - Java
    - Spring
---

>
> 使用SpringMail发送带图片和附件的邮件

#### 发送带附件的邮件
```java
/**
     * 【发送带附件邮件】
     * @param mailContent 邮件内容
     * @param mailAddress 邮件地址
     * @param Mailtitle 邮件标题
     * @param attachments 附件Map
     * @param ID 消息id
     * @return
     */
    public static boolean sendMailByYY(String mailContent, String mailAddress, String Mailtitle,
        Map<String, InputStreamSource> attachments, String ID) {
        boolean result = true;
        try {
            JavaMailSenderImpl senderImpl = new JavaMailSenderImpl();
            // 设定mail server
            senderImpl.setHost(Constants.MAILSERVER);
            senderImpl.setPort(Integer.parseInt(Constants.MAILSERVERPORT));
            senderImpl.setUsername(Constants.MAILSERVICEADD);
            senderImpl.setPassword(Constants.MAILPASSWORD);

            // 建立邮件消息，设置邮件头
            MimeMessage mailMessage = senderImpl.createMimeMessage();
            //mailMessage.setHeader("XXXXXX", "XXXXXX");
            //发送带附件邮件需要设置multipart为true
            MimeMessageHelper messageHelper = new MimeMessageHelper(mailMessage, true, "utf-8");

            // 设置收件人，寄件人
            String sender = Constants.MAILUSERNAME;
            String nick = MimeUtility.encodeText(sender);
            messageHelper.setTo(mailAddress);
            messageHelper.setFrom(new InternetAddress(nick + "<" + Constants.MAILADDR + ">"));
            messageHelper.setSubject(Mailtitle);
            messageHelper.setText(mailContent, true);

            //添加附件
            if (attachments != null && attachments.size() > 0) {
                Iterator<Entry<String, InputStreamSource>> iterator = attachments.entrySet()
                    .iterator();
                while (iterator.hasNext()) {
                    String key;
                    InputStreamSource value;
                    key = iterator.next().getKey();
                    value = attachments.get(key);
                    // MimeUtility.encodeWord(key)解决文件名乱码问题
                    messageHelper.addAttachment(MimeUtility.encodeWord(key), value);
                }
            }

            Properties prop = new Properties();
            // 将这个参数设为true，让服务器进行认证,认证用户名和密码是否正确
            prop.put("mail.smtp.auth", "true");
            prop.setProperty("mail.smtp.connectiontimeout", "60000");
            prop.put("mail.smtp.timeout", "60000");
            senderImpl.setJavaMailProperties(prop);
            // 发送邮件
            senderImpl.send(mailMessage);
            logger.info(ID + "|" + mailAddress + "| send mail OK ");
        }
        catch (Exception e) {
            logger.error(ID + "|" + mailAddress + "| send mail fail ", e);
            result = false;
        }
        return result;
    }
```
<font color="#FF0000">附件有下面三种方式可以添加：</font>
```java
public void addAttachment(String attachmentFilename, InputStreamSource inputStreamSource) throws MessagingException

public void addAttachment(String attachmentFilename, File file) throws MessagingException

public void addAttachment(String attachmentFilename, DataSource dataSource) throws MessagingException
```




####发送带图片的邮件

```java
/**
     * 【发送带图片的邮件】html中需要修改imgSrc,<img src="cid:imgId"/>
     * @param mailContent 邮件HTML内容
     * @param mailAddress 收件地址
     * @param Mailtitle 标题
     * @param ID
     * @param mapImgs 邮件中包含的图片,其中Key是imgId，Value是图片的存储路径
     * @return
     */
    public static boolean sendMailByYY(String mailContent, String mailAddress, String Mailtitle,
        String ID, Map<String, String> mapImgs) {
        boolean result = true;
        try {
            JavaMailSenderImpl senderImpl = new JavaMailSenderImpl();
            // 设定mail server
            senderImpl.setHost(Constants.MAILSERVER);
            senderImpl.setPort(Integer.parseInt(Constants.MAILSERVERPORT));
            senderImpl.setUsername(Constants.MAILSERVICEADD);
            senderImpl.setPassword(Constants.MAILPASSWORD);

            // 建立邮件消息
            MimeMessage mailMessage = senderImpl.createMimeMessage();
            MimeMessageHelper messageHelper = new MimeMessageHelper(mailMessage, true, "utf-8");

            // 设置收件人，寄件人
            String sender = Constants.MAILUSERNAME;
            String nick = javax.mail.internet.MimeUtility.encodeText(sender);
            messageHelper.setTo(mailAddress);
            messageHelper.setFrom(new InternetAddress(nick + "<" + Constants.MAILADDR + ">"));
            messageHelper.setSubject(Mailtitle);
            messageHelper.setText(mailContent, true);

            // 添加图片
            if (mapImgs != null && mapImgs.size() > 0) {
                Iterator<Entry<String, String>> iterator = mapImgs.entrySet().iterator();
                while (iterator.hasNext()) {
                    String key;
                    String value;
                    key = iterator.next().getKey();
                    value = mapImgs.get(key);
                    FileSystemResource img = new FileSystemResource(new File(value));
                    messageHelper.addInline(key, img);
                }
            }
            Properties prop = new Properties();
            // 将这个参数设为true，让服务器进行认证,认证用户名和密码是否正确
            prop.put("mail.smtp.auth", "true");
            prop.setProperty("mail.smtp.connectiontimeout", "60000");
            prop.put("mail.smtp.timeout", "60000");
            senderImpl.setJavaMailProperties(prop);
            // 发送邮件
            senderImpl.send(mailMessage);
            logger.info(ID + "|" + mailAddress + "| send mail OK ");
        }
        catch (Exception e) {
            logger.error(ID + "|" + mailAddress + "| send mail fail ", e);
            result = false;
        }
        return result;
    }
```

<font color="#FF0000">发送带图片的邮件需要修改img标签的src属性：</font>
```
<img src="cid:imgId"/>
```

