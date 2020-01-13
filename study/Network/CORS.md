# CORS(Cross-Origin Resource Sharing)

## CORS란?

- 추가적인 HTTP 헤더를 사용함으로써 브라우저에게 '한 origin에서 실행중인 웹 어플리케이션이 다른 origin에서 온 선택된 자원에 접근 할 수 있도록' 알리기 위한 메커니즘
- 웹 어플리케이션은 자신과 다른 origin에게 리소스를 요청할 때 cross-origin HTTP 요청을 실행한다.

> origin이 다르다 === 도메인, 프로토콜, 포트번호 등 중 하나라도 다르다.

> #### cross origin 요청 예
> 'https://domain-a.com'의 Front-end에서 실행중인 Javascript 코드가 XMLHttpRequest(이하 XHR)를 이용하여 'https://domain-b.com/data.json.'에 리소스를 요청

- 보안적인 이유로 브라우저는 스크립에서 초기화되는 cross origin HTTP 요청을 제한
- 예로 XHR과 Fetch API은 same-origin 정책을 따른다.
	- 이는 기본적으로 API를 사용하는 웹 어플리케이션은 로드된 곳과 같은 origin에만 자원을 요청할 수 있다는 말이다.
	- 하지만 올바른 CORS 헤더를 포함한 응답이라면 다른 origin에도 요청이 가능하다.

- CORS 메커니즘은 브라우저와 서버간 안전한 cross-origin 요청과 데이터 전송을 지원한다.
- 최신 브라우저들은 XHR, Fetch와 같은 API에서 CORS를 사용하여 cross-origin 요청의 위험을 완화시킨다.

## CORS를 사용하는 요청

- cross-origin sharing 표준은 cross-site HTTP 요청을 허가할 수 있는 목록 
	1. XHR 혹은 Fetch API의 호출 
	2. CSS에서 사용하는 웹 폰트
		- 서버는 허가된 cross-site에서만 로드되고 사용될 수 있는 TrueType 폰트를 배포할 수 있다.
	3. WebGL textures
	4. canvas를 사용하여 그려진 이미지/동영상 프레임 
	5. 이미지 기반의 CSS 형태(CSS Shapes from images)

- 이 글은 일반적으로 이야기되는 Cross-Origin Resource Sharing이고 필수 HTTP 헤더에 대한 이야기가 포함된다.

## 개요

- Cross-Origin Resource Sharing 표준은 서버가 어떤 origin이 이 정보를 브라우저에서 읽는 것이 허가되었는지에 대해 명시할수 있게 해주는 새로운 HTTP 헤더를 추가함으로써 동작한다.
- 추가적으로 서버 데이터의 사이드 에펙트를 유발 할 수 있는 HTTP 요청 메서드(특히 	GET이 아닌, 특정 MIME 타입을 사용하는  POST 요청)를 위해 명세서에서는 브라우저에 preflight 요청을 강제한다.
- 지원되는 메서드를 서버에 HTTP OPTIONS 요청 메서드와 함께 요청하고 서버에서 승인한다면 실제 요청을 전송한다.
- 서버는 클라이언트에게 credentials(쿠키나 HTTP 인증)이 요청과 함께 와야 하는지에 대해 알릴 수 도 있다.

- CORS 실패는 에러의 원인이 되지만 보안상의 이유로 Javascript에서는 세부적인 에러에 대해 알 수 없다.
- 코드는 단지 에러가 발생했다는 것만 안다.
- 문제점의 세부 사항을 알기 위한 유일한 식별법은 브라우저 콘솔에서 확인할 수 밖에 없다.

- 계속해서 시나리오와 사용중인 HTTP 헤더의 분석에 대해 보겠다.

## 접근 제어 시나리오 예

- 어떻게 Cross-Origin Resource Sharing이 작동되는지에 대한 세가지 시나리오가 있다.
- 이 예들은 어떤 브라우저에서도 cross-site 요청을 할 수 있는 XHR을 사용할 것이다.

### Simple requests

