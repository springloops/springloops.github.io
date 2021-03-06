---
author: springloops
comments: true
date: 2017-11-04
layout: post
title: '[발번역] Bag of words (BoW) - Natural Language processing'
categories:
- Data Mining
tags:
- Bag of Words
- BoW
- tf-idf
- hashing trick
---

# Bag of words (BoW) - Natural Language processing

Bag of Word 관련글 읽으면서 작성. 

[원문 - https://ongspxm.github.io/blog/2014/12/bag-of-words-natural-language-processing/](https://ongspxm.github.io/blog/2014/12/bag-of-words-natural-language-processing/)


## 개요
Bag of Words (Bow) 는 자연어 처리에서 사용되는 모델이다. Bow 의 목적은 문서를 분류하는 것이다. 아이디어는 다른 "bags of words" (단어 모음/ 코퍼스) 를 분석하고 분류하는 것이다. 그리고 다른 카테고리를 일치 시침으로써, 우리는 텍스트의 특정 블록이 나오는 "bag" 을 식별한다.

## 문맥에 넣기
이것을 설명하기 위한 한가지 훌륭한 방법은 이 모델을 컨텐츠 안으로 넣는것이다. BoW의 한가지 고전적 용도는 스팸필터다. BoW 모델의 사용을 통해서, 시스템은 스팸과 햄을 구분할 수 있도록 훈련된다. 은유를 확장하기 위해, 우리는 "bag of spam" 또는 "bag of ham" 어디에서 문서가 오는지 추측한다.

**Note**: 나는 스팸 필터가 어떻게 동작하는지에 대해 논리으로 설명하지 않는다. 다른 텍스트 분류의 이론을 이해할 수 있도록 예제만 제공한다.

## BoW 동작 원리

### 벡터 형성
예제를 위해 두개의 텍스트 샘플을 가지고 오자: `The quick brown fox jumps over the lazy dog` and `Never jump over the lazy dog quickly`

그 코퍼스(텍스트 샘플) 로 사전을 만든다:
```
{
    'brown': 0,
    'dog': 1,
    'fox': 2,
    'jump': 3,
    'jumps': 4,
    'lazy': 5,
    'never': 6,
    'over': 7,
    'quick': 8,
    'quickly': 9,
    'the': 10,
}
```
그 다음 각 단어의 수를 나타내기 위한 벡터가 만들어 진다. 이 경우, 각 텍스트 샘플(즉, 문장)은 10개 요소를 갖는 벡터를 생성한다.
```
[1,1,1,0,1,1,0,1,1,0,2]
[0,1,0,1,0,1,1,1,0,1,1]
```
각 요소는 각 단어가 코퍼스(텍스트 샘플) 안에서 발생한 횟수를 표현한다. 그래서, 첫번째 문장에서, "brown" = 1, "dog" = 1, "fox" = 1 등이된다. (첫번째 배열로 표현된). 반면, 두번째 벡터는 "brown" = 0, "dog" = 1, "fox" = 0 로 등으로 나타난다.

### 단어의 가중치 : tf-idf
대부분의 언어에서, 어떤 단어는 다른 단어 보다 더 자주 나타나는 경향이 있다. "is", "the", "a" 와 같은 단어들은 영어에서 아주 흔한 단어들이다. 만약 원시적(날, 생짜) 빈도를 고려한다면, 우리는 서로 다른 클래스의 문서를 효과적으로 구별하지 못할 수도 있다.

이에 대한 일반적인 해결법은 tf-idf 라는 통계적 방법을 사용하여 텍스트 샘플의 문맥을 보다 잘 반영하여 데이터를 더 정확하게 만드는 것이다. term frequency-inverse document frequency의 약어 TF-IDF는 2가지 값을 고려한다: **term frequency**(tf) 와 **inverse document frequency**(idf).

이러한 값을 경정하기 위해 몇가지 방법이 있지만, **term frequency**(용어 빈도)의 값을 결정하는 일반적인 방법은 기본적으로 용어 빈도를 문서의 모든 용어의 최대 빈도로 나눈 값이다.
> 문서의 길이에 때라서 문서별 단어의 빈도차이가 매우 커질 수 있어 정규화 작업을 하는데, 이 블로그에서는 double normalization 을 사용한다. (0.5~1.0 스케일)

```
0.5 + 0.5 * freq(term in document) / max(freq(all word in document))
```

**inverse document frequency**(역 문서 빈도)를 결정하는 한 가지 일반적인 방법은 용어를 포함하는 문서의 비율의 역(inverse) 로그를 취하는 것이다.
```
log( document_count/len(documents containing term) )
```

그리고 두 값을 곱함으로써, 우리는 매직 값, term frequency-inverse document frequency (tf-idf)를 얻음으로써 다른 문서에서 사용되는 일반적인 단어의 가치를 감소시킨다.

값을 얻은 다음 추가적인 단계로는 tf-idf로 벡터가 벡터를 정규화 하는것으로 , 다른 연산자를 적용하는데 덜 성가실것이다.


### 더 나아가: 피처 해싱 (Feature hasing) / 해싱 트릭 (Hashing trick)
여기에 설명된 기본 개념은 사전 크기가 다소 작은 텍스트의 작은 샘플을 위해 작동할 수 있다. 그러나 실제 훈련 과정에는 수만 개의 고유 단어가 포함된 텍스트가 있고, 우리는 문서를 더 효과적으로 표현하기 위한 방법이 필요하다.

용어를 해싱함으로써, 우리는 생성된 벡터의 요소에 일치하는 인덱스를 얻는다. 따라서 사전안에 단어를 저장하고 각 문서를 위한 10만개 요소의 긴 벡터를 갖는 대신, N 개 짜리 벡터를 사용한다. (N은 사용자가 결정한다.)

해시 충돌을 해결하기 위해, 추가 해시 함수가 연산자 선택 함수로 구현될 것이다. ( 0 또는 1 리턴)  이렇게 하면 서로 다른 항목이 충돌할때, 충돌한 항목들은 서로를 취소할 것이고, 각 요소에 대해 기대 값 0 을 준다. ([More on that here](https://en.wikipedia.org/wiki/Feature_hashing#Properties)) 그런다음 문서의 최종 벡터가 여러 유형의 문서를 분류 하는데 사용된다. (예: 스팸 VS 햄)

함수 예시:
```
function hashing_vectorizer(features: array of strings, N: integer):
    x = new Vector[N]
    for f in features:
        h = hash(f)
        ### sign operator
        if hash2(f):
            x[h%N] += 1
        else:
            x[h%N] -= 1
    return x
```

## 결론
BoW가 완성된 후에는 각 개별 문서의 벡터가 된다. 그런 다음 이 문서는 서로 다른 기계 학습 알고리즘을 통과하여 서로 다른 문서를 구분하는 기능을 결정한다.

BoW는 텍스트 문서의 기능 및 내용을 설명하는 벡터로 변환하는 도구 일 뿐이다.

https://spark.apache.org/docs/latest/mllib-feature-extraction.html

## Readings:
* http://deeplearning4j.org/bagofwords-tf-idf.html
* http://en.wikipedia.org/wiki/Tf%E2%80%93idf
* http://en.wikipedia.org/wiki/Bag-of-words_model
* http://en.wikipedia.org/wiki/Feature_hashing

번역하며 참고함 :
* https://spark.apache.org/docs/latest/mllib-feature-extraction.html#tf-idf
* https://www.3pillarglobal.com/insights/topic-clusters-tf-idf-vectorization-using-apache-spark
* https://en.wikipedia.org/wiki/Tf%E2%80%93idf#Term_frequency_2)
* http://preshing.com/20110504/hash-collision-probabilities/
* https://datascience.stackexchange.com/questions/22250/what-is-the-difference-between-a-hashing-vectorizer-and-a-tfidf-vectorizer/22258

