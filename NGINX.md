## NGINX



#### Serving Static Content

```
http {
    server {
        location / {
            root /data/www;
        }
        
        location /images/ {
            root /data;
        }
    }
}
```



- http 요청이 / 로 시작한다면 /data/www로 연결

- http 요청이 /images로 시작한다면 /data + /images -> /data/images로 연결

/images/1.png로 들어오면 두 조건 모두에 부합하지만 긴쪽(longest prefix)에 매칭

----------

#### Setting up a Simple (Reverse) Proxy Sever

```
http {
    server {
        listen 8080;
        root /data/up1;

        location / {
        }
    }
}

```

8080포트로 listen하는 방법이다.

```
http {
    server {
        location / {
            proxy_pass http://localhost:8080;
        }

        location ~ \.(gif|jpg|png)$ {
            root /data/images;
        }
    }
}
```

모든 http요청은 8080서버로 proxy해주고 

.gif .jpg .png로 끝나는 파일은 모두 /data/images로 mapping해준다.



- forward proxy는 우리가 흔히 proxy라 부르는 외부망에 요청을 보낸 후  response를 사용자에게 전달(forward)만 해주는 걸 말하는것같다.