- 몇몇 요청은 CORS preflight을 트리거하지 않는다.
- 이러한 요청을 simple request라 한다. 하지만 (CORS를 정의하는)Fetch spec에서는 이러한 용어를 사용하지 않는다.
- simple requests는 다음과 같은 다음과 같은 조건을 모두 충족해야 한다.
Those are called “simple requests” in this article, though the Fetch spec (which defines CORS) doesn’t use that term. A “simple request” is one that meets all the following conditions:

	- 메서드 : GET, HEAD, POST 중 하나.
	- user agent가 자동으로 설정한 헤더와 다른 헤더 외에 Fetch spec에서 "CORS-safelisted request-header"라고 정의 된 수동으로 설정 가능한 헤더
        - Examples:
            - Accept
            - Accept-Language
            - Content-Language
            - Content-Type (but note the additional requirements below)
            - DPR
            - Downlink
            - Save-Data
            - Viewport-Width
            - Width

	> user agent가 자동으로 설정한 헤더와 다른 헤더란?
	 Connection, User-Agent, 혹은 "forbidden header name"과 같은 Fetch spec에 정의된 헤더

	- Content-Type 헤더에 허용된 값은 다음과 같다.
		- application/x-www-form-urlencoded
		- multipart/form-data
		- text/plain
	- 요청에 사용된 어떤 XMLHttpRequestUpload object에도 이벤트 리스너는 등록되어 있지 않다. 이벤트 리스너는 모두 XMLHttpRequest.upload 속성을 사용해서 엑세스 해야 한다.
	- 어떤 ReadableStream object도 요청에서 사용되지 않는다.

	> Note: 이것은 웹 컨텐츠가 이미 발행한 것과 같은 종류의 cross-site 요청이고 서버가 적절한 헤더를 보내지 않으면 요청한 주체에게 응답 데이터가 공개되지 않는다. 따라서 cross-site 요청 위조를 방지하는 사이트는 HTTP 접근 제어에서 신경 쓸 것이 없다.
	
	> Note: WebKit Nightly 및 Safari Technology Preview는 Accept, Accept-Language 및 Content-Language 헤더에서 허용되는 값에 대한 추가 제한을 설정한다. 만약 이 헤더 중 하나라도 "nonstandard" 값이 있으면 WebKit/Safari는 이 요청을 simple request로 여기지 않는다. 다음 WebKit버그를 제외하고 WebKit/Safari가 "nonstandard"라 판단한 값은 문서화되어 있지 않다.
	> - non-standard CORS-safelisted 요청의 헤더 Accept, Accept-Language, Content-Language에서 preflight을 요청하는 것 (Require preflight for non-standard CORS-safelisted request headers Accept, Accept-Language, and Content-Language)
	> - simple CORS를 위한 Accept, Accept-Language, Content-Language 요청 헤더에서 쉼표를 허용하는 것 (Allow commas in Accept, Accept-Language, and Content-Language request headers for simple CORS)
	> - simple CORS 요청에서 제한된 Accept 헤더에 대한 블랙리스트 모델로 전환 (Switch to a blacklist model for restricted Accept headers in simple CORS requests)
	> <br>
    > 다른 브라우저에서는 이런 사항들은 spec에 없기 때문에 이런 추가적인 제한사항이 구현되어 있지 않다.

- 예를 들어 https://foo.example 의 웹 컨텐츠가 https://bar.other 의 컨텐츠를 호출하려 한다 가정하자. 이런 코드가 foo.example에서 사용되었을 것이다.

```Javascript
const xhr = new XMLHttpRequest();
const url = 'https://bar.other/resources/public-data/';
   
xhr.open('GET', url);
xhr.onreadystatechange = someHandler;
xhr.send(); 
```

- 이것은 CORS 헤더를 사용하여 권한을 처리하는 클라이언트와 서버 간 간단한 교환작업이 수행된다.

