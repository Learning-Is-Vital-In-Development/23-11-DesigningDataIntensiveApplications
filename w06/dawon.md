## 데이터 베이스를 통한 데이터 플로

일반적으로 동시에 다양한 프로세스가 데이터베이스를 접근하는 일은 흔하다. 이런 프로세스는 다양한 애플리케이션이나 서비스일 수 있다. 어느 방법이든 애플리케이션이 변경되는 환경에서 데이터 베이스에 접근하는 경우 아마도 일부 프로세스는 새로운 코드로 수행 중잉고 일부 다른 프로세스는 예전 코드로 수행 중일 것이다. 예를 들어 순회식 업그레이드로 현재 새로운 버전을 배포하는 도중이라면 일부 인스턴스는 아직 갱신되지 않았지만 일부 인스턴스는 이미 갱신됐다. 

- 데이터를 부호화하고 데이터베이스에서 읽는 프로세스는 데이터를 복호화함
→ 애플리케이션의 변환 과정에서 알지못하는 필드가 유실 될 수 있다.
- 데이터를 새로운 스키마로 다시 기록(rewriting 마이그레이션)하는 작업은 분명 가능하다. 하지만
대용량 데이터를 마이그레이션 하는 작업은 비싼 비용의 작업이기 때문에 대부분의 관계형 데이터베이스는 기존 데이터를 다시 기록하지 않고 null을 기본값으로 갖는 새로운 칼럼을 추가함
- 백업 목적이나 데이터 웨어하우스 저장 목적의 데이터 덤프는 복사본을 일관되게 부호화 하는 편이 나음
→ 데이터 덤프는 한번에 기록하고 변하지 않으므로 아브로 객체 컨테이너 파일과 같은 형식이 적합하다. 또한 이것은 파케이와 같은 분석 친화적인 칼럼 지향 형식으로 데이터를 부호화할 좋은 기회이다.

## 서비스 호출을 통한 데이터 플로 : REST와 RPC

네트워크를 통해 통신해야 하는 프로세스가 있을 때 해당 통신을 배치하는 가장 일반적인 방법은 `클라이언트`와 `서버` 두 역할로 배치하는 것

**클라이언트와 서버** 

- 클라이언트 : API로 요청을 만들어 서버에 연결할 수 있다.
- 서버 : 네트워크를 통해 API를 공개한다.
    - **서비스 :** 서버가 공개한 API

즉 **요청**을 보내는 애플리케이션을 **클라이언트**, **응답**을 보내는 애플리케이션을 **서버**라 할 수 있다.

### **웹 동작 방식**

- 클라이언트(웹 브라우저, 모바일 디바이스, 데스크톱 등등 )는 웹 서버로 요청을 보냄 
→ HTML, CSS, 자바스크립트, 이미지 등을 다운로드하기 위해 GET요청을 보냄
- API는 표준화된 프로토콜과 데이터 타입(HTTP, URL, SSL/TLS, HTML 등)으로 구성된다.
- 웹 브라우저 내에서 수행되는 클라이언트 측 자바스크립트 애플리케이션은 XMLHttpRequest를 사용해 HTTP 클라이언트가 될 수 있다. (이 기술을 에이젝스(Ajax)라 칭함.)
    
    > Javascript를 통해 Ajax를 사용하는데 Javascript는 원래 클라이언트 쪽 그러니까 웹 브라우저에서 출발한 언어여서 서버와 교신하는 기능은 없었지만 Ajax를 통해서 가능하게 되었다.
    > 
    
    > *`에이젝스(Ajax)`
    > 
    > 
    > http 프로토콜은 기본적으로 클라이언트 쪽에서 request를 보내고 server의 response를 받으면 이어졌던 연결이 끊기게 설계가 되어 있습니다. 그래서 화면의 내용을 갱신하기 위해서는 다시 request를 하고 response로 페이지 전체를 다시 받게 됩니다. 하지만 페이지 내용을 일부만을 업데이트 하고자 하는데 페이지 전체를 다시 로드하게 된다면 자원 낭비겠죠?
    > 
    > Ajax는 html페이지 전체가 아니라 필요한 부분만을 갱신할 수 있도록 XMLHttpRequset 객체를 통해서 요청합니다. Json이나 xml 형태로 최소한의 필요한 데이터만 받아서 갱신하게 됨으로 자원낭비가 그만큼 줄어들기에 더 나은 서비스를 구현할 수 있습니다.
    > 
    > 이경우 서버 응답은 보통 사람이 볼 수 있게 표시하는 HTML보다는 클라이언트 측 애플리케이션 코드가 이후 처리를 편리하게 할 수 있게 부호화한 JSON, XML과 같은 형태의 파일로 이루어 있다.
    > 
