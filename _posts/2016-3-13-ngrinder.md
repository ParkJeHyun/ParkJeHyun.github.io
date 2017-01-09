---
layout: post
title: nGrinder를 써보자
cover: cover.jpg
---

* * *

<br> 

# nGrinder 


## 설명

nGrinder는 네이버에서 2011년부터 제공하고 있는 오픈소스 성능 측정 도구입니다. 

웹 서비스를 개발하면서 개발한 서비스의 성능에 관해서 관심이 모두 많으실 거로 생각합니다. 

(이번에 특히나 조회수 관련으로 write-back을 구현하면서 많은 조회수 관련 트랜잭션이 있을 경우 조회수가 원하는 방식대로 write-back을 지원하는지 테스트 하고 싶으신 분들도 계실 것입니다.)

그러한 테스트를 하면서 원칙에 입각한 정확한 수치 자료를 얻고 싶다면 성능 측정 도구를 사용하는 것은 필연적입니다.

성능 테스트라하면 로드 테스트와 스트레스 테스트 등 여러 측면에 테스트가 있을 수 있습니다.

nGrinder는 이러한 성능 테스트를 간편하게 혹은 스크립트를 작성하여 복잡한 테스트까지 커버할 수 있도록 지원합니다. 



## 실행 방법

ngrinder는 오픈소스로 제공되어 있기 때문에 

https://sourceforge.net/projects/ngrinder/files/ 에서 직접 다운 받으실 수도 있습니다. 

설치 방법에도 로컬에서 직접 실행하거나 서버 환경에서 실행할 수도 있고 방법은 다양합니다. (아래 하단 링크를 참조하시면 됩니다.)

다만 결국 테스트를 실행하기 위한 리소스를 로컬에서 실행하는 것까진 괜찮겠지만,
설치 및 세팅에도 어느 정도 코스트가 들고 서비스가 실행 중인 웹서버에서 실행하는 것은 테스트를 위한 환경의 본질적인 면에서 충돌되므로 지양합니다. 

> **따라서 사내에서는 ngrinder를 쉽게 테스트할 수 있는 환경을 제공하고 있습니다.  
>  비공개 URL
 

 
## 메인 화면

<img src="/images/posts/ngrinder1.png">
 
 로그인을 하면서 위와 같은 화면을 보실 수 있습니다. 
 간단하게 테스트를 할 수 있도록 Quick Start 메뉴가 있는데 이곳에 원하는 서비스의 URL을 입력합니다. 
 테스트를 측정하기 위한 스크립트 언어를 선택할 수 있는데 groovy와 jython, groovy maven project 중에 선택할 수 있습니다.


 
## 설정 페이지

<img src="/images/posts/ngrinder2.png">

에이전트의 수와 에이젠트별 가상사용자 수를 지정하고
지역을 **★★IDC**로 설정합니다.

테스트 기간의 옵션으로 실행하면 해당 시간동안 테스트를 계속 진행하고
횟수로 진행할 경우 그 횟수만큼 진행합니다. 
(실행 일정의 경우에도 예약을 통해 실행 할 수 있습니다.)

좀 더 다양한 옵션이나 검증용 테스트를 커스터마이징 하기 위해서는 스크립트를 수정해야합니다.

스크립트에서 R HEAD를 클릭하면 그루비스크립트 코드 내용을 보실 수 있습니다.
(메인에서 설정했던 스크립트 방식으로 생성됩니다.)


## 스크립트

<img src="/images/posts/ngrinder3.png">

구조를 딱 보시면 눈치채신 분들도 있으시겠지만 저희가 이제까지 작성했었던 TC와 개념적으로 같습니다. 

```
@RunWith(GrinderRunner)
class TestRunner {

	public static GTest test
	public static HTTPRequest request
	public static NVPair[] headers = []
	public static NVPair[] params = []
	public static Cookie[] cookies = []

	@BeforeProcess
	public static void beforeProcess() {
		HTTPPluginControl.getConnectionDefaults().timeout = 6000
		test = new GTest(1, "비공개 url")
		request = new HTTPRequest()
		grinder.logger.info("before process.");
	}

	@BeforeThread 
	public void beforeThread() {
		test.record(this, "test")
		grinder.statistics.delayReports=true;
		grinder.logger.info("before thread.");
	}
	
	@Before
	public void before() {
		request.setHeaders(headers)
		cookies.each { CookieModule.addCookie(it, HTTPPluginControl.getThreadHTTPClientContext()) }
		grinder.logger.info("before thread. init headers and cookies");
	}

	@Test
	public void test(){
		HTTPResponse result = request.GET("비공개 url", params)

		if (result.statusCode == 301 || result.statusCode == 302) {
			grinder.logger.warn("Warning. The response may not be correct. The response code was {}.", result.statusCode); 
		} else {
			assertThat(result.statusCode, is(200));
		}
	}
}
```
리퀘스트에 매소드 방식부터, 헤더의 값이나 쿠키값을 테스트에 원하는 값으로 입력하여 테스트의 검증 방법까지 지정할 수 있습니다. 
따라서 로그인이 필요하거나 특정 정보가 필요한 페이지나 리소스가 필요한 경우에도 성능 측정을 할 수 있습니다. 

스크립트를 모두 작성한 후 '스크립트 검증'을 통해 테스트가 제대로 돌아가는지 확인합니다.

테스트를 위한 테스트 코드가 확인이 완료되었으면 '저장 후 시작'을 통해 테스트를 실행 할 수 있습니다. 


## 보고서 화면 

<img src="/images/posts/ngrinder3.png">

테스트가 완료되면 위와 같은 보고서 결과를 얻을 수 있습니다. 테스트 조건을 다양하게 한다면 더 가치 있는 테스트를 하실 수 있습니다.


> (위의 사항들에 대해서 더욱 자세한 내용은 [BaseCamp-공지사항/228 &#91;공유&#93; 작년 성능 강의자료 공유]() 에 첨부파일 중 ngrinder 부분을 참고해주시길 바랍니다.)
 

---

` 마지막으로 이러한 성능 측정 자체가 DDoS 공격과 의미상으로 같으므로 필요 이상의 강도 높은 스트레스 테스트를 남발하시는 것은 주의하셔야 할 것 같습니다. ㅠㅠ `


---
### 참고 사이트 

* [http://naver.github.io/ngrinder/](http://naver.github.io/ngrinder/)
* [http://deview.kr/2013/detail.nhn?topicSeq=2](http://deview.kr/2013/detail.nhn?topicSeq=2)
* [http://junoyoon.tistory.com/entry/nGrinder-33-%EC%84%A4%EC%B9%98%EB%B0%A9%EB%B2%95](http://junoyoon.tistory.com/entry/nGrinder-33-%EC%84%A4%EC%B9%98%EB%B0%A9%EB%B2%95)
* [http://www.groovy-lang.org/](http://www.groovy-lang.org/)