![](https://mdn.mozillademos.org/files/14293/simple_req.png)

- 그렇다면 이 상황에서 브라우저가 서버에서 보내는 코드와 서버의 응답코드를 살펴보자.

```
GET /resources/public-data/ HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:71.0) Gecko/20100101 Firefox/71.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Connection: keep-alive
Origin: https://foo.example
```
- 이 요청 헤더는 Origin에 관한 것이고 호출이 https://foo.example에서 온 것임을 보여준다.

```
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 00:23:53 GMT
Server: Apache/2
Access-Control-Allow-Origin: *
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Transfer-Encoding: chunked
Content-Type: application/xml

[…XML Data…]
```

- 응답에서, 서버는 ```Access-Control-Allow-Origin``` 헤더로써 돌려 준다.
- Origin 헤더와 ```Access-Control-Allow-Origin```의 사용은  접근 제어 프로토콜을 가장 단순하게 사용했다는 것을 보여준다.
- 이 예에서 서버는 ```Access-Control-Allow-Origin: *```로 응답하는데 어떤 도메인에서도 접근 가능한 리소스 임을 의미한다.
- 만약 ```https://bar.other```의 자원 소유자가 ```https://foo.example```에서만 리소스 접근을 허용하게 하고 싶다면 다음과 같이 보내면 된다.

> Access-Control-Allow-Origin: https://foo.example

- 이제 ```https://foo.example```이 아닌 다른 도메인에서는 cross-site manner에 의해 리소스에 접근 할 수 없다.
- 접근을 허가받기 위해서는 ```Access-Control-Allow-Origin``` 헤더가 요청의 <mark>Origin</mark> 헤더에 보낸 값을 포함하고 있어야 한다.

### Preflighted requests

- simple requests와는 다르게 preflighted 요청은 다른 도메인의 리소스로 OPTIONS 메서드의 HTTP 요청을 먼저 보내서 실제 요청이 안전하게 보내지는지 판단한다.
- Cross-site 요청은 이런식으로 preflight 되는데 사용자 데이터에 영향을 끼칠 수 도 있기 때문이다.

- 다음은 preflight 요청이 전송되는 예이다.

```
const xhr = new XMLHttpRequest();
xhr.open('POST', 'https://bar.other/resources/post-here/');
xhr.setRequestHeader('X-PINGOTHER', 'pingpong');
xhr.setRequestHeader('Content-Type', 'application/xml');
xhr.onreadystatechange = handler;
xhr.send('<person><name>Arun</name></person>'); 
```

- 이 예는 
The example above creates an XML body to send with the POST request. Also, a non-standard HTTP X-PINGOTHER request header is set. Such headers are not part of HTTP/1.1, but are generally useful to web applications. Since the request uses a Content-Type of application/xml, and since a custom header is set, this request is preflighted.

<img src="https://mdn.mozillademos.org/files/16753/preflight_correct.png"/>

(Note: as described below, the actual POST request does not include the Access-Control-Request-* headers; they are needed only for the OPTIONS request.)

Let's look at the full exchange between client and server. The first exchange is the preflight request/response:

```
OPTIONS /resources/post-here/ HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:71.0) Gecko/20100101 Firefox/71.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Connection: keep-alive
Origin: http://foo.example
Access-Control-Request-Method: POST
Access-Control-Request-Headers: X-PINGOTHER, Content-Type


HTTP/1.1 204 No Content
Date: Mon, 01 Dec 2008 01:15:39 GMT
Server: Apache/2
Access-Control-Allow-Origin: https://foo.example
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: X-PINGOTHER, Content-Type
Access-Control-Max-Age: 86400
Vary: Accept-Encoding, Origin
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
```

Once the preflight request is complete, the real request is sent:

```
POST /resources/post-here/ HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:71.0) Gecko/20100101 Firefox/71.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Connection: keep-alive
X-PINGOTHER: pingpong
Content-Type: text/xml; charset=UTF-8
Referer: https://foo.example/examples/preflightInvocation.html
Content-Length: 55
Origin: https://foo.example
Pragma: no-cache
Cache-Control: no-cache

<person><name>Arun</name></person>


HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:40 GMT
Server: Apache/2
Access-Control-Allow-Origin: https://foo.example
Vary: Accept-Encoding, Origin
Content-Encoding: gzip
Content-Length: 235
Keep-Alive: timeout=2, max=99
Connection: Keep-Alive
Content-Type: text/plain

[Some GZIP'd payload]
```

Lines 1 - 11 above represent the preflight request with the OPTIONS method. The browser determines that it needs to send this based on the request parameters that the JavaScript code snippet above was using, so that the server can respond whether it is acceptable to send the request with the actual request parameters. OPTIONS is an HTTP/1.1 method that is used to determine further information from servers, and is a safe method, meaning that it can't be used to change the resource. Note that along with the OPTIONS request, two other request headers are sent (lines 10 and 11 respectively):

```
Access-Control-Request-Method: POST
Access-Control-Request-Headers: X-PINGOTHER, Content-Type
```

The Access-Control-Request-Method header notifies the server as part of a preflight request that when the actual request is sent, it will be sent with a POST request method. The Access-Control-Request-Headers header notifies the server that when the actual request is sent, it will be sent with a X-PINGOTHER and Content-Type custom headers. The server now has an opportunity to determine whether it wishes to accept a request under these circumstances.

Lines 14 - 23 above are the response that the server sends back indicating that the request method (POST) and request headers (X-PINGOTHER) are acceptable. In particular, let's look at lines 17-20:

```
Access-Control-Allow-Origin: http://foo.example
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: X-PINGOTHER, Content-Type
Access-Control-Max-Age: 86400
```

The server responds with Access-Control-Allow-Methods and says that POST and GET are viable methods to query the resource in question. Note that this header is similar to the Allow response header, but used strictly within the context of access control.

The server also sends Access-Control-Allow-Headers with a value of "X-PINGOTHER, Content-Type", confirming that these are permitted headers to be used with the actual request. Like Access-Control-Allow-Methods, Access-Control-Allow-Headers is a comma separated list of acceptable headers.

Finally, Access-Control-Max-Age gives the value in seconds for how long the response to the preflight request can be cached for without sending another preflight request. In this case, 86400 seconds is 24 hours. Note that each browser has a maximum internal value that takes precedence when the Access-Control-Max-Age is greater.

### Preflighted requests and redirects
Not all browsers currently support following redirects after a preflighted request. If a redirect occurs after a preflighted request, some browsers currently will report an error message such as the following.


> The request was redirected to 'https://example.com/foo', which is disallowed for cross-origin requests that require preflight

> Request requires preflight, which is disallowed to follow cross-origin redirect

The CORS protocol originally required that behavior but was subsequently changed to no longer require it. However, not all browsers have implemented the change, and so still exhibit the behavior that was originally required.

Until browsers catch up with the spec, you may be able to work around this limitation by doing one or both of the following:

Change the server-side behavior to avoid the preflight and/or to avoid the redirect
Change the request such that it is a simple request that doesn’t cause a preflight
If that's not possible, then another way is to:

Make a simple request (using Response.url for the Fetch API, or XMLHttpRequest.responseURL) to determine what URL the real preflighted request would end up at.
Make another request (the “real” request) using the URL you obtained from Response.url or XMLHttpRequest.responseURL in the first step.
However, if the request is one that triggers a preflight due to the presence of the Authorization header in the request, you won’t be able to work around the limitation using the steps above. And you won’t be able to work around it at all unless you have control over the server the request is being made to.

Requests with credentials
The most interesting capability exposed by both XMLHttpRequest or Fetch and CORS is the ability to make "credentialed" requests that are aware of HTTP cookies and HTTP Authentication information. By default, in cross-site XMLHttpRequest or Fetch invocations, browsers will not send credentials. A specific flag has to be set on the XMLHttpRequest object or the Request constructor when it is invoked.

In this example, content originally loaded from http://foo.example makes a simple GET request to a resource on http://bar.other which sets Cookies. Content on foo.example might contain JavaScript like this:

const invocation = new XMLHttpRequest();
const url = 'http://bar.other/resources/credentialed-content/';
    
function callOtherDomain() {
  if (invocation) {
    invocation.open('GET', url, true);
    invocation.withCredentials = true;
    invocation.onreadystatechange = handler;
    invocation.send(); 
  }
}
Line 7 shows the flag on XMLHttpRequest that has to be set in order to make the invocation with Cookies, namely the withCredentials boolean value. By default, the invocation is made without Cookies. Since this is a simple GET request, it is not preflighted, but the browser will reject any response that does not have the Access-Control-Allow-Credentials: true header, and not make the response available to the invoking web content.



Here is a sample exchange between client and server:

GET /resources/access-control-with-credentials/ HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:71.0) Gecko/20100101 Firefox/71.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Connection: keep-alive
Referer: http://foo.example/examples/credential.html
Origin: http://foo.example
Cookie: pageAccess=2


HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:34:52 GMT
Server: Apache/2
Access-Control-Allow-Origin: https://foo.example
Access-Control-Allow-Credentials: true
Cache-Control: no-cache
Pragma: no-cache
Set-Cookie: pageAccess=3; expires=Wed, 31-Dec-2008 01:34:53 GMT
Vary: Accept-Encoding, Origin
Content-Encoding: gzip
Content-Length: 106
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Content-Type: text/plain


[text/plain payload]
Although line 10 contains the Cookie destined for the content on http://bar.other, if bar.other did not respond with an Access-Control-Allow-Credentials: true (line 17) the response would be ignored and not made available to web content.

Credentialed requests and wildcards
When responding to a credentialed request, the server must specify an origin in the value of the Access-Control-Allow-Origin header, instead of specifying the "*" wildcard.

Because the request headers in the above example include a Cookie header, the request would fail if the value of the Access-Control-Allow-Origin header were "*". But it does not fail: Because the value of the Access-Control-Allow-Origin header is "http://foo.example" (an actual origin) rather than the "*" wildcard, the credential-cognizant content is returned to the invoking web content.

Note that the Set-Cookie response header in the example above also sets a further cookie. In case of failure, an exception—depending on the API used—is raised.

Third-party cookies
Note that cookies set in CORS responses are subject to normal third-party cookie policies. In the example above, the page is loaded from foo.example, but the cookie on line 20 is sent by bar.other, and would thus not be saved if the user has configured their browser to reject all third-party cookies.

