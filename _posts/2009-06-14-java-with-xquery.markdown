---
author: springloops
comments: true
date: 2009-06-14 02:50:00+00:00
layout: post
link: https://springloops.wordpress.com/2009/06/14/java-whit-xquery/
slug: java-with-xquery
title: JAVA with XQuery
wordpress_id: 65
categories:
- Tip
tags:
- Java
- oracle
- xdk
- XQuery
---

  
이번에 Blackboard Academic suite Learn 9 버전의 Test/Survey 기능을 응용하여
추가 Buildingblock 을 개발하게 됬는데,  

Test/Survey 데이터 타입이 clob 이고 실 데이터가 XML 형식으로 등록 되어있었다.

검색결과 XQuery 라는 문법을 알게 되었고, 기본적인 사항을 기술 해보았다.  

XQuery!! 이건 W3C 표준이구요.  

  
오라클에서는 9i 부터 적용이 된 듯 합니다.  

![](http://somethingbook.wordpress.com/wp-includes/js/tinymce/plugins/wordpress/img/trans.gif)  

  
Blackboard Learn 9는 10g 를 사용 하고 있고, 쿼리 자체는 다를게 없지만  
  
Java 에서 해당 DB에 접속하여 트랜잭션을 수행하기 위해서는 jdbc 로 이외에 추가 적인 jar(lib) 파일이 필요합니다.  
  
관련 jar 파일은 9i 용과 10g 용이 따로 있습니다.  
  
XQuery를 활용한 쿼리는 다음과 같습니다.  

```sql
SELECT t.column_value
  FROM acc_comm_log a, 
       xmltable ('for $root in $date  
			   return $root/item_result/response/response_value/text()'
              passing a.comm_details as "date" ) t
  WHERE a.ACC_NO = 4
```
위에 쿼리에 대한 설명을 하겠습니다.  
  
우선 xmltable 이라고 정의 한 부분이 있습니다.  
  
오라클에서 xquery 와 xmltable 두가지를 지원 합니다.  
  
xquery 와 xmltable 두가지 문법은 같고, xquery 는 중간중간에 태그를 넣어서 xml 형태로 출력을 할수 있고,  
  
xmltable 은 관계형데이터베이스 형식으로 출력을 합니다.  
  
저는 데이터가 필요 하므로, xmltable 함수를 사용했구요.  
  
xquery 문법 (w3c 표준이므로 xquery 라고 부르겠습니다.) 은 다음과 같습니다.  
  
* for 루프는 기존 루프와 동일합니다.  
* let 절은 노드에서 가져온 값을 할당 합니다. (저는 사용안했습니다)  
* order by는 결과를 정렬합니다.
* return은 각 결과 값을 리턴 합니다.
* passing 은 별명 지정과 같구요.
* xmltable( ... ) t 이것은 t 라는 이름으로 쿼리에서 접근하여 사용합니다. (t 는 물론 아무 값이나 사용 가능합니다.)
  
return 문에 보면  `return $root/item_result/response/response_value/text()` 라고 되어 있는데요  
  
이것은 xml 노드들을 나열 한것입니다.  
  
xml 보시면 root 에서 item_resut 태그 자식에 response 자식에 response_value 노드의   
  
text() 로 값을 가져 오고있습니다.  
  
결국 4 를 가져 오는 것입니다.  
  
그 값을 쿼리에서 t 라는 별명을 이용해서 t.column_value 함으로써 4 라는 값을 적용 하게 됩니다.  
  
xquery 에 대해서는 이게 전부구요, 물론 더 많은 기능들이 있겠습니다만... 정말 간략하게 알아보고 정리합니다.  
  
더 자세한건 [http://www.w3schools.com/](http://www.w3schools.com/) 여기가시면 예제도 있고, 쉽게 설명되어 있습니다.  

  
여긴 오라클 [http://www.oracle.com/technology/global/kr/oramag/oracle/05-sep/o55xquery.html](http://www.oracle.com/technology/global/kr/oramag/oracle/05-sep/o55xquery.html)  

아래 xml 첨부 하구요.  
  
```xml
<xml version="1.0" encoding="UTF-8" ?>  
<item_result presented="No">  
    <result_metadata>  
        <bbmd_asi_object_id>_8500_1</bbmd_asi_object_id>  
        <bbmd_result_object_id>_916_1</bbmd_result_object_id>  
        <bbmd_resulttype>Item</bbmd_resulttype>  
    </result_metadata>  
    <date>  
        <type_label>항목 생성시간.</type_label>  
        <datetime>2009:04:27T14:06:04</datetime>  
    </date>  
    <response ident_ref="response">  
        <response_form cardinality="Single" render_type="choice" timing="No" response_type="lid" />
        <response_value response_status="Valid" response_time="-1">4</response_value>  
    </response>
    <outcomes status="Completed">  
        <score varname="AUTOGRADE" vartype="Decimal" status="Valid">  
            <score_value />
            <score_maximum />
        </score>
        <score varname="MANUALGRADE" vartype="Decimal" status="Valid">  
            <score_value />  
        </score>  
    </outcomes>  
</item_result>
```

JDBC 에서 위 쿼리에 대한 값을 가져 올때 일반적인 java.sql.ResultSet 으로는 계속 null 이 넘어 오더군요..ㅡㅡ;  

JDBC 를 이용하여 해당 쿼리 수행 방법 정말 간단히 알려 드립니다.  

오라클 xquery 사용 방법  
  
[http://download.oracle.com/docs/cd/B19306_01/appdev.102/b14259/xdb_xquery.htm#CBAEEJDE](http://download.oracle.com/docs/cd/B19306_01/appdev.102/b14259/xdb_xquery.htm#CBAEEJDE)  

여기 가면 잘정리 되어있는데요...    
약간만 수정해서 만든 소스 붙여 드립니다.

```java

import java.sql.*;  
import oracle.sql.*;  
import oracle.xdb.XMLType;  
import oracle.jdbc.driver.*;

public class Test {  

    public static void main(String[] args) throws Exception, SQLException {  
        System.out.println("*** JDBC Access of XQuery using Bind Variables ***");  

        DriverManager.registerDriver(new oracle.jdbc.driver.OracleDriver());  
        OracleConnection conn = (OracleConnection)DriverManager.getConnection(
        "jdbc:oracle:oci8:@localhost:1521:xxxx", 
        "ssss", 
        "yyyy");  

        StringBuilder sb = new StringBuilder();  
        sb.append("select t.column_value ")  
        .append("from acc_comm_log a, ")  
        .append("xmltable ( ")  
        .append("'for $root in $date ")  
        .append("return $root/item_result/response/response_value/text()' ")  
        .append("passing a.comm_details as "date" ")  
        .append(") t ")  
        .append("where a.ACC_NO = 4");

        OraclePreparedStatement stmt = (OraclePreparedStatement)conn.prepareStatement(sb.toString());  

        ResultSet rs = stmt.executeQuery();  

        while (rs.next()) {  
            XMLType desc = (XMLType) rs.getObject(1);  
            System.out.println("LineItem Description: " + desc.getStringVal());  
        }  
        
        rs.close();  
        stmt.close();
	}  
}

```

중요한건...

**1. Connection**

```java

OracleConnection conn =(OracleConnection)DriverManager.getConnection("jdbc:oracle:oci8:@localhost:1521:yyyy", "xxxx", "zzzz");

```

connection 가져올때 OracleConnection 을 이용 해야 하고, connection url 을 thin 으로 하면 `Exception in thread "main" java.sql.SQLException: ORA-22275: 부적합한 LOB 위치 지정자가 지정됨` 이런 exception 이 발생합니다. oci8로 해야만 수행이 되더라구요.  

네이버 검색시 thin으로 수정해서 했다는 사람 있었는데.. 거기까진 잘모르겠습니다...  

**2. 쿼리 안의 " 마크**

XQuery 수행시 사용 하는 " 는 필수로 필요 합니다.  로 감싸주시는것.. 아실테고요..  

(기본 sql 작성시 ' 생략 하는 것과 다릅니다.)  
  
**3. OraclePreparedStatement 사용.**

사실 기본적인 PreparedStatement는 해보지 않았습니다. 쉽게 가려고..쓰라는거 썼구요...ㅋ  

**4. XMLType**

```java

XMLType desc = (XMLType) rs.getObject(1);
System.out.println("LineItem Description: " + desc.getStringVal());
```

위에 말씀드린데로 기본적인 rs.getString(1); 하면 null 이 리턴 됩니다.

XMLType 으로 받아서 getStringVal() 하셔야 합니다.
(데이터 타입에 따라 달라 지겠죠.. api 자세히 안봤지만.. 있을겁니다.)

마지막으로 저거 실행하는데 필요한 lib 알려 드리겠습니다.

**ojdbc14.jar (기본 오라클 jdbc 드라이버)**

xdb.jar (아... 알아보지 못했습니다.. 하지만 압축 풀어보면.. XMLType 객체를 사용 하고 도와 주는 패키지 인듯 합니다.)

xmlparserv2.jar 이거 중요한데요.. 이게 9i 용에는 없는 듯 합니다.

컴파일 시에는 오류 없이 진행하는데 런타임시에 xmlparserv2.jar 가 없으면,
`Exception in thread "main" java.lang.NoClassDefFoundError: oracle/xml/parser/v2/XMLParseException1`

이런 exception 발생합니다.

오라클에서 XDK 라는 패키지를 제공합니다 9i 용 10g 용 있습니다.  
[http://www.oracle.com/technology/tech/xml/xdk/xdk_java.html](http://www.oracle.com/technology/tech/xml/xdk/xdk_java.html) 여기입니다.  

10g 에서도 os 별로 있습니다.
(윈도우에서 개발해서 리눅스로 올리는 상황에서 참... 벌써 이런 상황이 두번째라...)

그 안에 보시면 말씀 드린 jar 파일들있습니다.

추가 하시면 되겠습니다.

급한 마음에 대충 알아 봐서 좀 두서도 없었습니다.

이미 알고 있던 것이라도... 여유롭게 봐주시고요..

긴글 읽어 주셨다면 감사합니다.ㅎㅎ

아참..

XQuery 는 Clob 과 XMLType 두가지 데이터 타입에서 지원합니다.

MSSQL 에서는 XML 이라는 데이터 타입이 있더군요.

저는 xml 값이 들어간 컬럼이 Blob 타입이라.. 참 한번에 되는게 없습니다만..

참고 하세요~