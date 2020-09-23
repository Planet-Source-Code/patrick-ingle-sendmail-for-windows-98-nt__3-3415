<div align="center">

## Sendmail for Windows 98/NT


</div>

### Description

A simple sendmail utility for Windows 98/NT at the TCPIP socket level, based on RFC821. A needed addon for Windows. Operated in command-line/console mode.
 
### More Info
 
smtp mail server ip address

// sender email

// receiver email

// subject line (around in quotes)

// message text (around in quotes)

No error checking is performed on the response from the SMTP server, available in the rec buffer. Look for recv() function for meesages returned from the Server.

// The mail server ip address must be specified

// as name resolution is not included.

// Check the code comments.

// Nothing if successful,

// otherwise show socket errors

// None known


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Patrick Ingle](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/patrick-ingle.md)
**Level**          |Intermediate
**User Rating**    |3.8 (15 globes from 4 users)
**Compatibility**  |C\+\+ \(general\), Microsoft Visual C\+\+
**Category**       |[Complete Applications](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/complete-applications__3-7.md)
**World**          |[C / C\+\+](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/c-c.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/patrick-ingle-sendmail-for-windows-98-nt__3-3415/archive/master.zip)





### Source Code

```
/*
 * FILENAME:  sendmail.cpp
 *
 * DESCRIPTION:  A simple text-based implementation for the popular sendmail tool.
 *     Useful for Windows, since sendmail is not included
 *
 * BUILD DEPENDENCIES: Visual C++ 6.0 - must include ws2_32.lib during linking
 *      Visualage C++ 3.5.9 - must include wsock32.lib during linking
 *
 * Change History:
 * Date    Description
 * -------------------------------------------------------------------------------------
 * 03/04/2002  File created
 *
 *
 * TODO:
 * - mail server name resolution
 * - attachment implementation
 * - reading message text from a file
 * - reading subject line from a file with truncation
 * - integration with PHP/mail function
 *
 */
#ifdef WIN32
#include <windows.h>
#include <winsock.h>
#define WINSOCK_VERSION 512
#endif // WIN32
#include <stdio.h>
#include <string.h>
#define CR '\r' // carriage return
#define LF '\n' // line feed
#define SMTP_PORT  25  // IP Port for the SMTP Server
#define MAXBUF   80  // Maximum buffer size
void main(int argc,char **argv)
{
 char *remote_host;  // must use ip address
 char *sender;
 char *receiver;
 char *subject;
 char *message;
 //
 // Checks the command line arguments
 //
 if (argc == 6) {
  remote_host = argv[1];
  sender = argv[2];
  receiver = argv[3];
  subject = argv[4];
  message = argv[5];
 } else {
  printf("usage: <host ip> <sender> <receiver> <subject> <message>\n");
  return;
 }
#ifdef WIN32
 //
 // Initializes the Winsock stack, required for Windows
 //
 WORD wVersionRequested;
 WSADATA wsaData;
 int err;
 wVersionRequested = MAKEWORD( 2, 2 );
 err = WSAStartup( wVersionRequested, &wsaData );
 if ( err != 0 ) {
  /* Tell the user that we could not find a usable */
  /* WinSock DLL.         */
  return;
 }
#endif // WIN32
 PROTOENT *proto;
 SOCKET s;
 SOCKADDR_IN addr;
 SOCKADDR_IN addrServer;
 char buf[MAXBUF];
 char req[MAXBUF];
 //
 // Creates and opens a socket with the SMTP Mail Server
 //
 proto = getprotobyname("tcp");
 s = socket(PF_INET,SOCK_STREAM,proto->p_proto);
 if (s != INVALID_SOCKET) {
  addr.sin_family = PF_INET;
  addr.sin_port = 0;
  addr.sin_addr.s_addr = htonl(INADDR_ANY);
  if (bind (s, (LPSOCKADDR)&addr,sizeof(addr)) == SOCKET_ERROR) {
   printf("bind() generated error %d\n",WSAGetLastError());
  } else {
   addrServer.sin_family = PF_INET;
   addrServer.sin_port = htons(SMTP_PORT);
   addrServer.sin_addr.s_addr = inet_addr(remote_host);
   if (connect(s,(LPSOCKADDR)&addrServer,sizeof(addrServer)) == SOCKET_ERROR) {
    printf("connect() generated error %d\n",WSAGetLastError());
   } else {
    //
    // Tells the SMTP Server that a mail operation is about to begin
    //
    sprintf(req,"HELO %s\r\n",remote_host);
    if (send(s,req,strlen(req),0) == SOCKET_ERROR) {
     printf("send() generated error %d at line %d\n",WSAGetLastError(),__LINE__);
    } else {
     strncpy(buf,"\0",80);
     if (recv(s,buf,sizeof(buf),0) == SOCKET_ERROR) {
      printf("recv() generated error %d at line %d\n",WSAGetLastError(),__LINE__);
     } else {
#ifdef DEBUG
      printf("recv() [%s] at %d\n",buf,__LINE__);
#endif // DEBUG
     }
    }
    //
    // Tells the SMTP Server who is sending this message
    //
    sprintf(req,"MAIL FROM:<%s>\r\n",sender);
    if (send(s,req,strlen(req),0) == SOCKET_ERROR) {
     printf("send() generated error %d at line %d\n",WSAGetLastError(),__LINE__);
    } else {
     strncpy(buf,"\0",80);
     if (recv(s,buf,sizeof(buf),0) == SOCKET_ERROR) {
      printf("recv() generated error %d at line %d\n",WSAGetLastError(),__LINE__);
     } else {
#ifdef DEBUG
      printf("recv() [%s] at %d\n",buf,__LINE__);
#endif // DEBUG
     }
    }
    //
    // Tells the SMTP Server who will receive this message
    //
    sprintf(req,"RCPT TO: <%s>\r\n",receiver);
    if (send(s,req,strlen(req),0) == SOCKET_ERROR) {
     printf("send() generated error %d at line %d\n",WSAGetLastError(),__LINE__);
    } else {
     strncpy(buf,"\0",80);
     if (recv(s,buf,sizeof(buf),0) == SOCKET_ERROR) {
      printf("recv() generated error %d at line %d\n",WSAGetLastError(),__LINE__);
     } else {
#ifdef DEBUG
      printf("recv() [%s] at %d\n",buf,__LINE__);
#endif // DEBUG
     }
    }
    //
    // Defines the start of the message by adding the subject heading followed by
    // a CR. The subject is required if your want to see a subject in your
    // mail client.
    //
    sprintf(req,"DATA\r\nSubject: %s\r",subject);
    if (send(s,req,strlen(req),0) == SOCKET_ERROR) {
     printf("send() generated error %d at line %d\n",WSAGetLastError(),__LINE__);
    } else {
     if (recv(s,buf,sizeof(buf),0) == SOCKET_ERROR) {
      printf("recv() generated error %d at line %d\n",WSAGetLastError(),__LINE__);
     } else {
#ifdef DEBUG
      printf("recv() [%s] at %d\n",buf,__LINE__);
#endif // DEBUG
     }
    }
    //
    // Adds the receiver to the message header, required if you want
    // to the see the receiver's name in your mail reader
    //
    sprintf(req,"To: <%s>",receiver);
    if (send(s,req,strlen(req),0) == SOCKET_ERROR) {
     printf("send() generated error %d at line %d\n",WSAGetLastError(),__LINE__);
    }
    //
    // Sends the actual text meesage to the SMTP Server
    //
    sprintf(req,"\r\n%s\r\n.\r\n",message);
    if (send(s,req,strlen(req),0) == SOCKET_ERROR) {
     printf("send() generated error %d at line %d\n",WSAGetLastError(),__LINE__);
    }
    //
    // Closes the SMTP communication and tells the SMTP-Server to process mail
    //
    sprintf(req,"QUIT\r\n");
    if (send(s,req,strlen(req),0) == SOCKET_ERROR) {
     printf("send() generated error %d at line %d\n",WSAGetLastError(),__LINE__);
    } else {
     strncpy(buf,"\0",80);
     if (recv(s,buf,sizeof(buf),0) == SOCKET_ERROR) {
      printf("recv() generated error %d at line %d\n",WSAGetLastError(),__LINE__);
     } else {
#ifdef DEBUG
      printf("recv() [%s] at %d\n",buf,__LINE__);
#endif // DEBUG
     }
    }
   }
  }
  closesocket(s);
 }
 if ( LOBYTE( wsaData.wVersion ) != WINSOCK_VERSION ||
   HIBYTE( wsaData.wVersion ) != WINSOCK_VERSION ) {
  WSACleanup( );
  return;
 }
}
```

