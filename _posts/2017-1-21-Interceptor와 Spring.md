---
layout: post
title: Spring Interceptor를 사용한  Session 관리
cover: cover.jpg
---

* * *

<br> 

# Spring Interceptor를 사용한 Session 관리


## 개요

이번 신입 프로젝트를 진행하면서 내가 구현해야할 기능은 로그인, 회원가입등의 회원관리와
로그인을 했을 때의 세션을 유지시켜주고 로그 아웃시에 제거해주는 기능이었다.
로그인 Session을 관리하는데는 많은 방법들이 있지만 나는 이번 프로젝트에 Interceptor를 사용하여 
Session을 관리하는 방법을 선택했다.

## Interceptor란?

로그인세션체크에 대해 예를 들어 설명하면 
Iterceptor는 다음 그림과 같이 세션을 필요로 하는 요청이 Controller에 가기전에 세션을 체크 해주어서
세션체크가 성공하는 경우만 Controller에게 전해주는 기능을 하게 된다.

<img src="http://cfile27.uf.tistory.com/image/231D6B425462B05D058CD0">

## Interceptor의 사용

그러면 지금 진행중인 BUYCO 프로젝트에 작성한 내용을 통해 어떤식으로 구현하는지 확인해보자.

1. 먼저 HandlerInterceptorAdapter를 상속하는 InterceptorClass를 만든다. (LoginCheckInterceptor 라고 명명했다.)


```
public class LoginCheckInterceptor extends HandlerInterceptorAdapter{
	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
		throws Exception {
		HttpSession session = request.getSession(false);
		
		if(session != null) {
			//Request Has Session
			Integer user = (Integer)session.getAttribute("user");
			
			if(user != null) {
				//Request Has Session And User Information(id)
				
				return true;
			}
		}
		
		response.sendError(480, "Need TO Login");
		
		return false;
	}

}

```

preHandle 이란 메소드를 오버라이드 해서 구성했으며 session이 없는 경우 response에 480이란 에러와 Need TO Login 이라는 에러메시지를 실어서 보내도록 했다.
이처럼 에러를 보내면서 구현할 수도 있지만 response의 sendRedirect에 로그인 페이지의 url을 보내는 식으로 구현할 수도 있다.
    

2. 이제 생성한 Interceptor를 application-context.xml에 등록한다.
    이때 자신이 세션을 체크할 url도 함께 등록할 수 있다(여러개도 가능)

```

	<!-- InterCepter -->

	<mvc:interceptors>
    	<mvc:interceptor>
    		<mvc:mapping path="/isLogin/*"/>
    		<bean class="kr.nhn.buyco.interceptor.LoginCheckInterceptor"/>
    	</mvc:interceptor>
    </mvc:interceptors>

	
```

mapping path에는 interceptor로 먼저 매핑시킬 url
bean class에는 1번에서 제작한 Interceptor 클래스를 넣어준다.

interceptor는 여러개의 interceptor들을 만들어서 넣어줄 수 있다. 

3. Session의 등록 및 삭제

```
    @RequestMapping(value = "/login", method = RequestMethod.POST)
	public Integer login(User user, HttpSession session) {
		//Login User
		Integer ret = userService.login(user);
		if(ret > 0 ){
			session.setAttribute("user", ret);
		}
		return ret;
	}
	
	@RequestMapping(value = "/logout", method = RequestMethod.POST)
	public int logout(HttpSession session) {
		session.removeAttribute("user");
		
		return 0;
	}
	
```

프로젝트에서는 login 시 "user"라는 키로 user의 ID값 밸류를 넣어 놓았다. 
그리고 logout시에 해당 attribute를 제거 하는식으로 진행했다.


4. Interceptor로 가는 요청 사용 및 결과페이지

```
$.ajax({
		url: '/buyco/isLogin/' + url,
		success: function (responseData){
		    alert("Session 존재");
		},
		error: function (response, status, error){
			if(response.status == 480){
				alert("Session 없음");		
			}
		}
	});
```

<img src="/images/posts/session_error.png">

Chrome의 개발자도구에서 console을 보면 로그인 된 session이 없기때문에
/buyco/isLogin/myPage라는 요청에 Interceptor에 넣어놓은 480 error가 발생하는걸 알 수 있다.
또한 $.ajax통신의 error부분을 통해 에러 상황에 대해 컨트롤을 할 수도 있다.

## 느낀점

프로젝트에 Interceptor를 적용시키려 하면서 가장 고민이 많았던 부분은 우리 프로젝트는 로그인 페이지가 따로 있는게 아니라 modal창을 사용하기 때문에
sendRedirect를 사용하기 곤란하다는 점이었다.
그 때문에 480이라는 에러를 만들어서 에러를 넘기는 식으로 처리하게 되었다.
물론 지금 구현해놓은것이 정답이라 생각하지는 않지만 Session 유지 방법에 대해 깊이 생각해볼 수 있었다.


### 참고 사이트 

* http://hellogk.tistory.com/90


