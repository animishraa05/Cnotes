
# 12. Hypertext Transfer Protocol (HTTP)

## Introduction to HTTP

The Hypertext Transfer Protocol (HTTP) is an application-level protocol for distributed, collaborative, hypermedia information systems. HTTP is the foundation of data communication for the World Wide Web. It is a request-response protocol, which means that the client sends a request to the server, and the server sends a response back to the client.

HTTP is a stateless protocol, which means that the server does not keep any information about the client between requests. This makes HTTP a very scalable protocol, but it also means that the client has to send all the information that the server needs with each request.

## HTTP Messages

There are two types of HTTP messages: requests and responses.

### HTTP Requests

An HTTP request is a message that is sent from a client to a server. An HTTP request consists of a request line, a set of headers, and an optional message body.

The request line consists of a method, a URI, and an HTTP version. The method is the action that the client wants the server to perform. The URI is the resource that the client wants to access. The HTTP version is the version of the HTTP protocol that the client is using.

The headers are a set of key-value pairs that provide additional information about the request. The message body is the data that the client is sending to the server.

### HTTP Responses

An HTTP response is a message that is sent from a server to a client. An HTTP response consists of a status line, a set of headers, and an optional message body.

The status line consists of an HTTP version, a status code, and a status message. The status code is a three-digit number that indicates the status of the request. The status message is a short description of the status code.

The headers are a set of key-value pairs that provide additional information about the response. The message body is the data that the server is sending to the client.

## HTTP Methods

There are many different HTTP methods, but the most common methods are:

- **GET:** The GET method is used to retrieve a resource from the server.
- **POST:** The POST method is used to submit data to the server.
- **PUT:** The PUT method is used to update a resource on the server.
- **DELETE:** The DELETE method is used to delete a resource from the server.

## HTTP Status Codes

HTTP status codes are three-digit numbers that indicate the status of a request. The most common status codes are:

- **200 OK:** The request was successful.
- **301 Moved Permanently:** The resource has been moved to a new location.
- **400 Bad Request:** The request was invalid.
- **401 Unauthorized:** The client is not authorized to access the resource.
- **404 Not Found:** The resource could not be found.
- **500 Internal Server Error:** The server encountered an error while processing the request.

## HTTPS

HTTPS is a secure version of HTTP. It encrypts the data that is sent between the client and the server. This makes it much more difficult for an attacker to intercept and read the data. HTTPS is used for secure communication, such as online banking and e-commerce.
