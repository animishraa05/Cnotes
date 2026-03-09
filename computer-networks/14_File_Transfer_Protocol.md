
# 14. File Transfer Protocol (FTP)

## Introduction to FTP

The File Transfer Protocol (FTP) is a standard network protocol used for the transfer of computer files between a client and server on a computer network. FTP is built on a client-server model architecture and uses separate control and data connections between the client and the server.

FTP is a stateful protocol, which means that the server maintains state about the client session. This is in contrast to HTTP, which is a stateless protocol.

## How FTP Works

FTP uses two TCP connections to transfer files: a control connection and a data connection.

- **Control Connection:** The control connection is used to send commands and receive responses. The control connection is established on port 21.
- **Data Connection:** The data connection is used to transfer the actual files. The data connection is established on a different port, which is negotiated between the client and the server.

There are two modes for establishing the data connection: active mode and passive mode.

- **Active Mode:** In active mode, the client opens a random port and sends the port number to the server. The server then connects to the client on that port.
- **Passive Mode:** In passive mode, the server opens a random port and sends the port number to the client. The client then connects to the server on that port.

## FTP Commands

There are many different FTP commands, but the most common commands are:

- **USER:** The USER command is used to specify the username.
- **PASS:** The PASS command is used to specify the password.
- **LIST:** The LIST command is used to list the files in the current directory.
- **RETR:** The RETR command is used to retrieve a file from the server.
- **STOR:** The STOR command is used to store a file on the server.
- **QUIT:** The QUIT command is used to close the connection.

## FTP Responses

FTP responses are three-digit numbers that indicate the status of a command. The most common responses are:

- **125:** Data connection already open; transfer starting.
- **150:** File status okay; about to open data connection.
- **200:** Command okay.
- **220:** Service ready for new user.
- **221:** Service closing control connection.
- **226:** Closing data connection.
- **230:** User logged in, proceed.
- **331:** User name okay, need password.
- **425:** Can't open data connection.
- **452:** Error writing file.
- **500:** Syntax error, command unrecognized.
- **530:** Not logged in.
- **550:** Requested action not taken. File unavailable.

## SFTP and FTPS

SFTP and FTPS are two secure versions of FTP.

- **SFTP (SSH File Transfer Protocol):** SFTP is a completely different protocol from FTP. It is based on the SSH protocol and encrypts both the control and data connections.
- **FTPS (FTP over SSL/TLS):** FTPS is an extension of FTP that adds support for the Transport Layer Security (TLS) and the Secure Sockets Layer (SSL) cryptographic protocols. It encrypts both the control and data connections.
