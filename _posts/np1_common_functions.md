---
title: '[NP1]网络编程常用函数'
date: 2017-03-07 23:33:54
tags: unp
categories: 网络编程
toc: true
---
POSIX标准的套接字结构
<!--more-->
### 1.套接字地址结构
#### 1.1 IPv4套接字地址结构
```c
#include <netinet/in.h>
struct in_addr {
in_addr_t s_addr;	//32-bit IPv4 address
};
struct sockaddr_in {
uint8_t sin_len;	//length of strcut(16)
sa_family_t sin_family;	//AF_INET
in_port_t sin_port;
struct in_addr sin_addr;

char sin_zero[8]; //unused
}
```
#### 1.2 通用套接字地址结构
```c
#include <sys/socket.h>
struct sockaddr {
uint8_t sa_len;
sa_family_t sa_family;
char sa_data[14];	//protocol-specific address
};
```
#### 1.3 IPv6套接字地址结构
```c
#include <netinet/in.h>
struct in6_addr {
uint8_t s6_addr[16];	//128-bit IPv6 address
};
#define SIN6_LEN
struct sockaddr_in6 {
uint8_t sin6_len;	//length of strcut(28)
sa_family_t sin6_family;	//AF_INET6
in_port_t sin6_port;

uint32_t sin6_flowinfo;
strcut in6_addr sin6_addr;

uint32_t sin6_scope_id;
};
```
#### 1.4 新的通用套接字地址结构
```c
#include <netinet/in.h>
struct sockaddr_storage {
uint8_t ss_len;
sa_family_t family;
};
```
****
### 2. 字节排序函数
```c
#include <netinet/in.h>
//返回网络字节序
uint16_t htons(uint16_t host16bitvalue);
uint32_t htonl(uint32_t host32bitvalue);
//返回主机字节序
uint16_t ntohs(uint16_t net16bitvalue);
uint32_t ntohl(uint32_t net32bitvalue);
```
以上h指host，n指net，s指short，l指long
****
### 3. 字节操纵函数
```c
#include <string.h>
void bzero(void *dest, size_t nbytes);
void copy(const void *src, const void *ptr2, size_t nbytes);
int bcmp(const void *ptr1, const void *ptr2, size_t nbytes);	//相等返回0
```
****
### 4.地址转换函数
#### 4.1 IPv4地址转换
```c
#include <arpa/inet.h>
int inet_aton(const char *strptr, struct in_addr *addrptr);	//字符有效返回1，否则0
//有效32-bit二进制网络字节序IPv4地址，否则为INADDR_NONE
in_addr_t inet_addr(const chat *strptr);
char *inet_ntoa(struct in_addr inaddr);	//返回点分十进制字符串
```
以上a指address，n指net，上述只适用于IPv4地址转换

#### 4.2 通用地址转换
```c
#include <atpa/inet.h>
//有效返回1，无效返回0，出错返回-1
int inet_pton(int family, const char strptr, void *addrptr);
//出错返回NULL
char *inet_ntop(int family, const void *addrptr, char *strptr, size_t len);	
//其中len也可指定为以下值
#include <netinet/in.h>
#define INET_ADDRSTRLEN 16
#define INET6_ADDRSTRLEN 46
```
#### 4.3 I/O操作函数
```c
#include <unistd.h>
//返回:读到的字节数,若已到文件尾为 0,若出错为- 1
ssize_t read(int filedes, void *buff, size_t nbytes);	
//返回:若成功为已写的字节数,若出错为- 1
ssize_t write(int filedes, void *buff, size_t nbytes);
```
