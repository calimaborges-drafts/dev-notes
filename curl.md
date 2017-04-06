# curl examples

## Usefull for REST

### GET

```
curl -i -H "Accept: application/json" http://192.168.0.165/persons/person/1  
```

### POST

```
curl -X POST -u admin:admin http://example.com/myconfigs/status -Hcontent-type:application/xml -d @/home/user/file.xml
curl -X POST -u admin:admin http://example.com/myconfigs/status -d "param1=value1&param2=value2"
```

### PUT

```
curl -i -H "Accept: application/json" -X PUT -d "phone=1-800-999-9999" http://192.168.0.165/persons/person/1  
```

```
curl -i -H "Accept: application/json" -H "X-HTTP-Method-Override: PUT" -X POST -d "phone=1-800-999-9999" http://192.168.0.165/persons/person/1  
```

### DELETE

```
curl -i -H "Accept: application/json" -X DELETE http://192.168.0.165/persons/person/1  
```

```
curl -i -H "Accept: application/json" -H "X-HTTP-Method-Override: DELETE" -X POST http://192.168.0.3:8090/persons/person/1 
```

### Setting query parameters

```
curl -i -H "Accept: application/json" "http://192.168.0.165/persons?firstName=james&lastName=wallis"
```

### Setting cookies

```
curl -b "name=value" http://example.com
```

```
curl -b mycookies.txt http://example.com
```

### Setting HTTP Headers

```
curl -H "Accept: application/xml" -H "Content-Type: application/xml" http://example.com
```

### Setting form parameters

```
curl -i -H "Accept: application/json" -X POST -d "firstName=james" http://192.168.0.165/persons/person 
```

### Setting body

```
curl -H "Content-Type: application/json" -X POST -d '{"username":"xyz","password":"xyz"}' http://localhost:3000/api/login
```

## General

### Otuput to Screen

```
curl example.com
```

### Output to File

```
curl example.com > file.html
```

```
curl -o file.html example.com 
```

### Download multiple files

```
curl -O libiconv-1.14.tar.gz http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.10.tar.gz \
     -O http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.12.tar.gz \
     -O http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.13.tar.gz
```

### Follow redirect

```
curl -L example.com
```

### Resume download

#### Start download

```
curl -O http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.10.tar.gz
# cancel it with Ctrl + C
```

#### Resume Download

```
curl -C - -O http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.10.tar.gz
```

### View complete request and response headers

```
curl -v example.com
```

### View only response headers

```
curl -I example.com
```


### Use proxy

```
curl -x http://proxyserver:proxyport --proxy-user user:password -L http://example.com
```

### Ignore SSL certificate

```
curl -k https://example.com
```

### Set user agent

```
curl -A "USER AGENT" http://example.com
```

### Limit download rate

```
curl --limit-rate 100k -O http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.10.tar.gz
```

### Download from FTP

```
curl ftp://example.com/mydirectory/myfile.zip --user username:password -o myfile.zip
```

### List FTP directory structure

```
curl ftp://example.com --user username:password
```

### Upload file to FTP

```
curl -T myfile.zip ftp://example.com/mydirectory/ --user username:password
```

### Delete file on FTP

```
curl ftp://example.com/ -X 'DELE myfile.zip' --user username:password
```

### Send email

```
curl --url "smtps://smtp.example.com:465" --ssl-reqd   \
     --mail-from "user@example.com" --mail-rcpt "friend@example.com"   \
     --upload-file mailcontent.txt --user "user@example.com:password" \
     --insecure
```

### Download certificate

```
curl --cacert my-ca.crt https://example.com
```

### Baixar arquivo dependendo da data de alteração

```
curl -z 3-Jan-14 http://example.com/myfile.gz
```


