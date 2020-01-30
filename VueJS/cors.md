## Nuxt.js에서 CORS 문제 처리

프론트와 백엔드를 나눠서 개발할 때 웹쪽에서 골치가 아픈 것이 CORS 문제이다.
CORS란 간단히 말하자면, 보안상 다른 도메인에 있는 자원(API)을 사용하지 못하게 하는 것이다.

알아보니 웹서버에서 처리하는 방법과 Nuxt.js에서 axios의 proxy 기능을 이용하는 방법이 있었다.

API서버는 localhost의. 8000 포트에서 서비스 되고 있으며, 웹서버는 80으로 돌린다고 가정한다.


### 웹서버(Nginx)에서 처리

```markdown

# nginx.conf

http {
  ...
    server {
      ...
        location / {
          ...
        }
        location /v1/ {
          proxy_pass http://localhost:8080;
          proxy_redirect http://localhost:8080/ $scheme://$host:8080/;
        }
    }
}

```

### Nuxt.js의 axios의 Proxy 사용

```markdown

# nuxt.config.js

axios: {
  ...
  proxy: true
},
proxy: {
  '/v1/': 'http://localhost:8000/‘
}

```
위와 같이 /v1/test 로 요청하면 실제로는 http://localhost:8080/v1/test 로 요청이 처리되며, 같은 도메인상에서 호출하는 것이기 때문에 문제없이 잘 수행된다.


