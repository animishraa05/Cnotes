
# 13. Simple Mail Transfer Protocol (SMTP)

## Introduction to SMTP

The Simple Mail Transfer Protocol (SMTP) is an application-level protocol that is used to send and receive email. It is a connection-oriented protocol, which means that it establishes a connection before transferring the data. SMTP is a text-based protocol, which means that the commands and responses are in plain text.

## How SMTP Works

When you send an email, your email client sends the email to your email server. Your email server then uses SMTP to send the email to the recipient's email server. The recipient's email server then uses SMTP to deliver the email to the recipient's inbox.

The process of sending an email with SMTP is as follows:

1. The client establishes a TCP connection to the server on port 25.
2. The client sends a HELO or EHLO command to the server.
3. The server responds with a 250 OK message.
4. The client sends a MAIL FROM command to the server.
5. The server responds with a 250 OK message.
6. The client sends a RCPT TO command to the server.
7. The server responds with a 250 OK message.
8. The client sends a DATA command to the server.
9. The server responds with a 354 Start mail input message.
10. The client sends the email message to the server.
11. The client sends a QUIT command to the server.
12. The server responds with a 221 Bye message.

## SMTP Commands

There are many different SMTP commands, but the most common commands are:

- **HELO:** The HELO command is used to identify the client to the server.
- **EHLO:** The EHLO command is an extended version of the HELO command.
- **MAIL FROM:** The MAIL FROM command is used to specify the sender of the email.
- **RCPT TO:** The RCPT TO command is used to specify the recipient of the email.
- **DATA:** The DATA command is used to send the email message.
- **QUIT:** The QUIT command is used to close the connection.

## SMTP Responses

SMTP responses are three-digit numbers that indicate the status of a command. The most common responses are:

- **220:** The server is ready.
- **221:** The server is closing the connection.
- **250:** The command was successful.
- **354:** The server is ready to receive the email message.
- **421:** The server is not available.
- **500:** The command was not recognized.
- **550:** The mailbox is not available.

## POP3 and IMAP

POP3 and IMAP are two protocols that are used to retrieve email from a server. POP3 is a simple protocol that downloads the email to your computer. IMAP is a more advanced protocol that allows you to manage your email on the server.