- 서버 자체가 다른 서비스의 클라이언트가 될 수 있다. 이런 접근 방식은 다른 서비스의 일부 기능이나 데이터가 필요하다면 해당 서비스에 요청을 보낸다.
    - 이러한 애플리케이션 개발 방식을 아래와 같이 칭한다.
    **전통적** : `서비스 지향 설계` (service-oriented architecture, SOA)
    **개선** : `마이크로서비스 설계`(microservices architecture, MSA)
    - 위 아키텍쳐의 핵심 설계 목표는 서비스를 배포와 변경에 독립적으로 만들어 애플리케이션의 변경과 유지보수를 더욱 쉽게 만드는 것
        - 예전 버전과 새로운 버전의 서버와 클라이언트가 동시에 실행되기를 기대한다
        - 따라서 서버와 클라이언트가 사용하는 데이터 부호화는 서비스 API 버전간 호환이 가능해야한다.

### **웹 서비스**

웹서비스 예시) 

- 서비스와 통신하기 위한 기본 프로토콜로 HTTP 사용할 때 **(라고 정의할 수는 있지만 이는 너무 함축적인 표현임)**
- 사용자 디바이스에서 실행하며 HTTP를 통해 서비스에 요청하는 클라이언트 애플리케이션. 보통 공공 인터넷 통해 전달된다.
→ 모바일 디바이스에서의 기본 앱, 에이젝스를 사용하는 자바스크립트 웹 앱
- 서비스 지향/마이크로 서비스 아키텍처의 일부로서 대개 같은 데이터 센터에 위치한 같은 조직의 다른 서비스에 요청하는 서비스.(이런 종류의 사용 사례를 지원하는 소프트웨어를 **미들웨어**라 칭함.)
- 보통 인터넷을 통해 다른 조직의 서비스에 요청하는 서비스 이것은 다른 조직의 백엔드 시스템 간 데이터 교환을 위해 사용한다. 이 범주에 신용카드 처리 스스템 같은 온라인 서비스가 제공하는 공개 API나 사용자 데이터 공유 접근을 위한 OAuth가 포함된다.

### **웹서비스 방법 2가지 REST & SOAP**

REST와 SOAP는 각기 다른 두 가지의 온라인 데이터 전송 방식입니다. 둘 다 웹 애플리케이션 간 데이터 통신을 허용하는 [애플리케이션 프로그래밍 인터페이스(Application Programming Interface, API)](https://www.redhat.com/ko/topics/api)를 구축하는 방법을 정의한다. 

<aside>
💡 `API란?`

![api란 두 개 이상의 컴퓨터 프로그램이 서로 통신하는 방법](https://prod-files-secure.s3.us-west-2.amazonaws.com/fe5d7f0e-3b33-4513-9b73-373a7fef9157/897eb331-455d-475a-b6ed-c041b10f9595/Untitled.png)

api란 두 개 이상의 컴퓨터 프로그램이 서로 통신하는 방법

**API는 “**Application Programming Interface”의 준말. 풀이를 하자면, 여러 **프로그램들과 데이터베이스, 그리고 기능들의 상호 통신 방법을 규정하고 도와주는 매개체이다.** API는 데이터베이스가 아니지만, 액세스 권한이 있는 앱의 권한 규정과 “서비스 요청”에 따라 데이터나 서비스 기능을 제공하는 메신저 역할을 한다.

`API가 필요한 이유?`

API의 필요성은 Web의 진화와 밀접한 연관이 있으니 잠깐 살펴보면, 모놀리틱 아키텍처(Monolithic Architecture)가 주도적이었던 Web 1.0 시대에서는 (하지만 현재에도 사실 많이 쓰이고 있는 것이 사실!) 서버와 클라이언트가 분리되지 않고 모두 서버에서 동시에 처리하기 때문에 API 필요성이 그다지 절실하지 않았다.

그러나 2000년경부터 시작된 Web2.0의 “개방, 참여, 공유”의 정신을 바탕으로 정보가 쌍방향으로 소통하고 “사용자가 생성한 데이터”를 위주로 웹 앱의 붐, 그리고 2010년대 들어서 클라우드(Cloud) 기반 인프라와 MSA(Microservices Architecture)의 사용이 확산되면서 API 확산이 가속화되었고 이제 API에서 가장 흔한 구조인 REST 또는 RESTful API가 점차 새로운 웹 생태계의 기반으로 주목된 것이다.

API를 개인적으로는 이렇게 이해한다: “**request-to-serve**”, 한마디로 “각자 권한 분야에서 각자 필요한 것만 연계하기(철저한 개인주의 따로국밥?)”를 가능하게 해주는 서비스.

`API종류`

**1) Private API**

Private API는 내부 API로, 기업이나 연구 단체 등에서 자체 제품과 운영 개선을 위해 단체 내부에서만 사용. 따라서 제삼자에게 노출되지 않는다.

**2) Public API**

