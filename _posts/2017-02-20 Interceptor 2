---
layout: post
title: Spring Interceptor를 사용한  Session 관리 2
cover: cover.jpg
---

* * *

<br> 

# Interceptor를 사용한 Session 관리 2


## 개요

지난번 Interceptor를 이용한 Session관리에서는 HttpServlet Request의
Session객체에 접근하는 방식을 사용해서 구현을 했다. 
하지만 이번에 서버 스케일 아웃을하면서 Redis나 서버DB내에 사용자의 Session을
관리하는 방식으로 바뀌면서 Interceptor의 역할 또한 바뀌게 되었다. 

## 기존의 LoginInterceptor

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

기존의 LoginInterceptor는 현재 HttpSession에 UserId값이 있는지를 확인하고 없을시에 480 Error를 던지는 흐름이었다.


## Redis를 사용할떄의 LoginInterceptor

Reids에 로그인한 UserId값을 저장할때 Key값으로 로그인을 시도한 사용자의 HttpRequest안의 Session의 Id를 사용했다 
(이 Id값은 톰캣이 자동으로 발행하는 Cookie값으로 알고있다)

```
public class LoginCheckInterceptor extends HandlerInterceptorAdapter {
    @Autowired
    private RedisTemplate<String, String> redisTemplate;
    
    @Resource(name="redisTemplate")
    private ValueOperations<String, String> valueOps;
    
    
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    	String session = valueOps.get(request.getSession(false).getId());
    	
        if (session != null && session.length() > 0) {
            //Redis Has Session
            return true;
        }

        response.sendError(480, "Need TO Login");

        return false;
    }

}

```

Redis를 사용한 경우에 Interceptor는 요청을 한 사용자의 쿠키값을 가지고 Redis내에 사용자의 키값이 있는지 확인한 후에 없을 시에 
480 Error를 던지게 된다. 

## DataBase에 Session정보를 가지고 있을 시에 LoginInterceptor

DB상에에 로그인한 UserId값을 저장할때 로그인을 시도한 사용자의 HttpRequest안의 Session의 Id와 userId, 로그인 시간을 저장했다 

```
public class LoginCheckInterceptor extends HandlerInterceptorAdapter {

	@Autowired
	private UserService userService;

	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
	    
		String session = userService.findUserIdBySessionId(request.getSession(false).getId()).getContent();

		if (session != null && session.length() > 0) {
			return true;
		}

		response.sendError(480, "Need TO Login");

		return false;
	}

}

```

이경우에는 현재 DB에 요청을을 한 사용자의 쿠키값에 해당하는 row가 있는지를 확인하고 없을때에 480 error를 던지게 된다. 


## 개선해야 할 점

Interceptor를 변형해가면서 신경썼던 부분은 사용자의 cookie에 해당하는 값이 DB나 Redis상에 있는지 확인하고
없으면 480에러 있으면 Controller로 전달하는 단순한 부분일 뿐이다. 
사용자는 자신의 쿠키값만 가지고 있고 Controller나 Service가 필요한 UserId는 가지고 있지 않다. 
이부분에 대해 현재 프로젝트는 Web상에서 서버와의 통신을 통해 사용자의 userId를 가지고 있다가 Controller로 요청시에 Parameter로 전달한다.
하지만 Web상에서 parameter로 UserId를 보내는 방식은 조금만 생각해보면 상당히 위험한 방식이라는 생각이 든다. 
요청이 가다가 손실이 될 수 도 있고, 다른사람의 UserId를 보내는 식으로 사용자가 요청을 만들어 낼 수도 있다. 
따라서 앞으로 Interceptor에서 로그인 여부를 판별하면서 UserId를 Controller에게 보내는 방식으로 개선을 해 나가려고 한다. 
