# Advanced API Security

## Chapter 5 | API 게이트웨이를 이용한 에지 보안



### Zuul API 게이트웨이에서 TLS 활성화



- 키생성 ( alias = spring, end-point = zool,  keystore = keystore.jks, 키 패스워드 = springboot, 스토어 패스워드 = springboot )



```bash
keytool -genkey -alias spring \
-keyalg RSA \
-keysize 4096 \
-validity 3650 \
-dname "CN=zool,OU=bar,O=zee,L=sjc,S=ca,C=us" \
-keypass springboot \
-keystore keystore.jks \
-storeType jks \
-storepass springboot
```



### cURL 명령을 사용해 HTTPS 사용중인 Zuul 게이트웨이에 접근



```bash
curl -k https://localhost:9090/retail/order/11
```



- HTTPS 종단점을 보호하려고 자체 서명 ( 신뢰할 수 없는 ) 인증서가 있으므로 신뢰 유효성을 검증하지 않도록 -k 파라미터를 전달 해야한다.



### Keytool 명령어를 사용해 Zuul 게이트퀘이의 공개인증서 를 PEM형식의 ca.crt파일로 내보낼 수 있다.



```bash
keytool -export -file ca.crt \
-alias spring \
-rfc \
-keystore keystore.jks \
-storepass springboot
```



```bash
curl -x "http://70.10.15.10:8080" -k https://zool:9090/retail/order/11 --resolve zool:9090:127.0.0.1
```



- 운영에서 -k 옵션을 쓸 수 없으므로, 위에서 생성한 ca.crt 공개인증서 파일을 이용하여 TLS 통신
- zool 호스트 이름에 대한 DNS 항목이 없으므로 cURL에서 -resolve zool:9090:127.0.0.1 추가



#### 시스템 프록시를 사용한다면 ( e.g. 사내 프록시 )




```bash
curl -x "http://70.10.15.10:8080" --cacert ca.crt https://zool:9090/retail/order/11 --resolve zool:9090:127.0.0.1
```



- https 프록시를 사용한다면 하기와 같은 에러출력



```bash
curl: (35) error:1408F10B:SSL routines:ssl3_get_record:wrong version number
```



#### OAuth 2.0 보안 토큰 테스트



```bash
curl -v -X POST \
--basic -u 10101010:11110000 \
-H "Content-Type:application/x-www-form-urlencoded;charset=UTF-8" \
-k \
-d "grant_type=client_credentials&scope=foo" \
https://localhost:8443/oauth/token
```



-v, --verbose 더 상세하게 출력하도록 함

-X, --request Request 시 사용할 method 종류

-H, --header HTTP Header에 추가

-k, --insecure SSL로 통신할때, 비보안통신 허가

-d,  --data HTTP POST의 데이터



#### Result



```json
{
   "access_token":"716f6719-b5ee-4ff5-a7d5-79d4b8c94d2c",
   "token_type":"bearer",
   "expires_in":5999,
   "scope":"foo"
}
```