Public API는 말 그대로 public, 즉 개방형 API로, 모두에게 공개된다. Public API 중에서도 접속하는 대상에 대한 제약이 없는 경우를 OpenAPI라 한다.
**3) Partner API**

Partner API는 특정 비즈니스 파트너 간의 데이터 공유. 그러므로 동의하는 특정인들만 사용할 수 있다.

</aside>

### `REST` (**Representational State Transfe)**

```
GET /users          : 모든 사용자의 정보를 가져온다.
GET /users/{id}     : id에 해당하는 사용자의 정보를 가져온다.
POST /users         : 새로운 사용자를 추가한다.
PUT /users/{id}     : id에 해당하는 사용자 정보를 수정한다.
DELETE /users/{id}  : id에 해당하는사용자 정보를 삭제한다.
```

- REST는 프로토콜이 아니라 HTTP의 원칙을 토대로 한 설계 철학
- 간단한 데이터 타입을 강조하며 URL을 사용해 리소스를 식별하고 캐시 제어, 인증, 콘텐츠 유형 협상에 HTTP 기능을 사용
- REST 원칙에 따라 설계된 API를 `RESTful` 이라고 부름
- 코드 생성과 자동화된 도구와 관련되지 않은 간단한 접근 방식을 선호한다.
- Swagger(스웨거)로 알려진 오픈 API 같은 정의 형식을 사용해 RESTful API와 제품 문서를 기술하는데 사용할 수 있다.

![스크린샷 2023-09-05 오전 8.38.01.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fe5d7f0e-3b33-4513-9b73-373a7fef9157/f5022353-190d-4069-9bc4-8d12a47977ee/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-09-05_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_8.38.01.png)

- **사용예제**
    
    ```bash
    (1) API의 헤더 정보와 body 값을 정의한다. 보통 다른 업체의 API를 사용할 때는 고유의 token을 발급 받는다. 토큰 정보는 private하기 때문에 실행 환경에 넣어두는 것이 좋다.
    (2) API 통신은 다양한 에러가 발생할 수 있다. 따라서 사전에 try/catch을 사용해 에러를 잡는 것이 좋다. post/get/delete 등의 방식에 따라 HTTP.post()와 같이 사용한다. 첫 번째 인자에는 사용할 API주소, 두 번째 인자에는 option 정보를 넣는다.
    (3) API 통신 결과는 string으로 나오기 때문에 parse를 통해 객체 형태로 바꿔준다.
    
    let opt = {		--- (1)
        headers: {
            "content-type"	: "application/json",
            "token"		: "abcdefg123456"
        },
        data: {
            "name" : "minidoo",
            "blog" : "minidoo.log"
        }
    };
    
    try {			--- (2)
        let url = "https://velog.io/@minidoo/api/information";
        let result = HTTP.post(url, opt);
        let data = JSON.parse(result.content);	--- (3)
        
        return data;
    } catch(err) {
        console.log(err.message);
    }
    ```
    

### `SOAP`(**Simple Object Access Protocol)**

