**Create ssl certificate**
----
_This tutorial shows how to create a ssl certificate
[See this link](https://www.thesslstore.com/knowledgebase/ssl-generate/csr-generation-guide-for-nginx-openssl/)


## Tutorial
```
openssl req -new -newkey rsa:2048 -nodes -keyout server.key -out server.csr
```
