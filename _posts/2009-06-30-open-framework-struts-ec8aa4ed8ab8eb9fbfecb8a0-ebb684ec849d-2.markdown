---
author: springloops
comments: true
date: 2009-06-30 04:08:00+00:00
layout: post
link: https://springloops.wordpress.com/2009/06/30/open-framework-struts-%ec%8a%a4%ed%8a%b8%eb%9f%bf%ec%b8%a0-%eb%b6%84%ec%84%9d-2/
slug: open-framework-struts-%ec%8a%a4%ed%8a%b8%eb%9f%bf%ec%b8%a0-%eb%b6%84%ec%84%9d-2
title: 'Open FrameWork - Struts : 스트럿츠 분석 (2)'
wordpress_id: 7
categories:
- Open Source
tags:
- 분석
- framework
- 스트럿츠
- Java
- Struts
---

  




**스트럿츠 분석(2)  

  
****  

**




**이어서…  

**  

  
지난 시간에 ActionServlet 의 process() 메소드에 대해서 설명 하려다 접었습니다.  

  
실제 사용자의 요청에 대한 처리 부분이였지요~  

  
머 좀 쉬어 가는 뜻도 있고, process 메소드는 RequestProcessor 객체에 요청(request)를 위임하여 실제 처리를  

  

RequestProcessor 클래스가 하고 있기에, ActionServlet과 구분 짓기 위해서이기도 했습니다.  

  

그럼 사용자의 처리가 어떻게 이뤄지는지 같이 봅시다~  

  