- HTTP, HTTPS, SMTP 등을 사용하여 XML 기반의 메시지를 컴퓨터 네트워크 상에서 교환하는 통신 프로토콜 → 쉽게말해 HTTP, HTTPS등 통신프로토콜 위에서 XML 메시지를 요청응답 받는 것.
- HTTP 상에서 가장 일반적으로 사용되지만 HTTP와 독립적이며 대부분 HTTP 기능을 사용하지 않는다. 그 대신 다양한 기능을 추가한 광범위하고 복잡한 여러 표준(WS-*라 하여 Web Service Framework)을 제공한다.
- 웹서비스 기술 언어(Web Services Description Language) 또는 WSDL(XML 기반 언어)를 사용해 기술한다.
    
    > *`WSDL란?`
    > 
    > 
    > **WSDL(Web Services Description Language)** 는 웹서비스가 기술된 정의 파일의 총칭으로 XML을 사용해 기술됩니다.
    > 
    > **웹서비스의 구체적인 내용**이 기술되어있는 문서이고 서비스 제공장소, 서비스 메세지 포맷, 프로토콜들이 기술되어있습니다.
    > 
    > **특징** 
    > 
    > - **XML기반**으로 플랫폼에 독립적, 서로 다른 운영체제에서 실행되는 서비스간 통신방법을 제공
    > - 프록시와 방화벽에 구애받지 않고, **HTTP, HTTPS**등을 통해 메세지를 교환
    > - **확장**이 용이
    > - **에러 처리**에 대한 내용이 기본적으로 내장
    > - XML 형태로 메세지를 보내기때문에 다른 기술들에 비해 상대적으로 **처리속도가 느림**
    > 
    > 참고 | **[호다닥 공부해보는 SOAP](https://gruuuuu.github.io/programming/soap/)** 
    > 
- 사람이 읽을 수 없도록 설계되어 도구나 IDE에 크게 의존한다
- SOAP 벤더 가 지원하지 않는 프로그래밍 언어 사용자의 경우 SOAP 서비스와의 통합은 어렵다.
- **사용예제**
    
    ```bash
    import soapRequest from 'easy-soap-request';
    
    (1) 필수 Parameters 설정
    let params = `
        <DATA>
          <NAME>minidoo</NAME>
          <VLOG>minidoo.log</VLOG>
        </DATA>
    `;
    
    (1-1) 공백제거
    params = params.replace(/>\s*/g, '>').replace(/\s*</g, '<');
    
    (2) xml 스키마 설정
    let xml = `
        <?xml version='1.0' encoding='utf-8'?>
        <soap:Envelope xmlns:xsi='' ...>
          <soap:Body> ... </soap:Body>
        </soap:Envelope>
    `;
    
    (2-1) 공백제거
    xml = xml.replace(/>\s*/g, '>').replace(/\s*</g, '<');
    
    (3) SOAP API 통신
    let url = "https://velog.io/@minidoo/api/information";
    let soapHeader = {
        "Content-type" : "text/xml";
    };
    
    let opt = { url: url, soapHeader: soapHeader, xml: xml };
    
    (async () => {
        const { response } = await soapRequest(opt); 
        const { body } = response;
        
        console.log(body);
    })();
    ```
    

SOAP는 기업간 통합과 상호 운용성을 이루는데 있어서 가장 단순한 메커니즘이다. 
SOAP는 다른 언어로 다른 플랫폼에서 빌드된 애플리케이션이 통신할 수 있도록 설계된 최초의 표준 프로토콜이기 때문에 복잡성과 오버헤드를 증가시키는 빌트인 룰을 적용하므로, 페이지 로드 시간이 길어질 수 있다.

그러나 이러한 표준은 빌트인 컴플라이언스를 제공한다는 의미이므로, 기업에서 선호하는 방식이기도 합니다. 빌트인 컴플라이언스 표준에는 [보안](https://www.redhat.com/ko/topics/security/api-security)과 안정적인 데이터베이스 트랜잭션의 기본 속성인 원자성, 일관성, 격리성, 내구성(Atomicity, Consistency, Isolation and Durability, ACID)이 포함된다.
 
OAP와 다양한 확장이 표면상으로 표준이 됐지만 다른 벤터의 구현 간 `상호운용성`은 종종 문제를 일으킨다. 이와 같은 이유로 여전히 많은 대기업에서 SOAP를 사용하지만 대부분의 작은 기업에서는 선호하지 않는다. 

### SOAP장단점

**장점**

1.  웹 기반 서비스를 생성하는 HTTP프로토콜을 사용하므로 언어와 플랫폼에 독립적이다.
2. **WS-Security, SSL** 등의 기능을 지원하여 메세지 수준에서 암호화가 가능하고 개인 정보 보호 및 무결성을 제공한다,
3. SOAP은 **엄격한 보안성**을 가지고있기에 , **금융, 통신, 마케팅, 결제 시스템**과 같은 **기업용 서비스에 사용**되는 장점이있다.

**단점**

1. **XML 형식**만 사용할 수 있다.

2. **캐시기능을 사용할 수 없다**.

3. **많은 리소스와 대역폭을 필요**로 하므로 오버헤드가 큰 편이다.

4. SOAP API 서버 구축을 위해서는 매우 제한된 규칙에 대해 전문적이고 깊은 이해가 필요하다.
     & 사용을 위한 **복잡한 표준**이 정해져 있어, 러닝커브가 큰 편이고 **지식의 전문성이 필요하다.**

### ****SOAP REST 차이점****

| 차이점 | SOAP | REST |
| --- | --- | --- |
| 유형 | 프로토콜 | 아키텍처 스타일 |
| 기능 | 기능 위주 : 구조화된 정보 전송 | 데이터 위주: 데이터를 위해서 리소스에 접근 |
| 데이터 포맷 | XML만 사용 | 일반 텍스트, HTML, XML, JSON등 다양한 포맷 허용 |
| 보안 | WS-Security와 SSL 지원 | SSL와 HTTPS 지원 |
| 대역폭 | 상대적으로 더 많은 리소스와 대역폭이 필요 | 상대적으로 리소스가 적게 필요하고 무게가 가벼움 |
| 데이터 캐시 | 캐시를 사용할 수 없음 | 캐시 사용 가능 |
| 페이로드 처리  | 엄격한 통신 규약을 갖고 있으며 모든 메세지는 보내기 전에 알려져야함 | 미리 알릴 필요 없음 |
| ACID 준수 | 자체적인 ACID기준이 있어서 데이터 손상을 줄여줌 | ACID 준수와 관련된 내용이 없음 |
|  |  | 웹 프레임 워크 
- Fast API , Phoenix, Gin,Play,Fastify, Express JS, 장고 REST, Flask, Ruby on Rails, Spring Boot |

![SOAP는 위의 규약과 WSDL등의 규칙이 존재하기 때문에 데이터 요청을 주고 받을 떄도 **SOAP Standard (SOAP Envelope, SOAP Head, SOAP Body 등)를 지켜서 보내야 한다.**](https://prod-files-secure.s3.us-west-2.amazonaws.com/fe5d7f0e-3b33-4513-9b73-373a7fef9157/a40c730c-acb1-40e3-af14-a40b0dd5d6f9/Untitled.png)

SOAP는 위의 규약과 WSDL등의 규칙이 존재하기 때문에 데이터 요청을 주고 받을 떄도 **SOAP Standard (SOAP Envelope, SOAP Head, SOAP Body 등)를 지켜서 보내야 한다.**

참고 | 

 1.  [SOAP REST 차이, 두 방식의 가장 큰 차이점은?](https://blog.wishket.com/soap-api-vs-rest-api-%EB%91%90-%EB%B0%A9%EC%8B%9D%EC%9D%98-%EA%B0%80%EC%9E%A5-%ED%81%B0-%EC%B0%A8%EC%9D%B4%EC%A0%90%EC%9D%80/)

1. **RESTful과 SOAP 비교  [Roots of the REST/SOAP Debate](http://web.archive.org/web/20120421084456/http://www.prescod.net/rest/rest_vs_soap_overview/)** 
2. **현재까지 SOAP를 쓰는 이유  [Why are most of the flight booking providers still using WSDL and SOAP? Why would anyone use them? - Quora](https://qr.ae/pGj0ur)** 

### ****원격 프로시저 호출(RPC) 문제 The problems with remote procedure calls(RPCs)****

웹 서비스는 network 상에서 API를 호출하는 여러 기술중 가장 최신의 형상일 뿐이다.
이러한 웹 서비스는 `원격 프로시저 호출(Remote procedure call, RPC)`의 아이디어를 기반으로 한다. 

> **RPC란? 
원격 프로시저 호출**([영어](https://ko.wikipedia.org/wiki/%EC%98%81%EC%96%B4): remote procedure call, 리모트 프로시저 콜, RPC)은 별도의 원격 제어를 위한 코딩 없이 다른 [주소 공간](https://ko.wikipedia.org/wiki/%EC%A3%BC%EC%86%8C_%EA%B3%B5%EA%B0%84)에서 [함수](https://ko.wikipedia.org/wiki/%ED%95%A8%EC%88%98_(%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D))나 프로시저를 실행할 수 있게하는 프로세스 간 통신 기술이다. 다시 말해, 원격 프로시저 호출을 이용하면 프로그래머는 함수가 실행 프로그램의 위치가 로컬 위치에 있든 원격 위치에 있든 동일한 코드를 이용할 수 있다.
> 
> 
> <aside>
> 💡 **함수 vs 프로시저**
> 
> - **함수(Function)** : **Input에 따른 Output의 발생**을 목적으로 한다. 따라서 **Return값을 필수**로 가져야 하며, **Client단**에서 처리되기 때문에 주로 **간단한 계산 및 수치 등을 도출**할 때 사용한다.
> - **프로시저(Procedure)** : Output값 자체에 집중하기보단, '**명령 단위가 수행하는 절차**'에 집중한 개념이라고 보면 된다.
> 
> 따라서 Return 값이 있을수도 있고, 없을수도 있으며, **Server단**에서 처리되기 때문에 **함수보다 큰 단위의 실행, 프로세싱** 등을 할 때 사용한다.
> 
> </aside>
> 
> [객체 지향](https://ko.wikipedia.org/wiki/%EA%B0%9D%EC%B2%B4_%EC%A7%80%ED%96%A5)의 원칙을 사용하는 소프트웨어의 경우 원격 프로시저 호출을 **원격 호출**(remote invocation) 또는 **원격 메소드 호출**(remote method invocation)이라고 일컫는다.
> 
> RPC모델은 원격 네트워크 서비스 요청을 같은 프로세스 안에서 특정 프로그래밍 언어의 함수나 메서드를 호출하는 것과 동일하게 사용 가능하게 해준다. 이런 추상화를 `위치 투명성(location transparency)`라 한다.
> 
> **RPC 사용 예시** 
> **Command API에 사용된다.** RPC는 원격 시스템에 명령을 보내기에 매우 적절한 선택이다. 
> 예를 들어, 채널 가입, 채널 탈퇴, 메시지 전송 기능을 제공하는 Slack API는 매우 명령 중심적이다. 그래서 Slack API 설계자들은 작고 단단하며 사용하기 쉬운 RPC 스타일로 Slack API를 모델링했다.
> 

### RPC의 장단점

**장점**

1. **고유 프로세스 개발에 집중** 가능 (**하부 네트워크 프로토콜에 신경쓰지 않아도** 되기 때문)
2. **프로세스간 통신 기능을 비교적 쉽게** 구현하고 정교한 제어 가능
3. 다양한 언어를 가진 환경에서 쉽게 확장 가능

**단점**

1. **호출 실행과 반환 시간이 보장되지 않음** (네트워크 구간을 통하여 RPC 통신을 하는 경우,  치명적 문제 발생)
2. 네트워크가 끊겼을 때 **보안이 보장되지 않음**
3. 로컬 함수의 경우 예외를 내거나, 값을 반환하지 않을 수 있다.
    1. 네트워크는 timeout으로 결과가 없는것 처럼 만들 수 있지만, 무슨일이 일어났는지 알 방법이 없다.
    2. 정말 요청을 제대로 보낸건지 아닌지를 구분하기도 어려워진다
4. 실패한 네트워크 요청이 처리를 실행되지만, 응답만 유실된 경우일 수도 있다.
    1. 이때 멱등성 idempotence을 적용하지 않는다면 재시도는 여러 작업이 중복 실행될 수 있다.
5. 로컬 함수 호출에 비해 훨씬 시간이 많이 소요되고 그 소요시간을 예측할 수 없다.
6. 로컬함수는 pointer를 효율적으로 전달할 수 있다. 네트워크 요청은 부호화 하여 매개변수로 보내야 한다.
    1. 만약 큰 객체를 보내야 하는 상황이라면?
7. client와 서비스는 다른 언어로 구현할 수 있다. 따라서 RPC 프레임워크는 하나의 언어에서 다른 언어로 데이터 타입을 변환해야 한다.

이러한 한계와 문제가 있더라도 네트워크 통신을 로컬 함수처럼 사용하려는 수고는 비효율적인 것이 아니다.

### ****RPC REST 차이점****

**RPC는 내부 마이크로 서비스 제공을 위한 고객별 API 구현에 사용된다.** RPC는 단일 공급자와 단일 소비자를 직접 연결하기 위해, REST API 처럼 유선으로 수 많은 메타데이터 주고 받는데 많은 시간을 할애하기를 원하지 않는다.

|  | RPC | REST |
| --- | --- | --- |
| 정의 | 원격 클라이언트에서 서버의 프로시저를 마치 로컬인 것처럼 직접적으로 호출할 수 있게 하는 시스템 | 클라이언트와 서버 간의 정형 데이터 교환을 정의하는 일련의 규칙 |
| 용도 | 원격 서버에서 작업을 수행 | 원격 객체에 대한 생성, 읽기, 업데이트 및 삭제(CRUD) 작업을 수행 |
| 활용사례  | 복잡한 계산이 필요하거나 서버에서 원격 프로세스를 트리거하는 경우 | 서버 데이터 및 데이터 구조를 균일하게 노출해야 하는 경우 |
| 상태유지여부 | 상태 유지 또는 무상태 | 무상태 |
| 데이터 전달 형식 | 서버에서 정의하고 클라이언트에 적용되는 일관된 구조 | 서버에 의해 독립적으로 결정되는 구조. 동일한 API 내에서 여러 형식을 전달할 수 있다. |

## 비동기적 메시지 전달을 통한 데이터 플로

- 메시지를 직접 네트워크 연결로 전송하지 않고 임시로 메시지를 저장하는 메시지 브로커라는 중간 단계를 거쳐 전송한다는 점에서 데이터베이스와 유사하다.
- 클라이언트의 요청(메시지)을 낮은 지연 시간으로 다른 프로세스에 전달한다는 점에서는 RPC와 비슷하지만 **메시지 브로커(Message broker, Message Queue) 방식은 RPC 방식보다 여러 장점이 있다.**
    - 수신자가 사용 불가 상태에 빠져도 메시지 브로커가 버퍼로 동작할 수 있기 때문에 시스템 안정성이 향상된다.
    - 메시지 브로커가 메시지를 저장하므로 메시지 유실을 방지할 수 있다.
    - 송신자와 수신자가 서로 알 필요가 없으므로 서로간에 의존성이 없다.
        - 논리적으로 송신자와 수신자는 분리됨 → 송신자는 메시지를 게시(publish)할 뿐이고 누가 소비(consume)하는지 상관하지 않음
    - 하나의 메시지를 여러 수신자로 전송할 수 있다.
- **RPC 차이점 :** 메시지 전달 방식은 송신 프로세스가 응답을 기대하지 않으므로 **단방향**이다.
    - 프로세스가 응답을 전송하는 것은  가능하지만 이것은 보통 별도의 채널에서 수행된다. 이러한 통신 패턴을 `비동기`라 한다.

### 분산 액터 프레임워크

- 액터 모델(actor model)은 단일 프로세스 안에서 동시성을 위한 프로그래밍 모델
    - 예시) Akka, Orleans, erlang
    
    > 액터 모델은 “모든 것은 액터다(Everything is an actor)”라는 기본 철학을 가지고 갑니다. OOP에서의 “모든 것은 객체다(Everything is an object)”라는 철학과 비슷하지만, 객체 지향 소프트웨어는 기본적으로 순차적 실행을 하지만 액터 모델은 본질적으로 동시성을 제공하는 점이 다릅니다.
    > 
    > 
    > 그러면 액터(Actor)란 무엇이냐 라는 질문이 나올 수 있습니다. 액터는 비동기적으로 메세지를 처리할 수 있는 computational entity로 다음과 같은 특징이 있습니다:
    > 
    > - 다른 액터에게 유한한 개수의 메세지를 보낼 수 있습니다.
    > - 유한한 개수의 새로운 액터를 만들 수 있습니다.
    > - 다른 액터가 받을 메세지에 수반될 행동(behavior)을 지정할 수 있습니다.
    > 
    > 실제 사람과의 커뮤니케이션을 상상하면 좀 더 이해하기 편합니다. 사람들은 초능력이 존재하지 않기에 타인과 머릿속 생각을 직접 공유하지 못하고, 대화(message)를 통해 대화를 주고받습니다.
    > 
    > 액터 또한 똑같습니다. 메세지를 주고받아 다른 액터와 상호작용을 합니다. 액터가 차지하는 메모리 공간은 독립적이며, 다른 스레드나 액터가 접근할 수 없습니다. 다시 말하면, 메모리 공유 없이 메세지 전달만을 사용하기에 공유 메모리로 인한 교착 상태 등의 골치 아픈 상황들을 피할 수 있습니다.
    > 
    > 참고 | **[Akka 공부하기 - 00.액터 모델이란?](https://blog.rajephon.dev/2018/11/25/akka-00/)**
    >
