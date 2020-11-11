# Spring Boot에 Let's Encrypt로 만든 인증서 적용

이 방법을 적용하기 위해서는 Spring Boot가 default 서버(Tomcat)으로 적용되어 있어야한다.

아래는 letsencrpyt를 Tomcat서버에 적용하기 위해 인증서를 변환하는 스크립트다.

```bash
#!/bin/bash

CERT_PATH="$1"
PRIVATE_PATH="$2"
CHAIN_PATH="$3"
FULLCHAIN_PATH="$4"

function checkArg()
{
        result=1
        if [ -z "$CERT_PATH" ] || [ -z "$PRIVATE_PATH" ] || [ -z "$CHAIN_PATH" ] || [ -z "$FULLCHAIN_PATH" ]
        then
        echo "Usage : $0 <cert.pem path> <private.pem path> <chain.pem path> <fullchain.path>"
        return $result
        fi
        result=0
        return $result
}

if ! checkArg
then
        exit 1
fi

openssl pkcs12 -export -in $FULLCHAIN_PATH -inkey $PRIVATE_PATH -out keystore.p12 -name tomcat -CAfile $CHAIN_PATH -caname root
```

사용방법은 다음과 같다.

```bash
$ ./convert-to-tomcat-ssl.sh <cert.pem path> <privkey.pem path> <chain.pem path> <fullcain.pem path>Enter Export Password: <password>Verifying - Enter Export Password: <password>
```

변환된 `keystore.p12` 인증서를 적절한 위치에 옮겨놓고 적용할 프로젝트의 properties에 아래의 내용을 추가한다.

```
security.require-ssl=true
server.ssl.key-store=<keystore.p12 path>
server.ssl.key-store-password=<password>
server.ssl.key-store-type=PKCS12
server.ssl.key-alias=tomcat
```
