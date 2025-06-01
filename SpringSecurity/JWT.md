# JWT 가 만들어지는 원리
- JWT는 header.payload.signature로 구성됨
- header와 payload는 base64로 인코딩 
- header와 payload를 합친 후 암호화를통해 변조를하여 signature를 만들어준다
- signature와 headr.payload 부분을 통해 변조를 확인함

> base 64는 이진 데이터를 텍스트로 안전하게 인코딩하기 위한 방식
> 이진 데이터를 6비트 단위로 쪼개서 각각을 `A~Z`, `a~z`, `0~9`, `+`, `/`의 총 64개의 문자 중 하나로 변환한다.


# JWT 구성과 용어
## header
- 메시지에 대한 메타데이터가 저장된 곳, 주로 암호화 알고리즘 등의 정보를 name/value 쌍으로 저장

## payload
- 바이트 배열로 표현될 수 있는 모든 것을 담을 수 있다. 텍스트, 이미지, 문서등 payload가 json 형태로 사용될 경우 이를 claim이라고 한다.

## signature
- header와 payload가 변조되지 않았는지를 체크하는 영역


# JJWT 라이브러리

## 표준 Claim
payload 부분이 json으로 구성되었을 때 표준
- iss : 토큰 발급자
- **sub** : 토큰의 주체(Subject), 일반적으로 사용자 ID
- and : 토큰의 대상(Audience), 토큰이 사용될 시스템
- **exp** : 토큰의 말료 시간(Expiration Time), UNIX 타임 스탬프
- **iat** :  토큰이 발급죈 시간(Issued At), UNIX 타임 스탬프

## JJWT
자바진영에서 JWT 를 다룰 때 가장 많이 사용 되는 라이브러리
`JJWT`는 **Java에서 JWT(Json Web Token)를 쉽게 생성하고 파싱할 수 있도록 도와주는 라이브러리이다.  
JWT를 직접 문자열로 만들고 파싱하는 건 번거롭고 실수하기 쉬운데, JJWT는 이를 매우 간단하게 만들어줌.

### JWT 생성 예제

```java

import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import io.jsonwebtoken.security.Keys;

import java.security.Key;
import java.util.Date;

public class JwtGenerator {
    private static final Key key = Keys.secretKeyFor(SignatureAlgorithm.HS256); // 임시 키

    public static String generateToken() {
        return Jwts.builder()
                .setSubject("chaewon")               // sub 클레임
                .claim("role", "admin")              // 사용자 정의 클레임
                .setIssuedAt(new Date())             // iat
                .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 60)) // 1시간
                .signWith(key)
                .compact();
    }
}

```


### JWT 검증 및 파싱 예제

```java
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.Claims;

public class JwtParser {
    public static void parseToken(String token, Key key) {
        Claims claims = Jwts.parserBuilder()
                .setSigningKey(key)
                .build()
                .parseClaimsJws(token)
                .getBody();

        System.out.println("Subject: " + claims.getSubject());
        System.out.println("Role: " + claims.get("role"));
    }
}
```