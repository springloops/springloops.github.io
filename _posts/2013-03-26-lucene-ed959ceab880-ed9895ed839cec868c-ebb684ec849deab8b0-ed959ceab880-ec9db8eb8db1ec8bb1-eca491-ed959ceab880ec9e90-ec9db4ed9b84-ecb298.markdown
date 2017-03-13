---
author: springloops
comments: true
date: 2013-03-26 09:57:50+00:00
layout: post
link: https://springloops.wordpress.com/2013/03/26/lucene-%ed%95%9c%ea%b8%80-%ed%98%95%ed%83%9c%ec%86%8c-%eb%b6%84%ec%84%9d%ea%b8%b0-%ed%95%9c%ea%b8%80-%ec%9d%b8%eb%8d%b1%ec%8b%b1-%ec%a4%91-%ed%95%9c%ea%b8%80%ec%9e%90-%ec%9d%b4%ed%9b%84-%ec%b2%98/
slug: lucene-%ed%95%9c%ea%b8%80-%ed%98%95%ed%83%9c%ec%86%8c-%eb%b6%84%ec%84%9d%ea%b8%b0-%ed%95%9c%ea%b8%80-%ec%9d%b8%eb%8d%b1%ec%8b%b1-%ec%a4%91-%ed%95%9c%ea%b8%80%ec%9e%90-%ec%9d%b4%ed%9b%84-%ec%b2%98
title: '[Lucene] 한글 형태소 분석기 한글 인덱싱 중 한글자 이후 처리 하지 않는 문제에 대하여.'
wordpress_id: 87
categories:
- Open Source/Apache Lucene/Solr
tags:
- 한 글자 형태소 분석
- 한 글자 오류
- 한글자 형태소 분석
- 한글자 오류
- lucene
- solr
---

  
한글 형태소 분석기를 이용해서 테스트 해보다가 보니

인덱싱 중 한글자를 만나면 그 뒤 내용을 처리 하지 않고 끝내버리는 문제가 있네요.

  


관련글을 찾아 보니 이미 부진아님, zetaplus 님께서 처리해주셨더라구요.

부진아 님 글 : [http://cafe.naver.com/korlucene/483](http://cafe.naver.com/korlucene/483)

zetaplus 님 글 : [http://cafe.naver.com/korlucene/487](http://cafe.naver.com/korlucene/483)

  


하지만 그 당시 코드와 현재 cvs에 있는 코드가 다르고,

두분이 처리하신 방법은 zetaplus 님이 말씀하신것 처럼 "두 글자 부터 인덱싱을 한다."

라는 의도에 어긋나게 되더라구요.

  


루씬을 가만 보니 TokenFilter의 incrementToken() 메소드의 true, false 여부로

문서의 끝을 판단하고 있었습니다.

  


TokenFilter 를 상속 받은 KoreanFilter.java 역시 incrementToken() 의 결과로 true, false를 리턴하고 있는데요.

  


KoreanFilter.java 의 incrementToken()메소드 안에서는 한글 분석을 하고, morphQueue에 텀을 넣습니다.

그리고 morphQueue 가 비어 있을 경우에 false를 리턴합니다.

if(morphQueue==null||morphQueue.size()==0) return false;

  


하지만 한글자의 경우 한글을 분석하고 morphQueue에 텀을 채워 주는  analysisKorean(String input) 메소드에서

morphQueue에 텀을 넣지 않고 continue 하고 있습니다.

  


Iterator<String> iter = map.keySet().iterator();

int i=0;

while(iter.hasNext()) {

<blockquote>String text = iter.next();
> 
> if(text.length()<=1) continue;
> 
> morphQueue.add(text);
> 
> </blockquote>

}

  


때문에 incrementToken()메소드 내부 morphQueue 검사에서 false 를 리턴하고

  


그럼 lucene은 false 를 받아, 문서의 끝으로 판단 하는것 같네요.

  


장황하게 썼는데 여기까지는 zetaplus 님과 부진아 님의 말씀과 같네요;;

두분은 저 형광색 부분을 주석 처리 하는 것으로 처리를 해주셨는데,

저는 다르게 생각해봤네요..

  


왜 형태소 분석이 완료된 단어를 가지고 문서의 끝을 판단하는가 였습니다.

실제로 KoreanTokenizer.java 파일의 incrementToken() 에서

KoreanTokenizerImpl의 getNextToken()을 이용해서 문서의 끝을 판단하고 있더라구요..

  


문서에서 어절을 가져오는 스택의 순서입니다.

StopFilter --> LowerCaseFilter --> KoreanFilter  --> KoreanTokenizer

모두 각자의 incrementToken()을 가지고 있구요. 

스택의 순서대로 다음의 incrementToken()을 호출 하고 있습니다.

  


실제 문서에서 하나의 어절을 가져오는 Class 가 KoreanTokenizer이고, KoreanFilter는 분석을 해주는 아이인거 같아요.

  


그러니까 문장 끝의 판단을 KoreanFilter의 morphQueue의 길이로 하지 않고,

KoreanTokenizer에서 어절을 가져올 때 판단하는게 더 유리할것 같더라구요.

  


해서 KoreanFilter.java 의 incrementToken() 메소드의

if(morphQueue==null||morphQueue.size()==0) return false; 구문을 주석처리 하고,

  


setTermBufferByQueue() 메소드에서 morphQueue 의 길이를 판단 하여 term을 채우는 방식으로 변경 하였습니다.

  


[](http://localhost/wordpress/wp-content/uploads/1/cfile23.uf.0121EA3F515170FA262EF7.java)cfile23.uf.0121EA3F515170FA262EF7.java

  


/**

* queue 에 저장된 값으로 buffer의 값을 복사한다.

*/

private void setTermBufferByQueue() {

clearAttributes();

String term = null;

if(morphQueue!=null&&morphQueue.size()!=0) {

term = morphQueue.removeFirst();

System.out.println(term);

int pos = new String(curTermBuffer).indexOf(term);

termAtt.copyBuffer(term.toCharArray(), 0, term.length());

}

}