[#M_내용 보기|접기|


**Process()   

**  

ActionServlet은 servlet을 상속 받고 doGet, doPost를 오버라이딩 하고 있습니다.  

  
doGet, doPost 메소드는 모두 process() 을 호출하며, request, response 를 넘깁니다.  

  
요청에대한 모든 작업을 process() 에게 넘긴다고 보시면 되겠네요~


<table cellpadding="0" cellspacing="0" border="1" >
<tbody >
<tr >

<td width="678" valign="top" >


**protected** **void** process(HttpServletRequest request, HttpServletResponse response)




**throws** IOException, ServletException {







//내부적으로 초기화한 Context를 찾아 request의 속성(setAttribute()) 로 저장합니다.




ModuleUtils._getInstance_().selectModule(request, getServletContext());




ModuleConfig config = getModuleConfig(request);







RequestProcessor processor = getProcessorForModule(config);




**if** (processor == **null**) {




processor = getRequestProcessor(config);




}




processor.process(request, response);







}










</td></tr></tbody></table>





ModuleUtils._getInstance_().selectModule(request, getServletContext()); 는 내부적으로   

  
web.xml 에 기술 되어 있는 스트럿츠 정의 문서의 키값을 찾아 context에서 해당 모듈 config의 키를 찾아  

  
request의 속성에 저장 합니다.  

  
ModuleConfig config = getModuleConfig(request);  

  
위에서 저장한 request를 바탕으로 ModuleConfig를 얻어 옵니다.  

  
ModuleConfg 는 스트럿츠 정의 문서에 기술되어 있는 내용을 객체화 하여 담고 있는 자원으로 해당  

  
스트럿츠 모듈의 정보를 가지고 있다고 보시면 됩니다.  

  
자 이제 말씀드린대로 스트럿츠 모듈의 모든 정보를 RequestProcessor 객체로 위임을 합니다.  

  
RequestProcessor processor = getProcessorForModule(config);  

  
이때 먼저 생성한 스트럿츠의 정보 config 객체를 넘깁니다.  

  
내부적으로 받은 config 객체를 가지고 RequestProcessor 객체를 초기화 합니다.  

  
초기화 작업은 RequestUtils 객체를 활용하여 RequestProcessor 객체를 가져오고 난 다음  

  
RequestProcessor 의 init() 메서드를 호출 합니다.  

  
init() 메서드는 해당 ActionServlet 과 ModulConfig 를 받아 자신의 객체 변수로 등록을 합니다.  

  
이로써 RequestProcessor 객체는 스트럿츠의 정보를 가지게 되고,  

  
processor.process(request, response);  

  
자신의 process 메서드를 호출할 때 사용자 요청을 넘김으로서 모든 사용자 요청을 처리 할수 있게 되는 것이지요.




내용이 좀 어려워 지는데 능력이 부족하여… 나중에 더 업뎃을 하겠습니다.  

  
자 그럼 RequestProcessor 객체의 다른 기능에 대하여 알아 봅시다.  

  
현재까지 ActionServlet을 초기화 하고 사용자의 요청을 통하여 RequestProcessor 객체를 초기화 하고  

  
process 메서드까지 호출을 하였습니다.  

  
그럼 process 메서드에서 어떤 작업을 하는지 확인하고, 각각의 기능에 대해서 설명하도록 하지요.  

  
거의 마무리가 되어 갑니다 힘들 내봅시다~^^  

  
process 메서드에서는 내부적으로 다음과 같은 작업을 합니다.  

  
processMultipart() 메서드를 호출한다.  

  

제목과 같이 애플리케이션에서 다중 파일 업로딩 기능을 지원한다면 새로운 요청 래퍼(Request Wrapper)를 만든다.






  1. processPath() 는 요청URI에서 경로를 추출하여 반환한다. 이정보는 호출 할 스트럿츠 Action을 선택하는데 사용합니다.


  2. processLocale() 요청의 Locale을 결정합니다. 그리고 생성된 Locale 객체를 사용자의 HttpSession에 저장하여 사용하게 됩니다.


  3. processContent() 메소드는 요청의 컨텐츠 타입과 인코딩을 결정합니다.


  4. processNoCache() 는 nochace 속성이 true 로 되어 있는지 확인하여 true 로 되어 있으면 브라우저에 페이작 캐시되는 것을 막기 위해 응답 객체에 적절한 헤더 파라미터를 추가 합니다.


  5. processPreprocess() 후크 메소드로서 비어 있는 메소드라고 보시면 됩니다. 마지막에 다시 설명 하겠습니다.


  6. processmapping() 요청에서 사용할 ActionMapping 객체를 생성합니다.  





ActionMapping 객체는 struts-config.xml에 기술된 내용을 가지고 있습니다.  

  
processRoles() 사용자가 요청한 요청을 처리 할수 있는 규칙이 있는지 여부를 판별하고, 없다면 에러를 일으키며 처리 과정을 중단 합니다.  

  
processActionForm() Actionmapping에 ActionForm이 설정이 되어 있는지 확인합니다.




ActionForm 이 설정 되었다면 ActionForm 객체를 생성하거나 찾아 옵니다.




나중에 ActionForm 초기화 부분에서 다시 한번 거론 될것입니다.  

  
  








  1. processPopulate() ActionForm이 설정이 되었다면 ActionForm으로 요청 파라미터 값을 저장합니다.


  2. processValidate() 메서드는 스트럿츠 정의 기술 파일에 (Struts-config.xml) 파일에 해당 Action 엘리먼트에 validate 속성이 true 로 되어 있을 경우 호출하게 됩니다.


  3. 다음으로 forward, include 메소드를 호출하고,


  4. processActionCreate() 메소드를 호출 합니다



이 메서드는 요청을 처리 하기위한 Action을 생성합니다.  

  
processActionPerform() 메서드는 앞서 생성한 Action 의 execute() 메소드를 호출 합니다.  

  

맨 처음에 스트럿츠를 사용하면 기본적으로 Action 객체만 상혹하여 execute() 메소드안에 요청을 위한 로직을 기술 하기만 하면 된다고 한 점을 기억하시나요? 바로 이부분에서 구현한 execute를 실행하여 요청을 처리 합니다.  

  
processActionForward() 메소드는 Action 객체의 execute 메서드를 실행한 결과반환된 ActionForward에 해당하는 URL 로 포워드를 합니다.




이제 개발자는 Action 을 상속 하여 개발 하면 되겠습니다.  

  
오픈 프레임웍을 분석하는 것보다 글로 작성하는 것이 쉽지 않다는 것을 알게 됬네요.  

  
부족함에 시중의 교재중 일부를 발췌 한 것도 있구요, 도저히 설명이 되지 않아 빼버린 부분도 있고,  

  
기능들중 설명 안한 부분도 있네요.  

  
다음에 기회가 되면 업로드 하겠습니다.  

  
여튼 이렇게 해서 기본적인 스트럿츠의 초기화 작업과 작동 원리에 대해서 감을 잡으셨길 바라며…  

  
코드가 깔끔하니 한번쯤 소스를 돌려 보시는 것을 추천 하면서..  

  
물러갑니다.  

  
  



_M#]


  






프로그래머 오성민  

Phone: 010-6358-2651  

E-mail: opsbina@gmail.com  

nateon: if-iamyou@nate.com