The HTTP response headers
This section lists the HTTP response headers that servers send back for access control requests as defined by the Cross-Origin Resource Sharing specification. The previous section gives an overview of these in action.

Access-Control-Allow-Origin
A returned resource may have one Access-Control-Allow-Origin header, with the following syntax:

Access-Control-Allow-Origin: <origin> | *
Access-Control-Allow-Origin specifies either a single origin, which tells browsers to allow that origin to access the resource; or else — for requests without credentials — the "*" wildcard, to tell browsers to allow any origin to access the resource.

For example, to allow code from the origin https://mozilla.org to access the resource, you can specify:

Access-Control-Allow-Origin: https://mozilla.org
Vary: Origin
If the server specifies a single origin (that may dynamically change based on the requesting origin as part of a white-list) rather than the "*" wildcard, then the server should also include Origin in the Vary response header — to indicate to clients that server responses will differ based on the value of the Origin request header.

Access-Control-Expose-Headers
The Access-Control-Expose-Headers header lets a server whitelist headers that browsers are allowed to access.

Access-Control-Expose-Headers: <header-name>[, <header-name>]*
For example, the following:

Access-Control-Expose-Headers: X-My-Custom-Header, X-Another-Custom-Header
…would allow the X-My-Custom-Header and X-Another-Custom-Header headers to be exposed to the browser.

Access-Control-Max-Age
The Access-Control-Max-Age header indicates how long the results of a preflight request can be cached. For an example of a preflight request, see the above examples.

Access-Control-Max-Age: <delta-seconds>
The delta-seconds parameter indicates the number of seconds the results can be cached.

Access-Control-Allow-Credentials
The Access-Control-Allow-Credentials header Indicates whether or not the response to the request can be exposed when the credentials flag is true. When used as part of a response to a preflight request, this indicates whether or not the actual request can be made using credentials. Note that simple GET requests are not preflighted, and so if a request is made for a resource with credentials, if this header is not returned with the resource, the response is ignored by the browser and not returned to web content.

Access-Control-Allow-Credentials: true
Credentialed requests are discussed above.

Access-Control-Allow-Methods
The Access-Control-Allow-Methods header specifies the method or methods allowed when accessing the resource. This is used in response to a preflight request. The conditions under which a request is preflighted are discussed above.

Access-Control-Allow-Methods: <method>[, <method>]*
An example of a preflight request is given above, including an example which sends this header to the browser.

Access-Control-Allow-Headers
The Access-Control-Allow-Headers header is used in response to a preflight request to indicate which HTTP headers can be used when making the actual request.

Access-Control-Allow-Headers: <header-name>[, <header-name>]*
The HTTP request headers
This section lists headers that clients may use when issuing HTTP requests in order to make use of the cross-origin sharing feature. Note that these headers are set for you when making invocations to servers. Developers using cross-site XMLHttpRequest capability do not have to set any cross-origin sharing request headers programmatically.

Origin
The Origin header indicates the origin of the cross-site access request or preflight request.

Origin: <origin>
The origin is a URI indicating the server from which the request initiated. It does not include any path information, but only the server name.

Note: The origin value can be null, or a URI.
Note that in any access control request, the Origin header is always sent.

Access-Control-Request-Method
The Access-Control-Request-Method is used when issuing a preflight request to let the server know what HTTP method will be used when the actual request is made.

Access-Control-Request-Method: <method>
Examples of this usage can be found above.

Access-Control-Request-Headers
The Access-Control-Request-Headers header is used when issuing a preflight request to let the server know what HTTP headers will be used when the actual request is made.

Access-Control-Request-Headers: <field-name>[, <field-name>]*
Examples of this usage can be found above.

Specifications
Specification	Status	Comment
Fetch
The definition of 'CORS' in that specification.	Living Standard	New definition; supplants W3C CORS specification.
Browser compatibility
Update compatibility data on GitHub
Desktop
Mobile
Chrome
Edge
Firefox
Internet Explorer
Opera
Safari
Android webview
Chrome for Android
Firefox for Android
Opera for Android
Safari on iOS
Samsung Internet
Access-Control-Allow-Origin
Full support4
Full support12
Full support3.5
Full support10
Full support12
Full support4
Full support2
Full supportYes
Full support4
Full support12
Full support3.2
Full supportYes





What are we missing?
Legend
Full support 
Full support
Compatibility notes
Internet Explorer 8 and 9 expose CORS via the XDomainRequest object, but have a full implementation in IE 10


> Reference
> [MDN : Cross-Origin Resource Sharing](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)