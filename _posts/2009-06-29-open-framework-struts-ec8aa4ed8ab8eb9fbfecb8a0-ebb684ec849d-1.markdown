---
author: springloops
comments: true
date: 2009-06-29 04:06:00+00:00
layout: post
link: https://springloops.wordpress.com/2009/06/29/open-framework-struts-%ec%8a%a4%ed%8a%b8%eb%9f%bf%ec%b8%a0-%eb%b6%84%ec%84%9d-1/
slug: open-framework-struts-%ec%8a%a4%ed%8a%b8%eb%9f%bf%ec%b8%a0-%eb%b6%84%ec%84%9d-1
title: 'Open FrameWork - Struts : 스트럿츠 분석 (1)'
wordpress_id: 67
categories:
- Open Source
tags:
- 분석
- framework
- 스트럿츠
- Java
- Open Source
- Struts
---

  




**스트럿츠 분석(1)**




****




**시작하며…**




일반 서블릿을 이용한 Web Application의 구조와 스트럿츠를 활용한 Web Application 의 구조를 비교함으로써,   

  
스트럿츠의 목적 및 장점을 파악하고, 스트럿츠의 내부 흐름을 분석해보자.**   

  
  

  



[#M_내용 보기|접기|각 Web Application 의 구조  

  



Java Web Application 의 기본이라 할수 있는 Servlet/JSP 기반으로 된 구조먼저 알아보자.  

  
  

기본 Web Application 의 구조  

![](http://localhost/wordpress/wp-content/uploads/1/cfile9.uf.153A8A0E4AD5554D640DEE.jpg)[그림 1] 서블릿을 이용한 Web Application 의 구조  

  
위 그림에서 보는 것과 같이 사용자의 Request 는 각 요청의 Action 에 해당하는   

  
Servlet 을 호출하고 각 Servlet 의 기능에 따라 Model 부분을 수행하고   

  
View 부분을 통해 사용자에게 Response 하게 됩니다.  

  
매우 간단한 구조를 가지고 있습니다.  

  
다음으로 스트럿츠를 이용하여 개발하게 되는 Java Web Application 의 구조를 확인해 보겠습니다.  

  
스트럿츠의 구조  

![](http://localhost/wordpress/wp-content/uploads/1/cfile8.uf.173A8A0E4AD5554E654317.jpg)[그림 2]스트럿츠를 이용한 Web Application 의 구조  

  
스트럿츠를 이용한 Web Application 의 구조를 보면 Control 영역이 많이 복잡해 지고   

  
Veiw 단에 하나의 항목이 추가 된 것을 확인할수 있습니다.  

  
그림 1에서 본 서블릿 기반의 것도 컨트롤러 이고 그림2와 같이 스트럿츠 역시 컨트롤러입니다.  

  
컨트롤러를 스트럿츠와 같이 변화여 발생하는 차이점과 이점은 무엇이 있을까요?  

  
먼저 차이점을 찾아보고 그로 인한 시스템의 영향을 생각해 봅시다.  

  
자연스럽게 스트럿츠의 기능과 장점에 대해서 알수 있겠지요.  

  
  

**차이점과 흐름.  

  
**서블릿(그림1)도 스트럿츠(그림2)도 MVC 구조를 가지고 있습니다.  

  
간단한 차이점부터 설명해봅시다.  

  
스트럿츠는 View 단에 커스텀 태그 라이브러리를 제공하네요.   

  
그렇다고 스트럿츠가 view 컴포넌트라고 생각하시는 분은 없길 바랍니다.  

  
모델영역은 오직 서블릿으로 되어 있는 환경이나 스트럿츠를 활용한 환경이나 똑같습니다.  

  
스트럿츠는 모델에서는 해주는 일이 아무것도 없나 봅니다.  

  
그럼 가장 많은 변화가 있는 컨트롤러 영역을 봅시다.  

  
요청이 오는대로 곧장 요청된 각각의 서블릿을 찾아 요청에 대한 처리를 수행하는 서블릿 환경 과는 다르게,  

  
스트럿츠는 사용자의 모든 요청을 ActionServlet이 받아 처리합니다.  

  
ActionServlet은 스트럿츠의 설정을 초기화 하고 모든 정보를 RequestProcessor 에게 위임합니다.  

  
모든 정보를 위임 받은 RequestProcessor는 내부적으로 ActionForm 객체를 생성하여 요청 데이터를 검증하고  

  
요청에 대한 Action 객체를 생성또는 호출하여 요청에 대한 작업을 수행합니다.  

  
Action은 요청에 대한 작업을 Model 영역의 자원을 통해 수행하고,   

  
RequestProcessor에게 요청 처리 결과를 넘기게 됩니다.  

  
RequestProcessor는 ActionForward 객체를 통하여 View 단으로 데이터를 넘겨 사용자에게 응답을 합니다.  

  
이 설명에 대한 스트럿츠 내부 요청 처리 절차에 대한 흐름도 입니다.  

  



    스트럿츠 시스템의 사용자 요청의 흐름![](http://localhost/wordpress/wp-content/uploads/1/cfile23.uf.183A8A0E4AD5554E66CEA2.jpg)[그림 3]스트럿츠를 이용한 Web Application 의 요청 처리 절차

    **  

  
장점?  

  
**우선 변화에 대한 장점을 찾아 봅시다.

    

    첫번째로, 모든 요청을 ActionServlet 이 도맡아 처리 합니다.  

  
사실 위와 같은 형식을 이미 패턴으로 정의하여 사용하고 있습니다. 바로 Front Controller 패턴입니다.

    



패턴에 대한 예기를 하게 됩니다만, 이 부분은 이미 패턴으로 정의 된것이기에   

  
Front Controller 패턴의 장점을 집고 넘어가면 될듯합니다.






  * 동일한 제어 로직이 중복되지 않도록 하려는 경우


  * 다중 요청에 일반 로직을 적용해야 하는 경우


  * 뷰에서 시스템 처리 로직을 분리시켜야 하는 경우


  * 제어 접근 지점을 시스템 안으로 중앙 집중해야 하는경우  

  
Front Controller 패턴을 사용하면 위와 같은 효과를 볼수 있습니다.



간단한 예를 하나 들어보겠습니다.  

  
기존의 시스템대로 사용 한다면,   

한글 요청에 대하여 모든 페이지,   

컨트롤러에서 한글 처리를 위한 로직(requset.setCharacterEncoding( ))  

  
이 포함되어야 했지요?   

  
이 작업을 모든 페이지에서 하지 않고   

  
맨 앞단에 중앙 집중적인 코드를 두어 한곳에서 처리 하면 모든 곳에 적용 될수 있도록 하는 것이지요.  

  
실제로 제가 맡아 진행중인 프로젝트인 블랙보드에서는 서버로 들어 오는 모든 요청에 대하여  

  
인증(Authentication) 로직을 수행합니다. 인증된 사용자이면 요청을 처리 하고, 그렇지 않으면  

  
초기 로그인 페이지로 리다이렉트 시켜 버립니다. 모든 요청에 대하여 입니다.  

  
모든 페이지에 if 문으로 확인을 할까요? 아니겠지요.   

  
방금 알게된 Front Controller 패턴을 활용하는 것 일 겁니다.  

  
두번째로 스트럿츠는 서버로 들어 오는 파라미터 처리,   

  
그리고 해당 서블릿(Action)을 호출, 응답 페이지(View)로전달 하는 작업을 객체화 하였습니다.  

  
이는 객체지향에서 예기하는 장점을 갖게 됨을 말합니다.  

  
각각의 작업에 대한 캡슐화를 하기 때문에 스트럿츠 기능 자체 만으로도   

  
내부 적인 로직을 몰라도 원하는 기능을 가져다 사용할 수가 있습니다.  

  
또한 추가적인 기능을 원할 경우 확장을 통한 기능 추가를 통해 유연함을 제공할 수 있고,   

  
같은 처리를 하는 곳에서는 공통된 로직으로 작업을 처리 할 수 있습니다.   

(ActionForm, ActionForward, Action)  

  
위와 같은 객체지향적인 장점을 이용할 수 있기 때문에,   

  
스트럿츠를 사용한 시스템에서 개발자는 Action 객체만 확장 하여,   

  
자신의 로직을 기술 하면, 기본적인 처리가 가능하게 됩니다.  

  
**역할, 구현  

  
**그럼 각각의 영역에 대하여 역할과 어떻게 구현 되어 있는지 간단하게 알아 보겠습니다.  

  
스트럿츠는 각각의 영역마다 하는 업무와 흐름이 명확히 구분이 되기 때문에   

  
흐름대로만 따라가면 분석할수 있도록 잘 구조화 되어있다고 생각합니다.  

  
저와 같이 내부 분석을 하는데 있어 매우 편리한 프레임워크라고 생각합니다.^^  

  
우선 모든 요청은 ActionServlet 으로 갑니다. 그럼 ActionServlet을 확인해 봅시다.




**ActionServlet   

  
**역할 : 서버의 단일 진입점 제공, 스트럿츠 프레임웍 초기화 수행.  

  
ActionServlet은 javax.servlet.http.HttpServlet 클래스를 상속합니다.  

  
이것으로 인해 ActionServlet의 초기화 작업을 유추할 수 있습니다.  

  
컨테이너는 로드 할 때 web.xml 에 기술된 서블릿을 우선 로드 합니다.   

(물론 다른 옵션 사항들이 있지만 넘어가겠습니다.)  

  
기술된 서블릿을 로드 할 때 순서는 다음과 같습니다.  

  
생성자 메서드 init() 메서드, 클라이언트의 요청이 있을 때마다 service() 메서드 doGet() / doPost()  

  
위에 말한대로 ActionServlet은 HttpServlet 클래스를 상속 하였으므로,   

  
컨테이너가 로드 될 때 web.xml에 기록한  

  
ActionServlet 클래스의 생성자 메서드를 호출 하고 init() 메서드를 한번 호출 합니다.  

  
그럼 ActionSerlvet 객체가 초기화가 되겠지요.  

  
( init() 메서드는 일반적으로 객체를 초기화 하는 메서드로 이름 짓습니다. )  

  
실제 코드에서 ActionServlet의 초기화 과정을 확인해 봅시다.


[#M_내용 보기|접기|
<table cellpadding="0" cellspacing="0" border="1" >
<tbody >
<tr >

<td width="711" valign="top" >


**public** **void** init() **throws** ServletException {







initInternal(); //스트럿츠 내부에서 사용하는 메시지들을 초기화




initOther(); //web.xml에서 전달하는 초기화 인자를 초기화




initServlet(); //web.xml의 서블릿 매핑 정보를 초기화







getServletContext().setAttribute(Globals._ACTION_SERVLET_KEY_, **this**);




// 필요한 설정 정보를 Context에 저장







//스트럿츠의 추가 모듈에 대하여 초기화 작업




initModuleConfigFactory();




// Initialize modules as needed




Module moduleConfig = initModuleConfig("", config);




initModuleMessageResources(moduleConfig);




initModuleDataSources(moduleConfig);




initModulePlugIns(moduleConfig);




moduleConfig.freeze();




Config







Enumeration names = getServletConfig().getInitParameterNames();




**while** (names.hasMoreElements()) {




String name = (String) names.nextElement();




**if** (!name.startsWith("config/")) {




**continue**;




}




String prefix = name.substring(6);




moduleConfig = initModuleConfig




(prefix, getServletConfig().getInitParameter(name));




initModuleMessageResources(moduleConfig);




initModuleDataSources(moduleConfig);




initModulePlugIns(moduleConfig);




moduleConfig.freeze();




}







**this**.initModulePrefixes(**this**.getServletContext()); // 추가 모듈역시 Context에 저장







**this**.destroyConfigDigester(); //xml parser 역할을 하는 Digester 객체를 삭제




}



















</td></tr></tbody></table>_M#]


[코드 1] ActionServlet init() 메서드




코스에 포함된 주석과 같이 ActionServlet은 스트럿츠를 초기화 합니다.  

**  

initInternal()**


[#M_내용 보기|접기|
<table cellpadding="0" cellspacing="0" border="1" >
<tbody >
<tr >

<td width="711" valign="top" >


**protected** **void** initInternal() **throws** ServletException {







// :**FIXME**: Document UnavailableException







**try** {




internal = MessageResources._getMessageResources_(internalName);




} **catch** (MissingResourceException e) {




_log_.error("Cannot load internal resources from '" + internalName + "'",




e);




**throw** **new** UnavailableException




("Cannot load internal resources from '" + internalName + "'");




}







}** **










</td></tr></tbody></table>_M#]





스트럿츠 내부에서 사용하는 ActionResources.properties 파일을 초기화 하고, 관리 할수 있는  

  
org.apache.struts.util.PropertyMessageResources 클래스를 객체화 합니다.




PropertyMessageResources 클래스는 org.apache.struts.util.MessageResources 클래스를  

  
구현(implements) 하고, 다형성을 위한 Upcating 하고 있습니다.  

  
이말은 추가할 기능이 있다면 확장하여 구현하고 ActionServlet의 internalName을 새로 구현한 Resourse




객체로 대체 할수 있음을 나타 냅니다.  

**  

  
initOther()**


[#M_내용 보기|접기|
<table cellpadding="0" cellspacing="0" border="1" >
<tbody >
<tr >

<td width="711" valign="top" >


**protected** **void** initOther() **throws** ServletException {







String value = **null**;




value = getServletConfig().getInitParameter("config");




**if** (value != **null**) {




config = value;




}







// Backwards compatibility for form beans of Java wrapper classes




// Set to true for strict Struts 1.0 compatibility




value = getServletConfig().getInitParameter("convertNull");




**if** ("true".equalsIgnoreCase(value)




|| "yes".equalsIgnoreCase(value)




|| "on".equalsIgnoreCase(value)




|| "y".equalsIgnoreCase(value)




|| "1".equalsIgnoreCase(value)) {







convertNull = **true**;




}







**if** (convertNull) {




ConvertUtils._deregister_();




ConvertUtils._register_(**new** BigDecimalConverter(**null**), BigDecimal.**class**);




ConvertUtils._register_(**new** BigIntegerConverter(**null**), BigInteger.**class**);




ConvertUtils._register_(**new** BooleanConverter(**null**), Boolean.**class**);




ConvertUtils._register_(**new** ByteConverter(**null**), Byte.**class**);




ConvertUtils._register_(**new** CharacterConverter(**null**), Character.**class**);




ConvertUtils._register_(**new** DoubleConverter(**null**), Double.**class**);




ConvertUtils._register_(**new** FloatConverter(**null**), Float.**class**);




ConvertUtils._register_(**new** IntegerConverter(**null**), Integer.**class**);




ConvertUtils._register_(**new** LongConverter(**null**), Long.**class**);




ConvertUtils._register_(**new** ShortConverter(**null**), Short.**class**);




}







}** **
















</td></tr></tbody></table>_M#]





web.xml에서 전달하는 Config, convertNull의 initParameter 값을 서블릿에 초기화합니다.  

  
config : 웹 애플리케이션이 설치된 경로를 기준으로 스트럿츠 설정 파일의 위치를 지정해 줍니다.  

  
별도로 지정하지 않으면 "/WEB-INF/struts-config.xml"로 간주합니다. ','로 구분하여 여러 개를 지정할 수 있다.  

  
convertNull : 파라미터로 전달된 값이 없을 때 ActionForm의 java.lang.Integer와 같은 숫자형 래퍼 클래스  

  
(Wrapper Class)의 타입 속성을 지정하는 방법을 지정합니다. "true"로 지정하면 null이 설정될 것이며,   

  
"false"로 설정하면 0이 설정될 것입니다.. 별도로 지정하지 않으면 "false"로 간주하게 됩니다.  

**  

initServlet()**** **


[#M_내용 보기|접기|
<table cellpadding="0" cellspacing="0" border="1" >
<tbody >
<tr >

<td width="711" valign="top" >


**protected** **void** initServlet() **throws** ServletException {







// Remember our _servlet_ name




**this**.servletName = getServletConfig().getServletName();







// Prepare a Digester to scan the web application deployment descriptor




Digester digester = **new** Digester();




digester.push(**this**);




digester.setNamespaceAware(**true**);




digester.setValidating(**false**);







// Register our local copy of the DTDs that we can find




**for** (**int** i = 0; i < registrations.length; i += 2) {




URL url = **this**.getClass().getResource(registrations[i+1]);




**if** (url != **null**) {




digester.register(registrations[i], url.toString());




}




}







// Configure the processing rules that we need




digester.addCallMethod("web-app/servlet-mapping",




"addServletMapping", 2);




digester.addCallParam("web-app/servlet-mapping/servlet-name", 0);




digester.addCallParam("web-app/servlet-mapping/url-pattern", 1);







// Process the web application deployment descriptor




**if** (_log_.isDebugEnabled()) {




_log_.debug("Scanning web.xml for controller servlet mapping");




}







InputStream input =




getServletContext().getResourceAsStream("/WEB-INF/web.xml");







**if** (input == **null**) {




_log_.error(internal.getMessage("configWebXml"));




**throw** **new** ServletException(internal.getMessage("configWebXml"));




}







**try** {




digester.parse(input);







} **catch** (IOException e) {




_log_.error(internal.getMessage("configWebXml"), e);




**throw** **new** ServletException(e);







} **catch** (SAXException e) {




_log_.error(internal.getMessage("configWebXml"), e);




**throw** **new** ServletException(e);







} **finally** {




**try** {




input.close();




} **catch** (IOException e) {




_log_.error(internal.getMessage("configWebXml"), e);




**throw** **new** ServletException(e);




}




}







// Record a _servlet_ context attribute (if appropriate)




**if** (_log_.isDebugEnabled()) {




_log_.debug("Mapping for servlet '" + servletName + "' = '" +




servletMapping + "'");




}







**if** (servletMapping != **null**) {




getServletContext().setAttribute(Globals._SERVLET_KEY_, servletMapping);




}







}











































</td></tr></tbody></table>_M#]


Servlet 이름을 저장하고, web.xml 파일을 로드합니다.  

  
그리고 나서 Digester 클래스를 이용하여 web.xml 파일을 파싱합니다.  

  
실제 parsing 할때 web.xml 에 대한 기초 데이터로 스트럿츠가 초기화 됩니다.  

  
리턴값 없이 이것이 가능한 이유!  

  

digester.push(**this**);  

  

ActionServlet을 밀어넣어 주고 있습니다!!!  

  
객체지향적이지 않습니까?  

  
이것으로 스트럿츠는 Digester를 몰라도 스트럿츠를 초기화 하는데 문제가 없습니다.  

  
이렇게 Digester는 독립적으로 개발이 되었고,   

  
덕분에 Digester는 스트럿츠 내부 라이브러리에서 아파치 프로젝트로 승급 됩니다. 짝짝짝  

  
다음으로 모듈의 키값으로 각 컨텍스트를 저장 합니다.  

**  

this**.initModulePrefixes(**this**.getServletContext());  

  
여기서 문제!  

  
그럼 각각의 모듈 별로 등록한 스트럿츠들은.. 서로 데이터를 공유 할까요?  

  
컨텍스트가 다르므로 되지 않습니다. 하나로 같이 있지만 모듈별로 독립적인 시스템이 되는 것입니다.  

  
여튼 이렇게 초기화 된 스트럿츠를 컨텍스트에 담고, Digester 는 죽습니다.  

  
자 스트럿츠의 초기화는 이렇게 완료 됩니다.  

  
그런데 먼가 부족해보입니다.




스트럿츠는 초기화 하였지만 실제 요청에 대한 작업은 어떻게 처리 할까요?  

  
기억하십니까? 스트럿츠는 서블릿을 상속 받습니다.  

  
스트럿츠는 doGet, doPost를 오버라이딩 합니다.  

  
그말은 요청이 들어 오면   

  
service() 까지는 아버지인 서블릿으로 사용하고 doGet 이나 doPost는 자신의 것을 사용 한다는 말입니다.  

  
아래 소스를 보겠습니다.




  


<table cellpadding="0" cellspacing="0" border="1" >
<tbody >
<tr >

<td width="711" valign="top" >


**public** **void** doGet(HttpServletRequest request,




HttpServletResponse response)




**throws** IOException, ServletException {







process(request, response);







}










</td></tr></tbody></table>
<table cellpadding="0" cellspacing="0" border="1" >
<tbody >
<tr >

<td width="711" valign="top" >


**public** **void** doPost(HttpServletRequest request,




HttpServletResponse response)




**throws** IOException, ServletException {







process(request, response);







}







</td></tr></tbody></table>





아! 모든 요청에 대하여 process 를 호출 하고 있네요~  

  
여담으로 Http 요청에는 doGet, doPost 이외의 것들이 많이 있습니다 Header, debug 등… 이런 요청은 당연히 아버지 서블릿이 수행해 주겠지요  

  
다시 돌아와서 사용자의 요청은 process() 메서드가 처리를 해주는군요.  

  
일단 사용자의 요청 부분은 다음 post 에서 이어 나가겠습니다.  

  
  



_M#]


****







  

**




프로그래머 오성민  

Phone: 010-6358-2651  

E-mail: opsbina@gmail.com  

nateon: if-iamyou@nate.com









































































****
