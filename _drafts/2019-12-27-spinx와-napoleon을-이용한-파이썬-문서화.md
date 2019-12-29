---
layout: post
title:  "Sphinx와 Napoleon을 이용한 파이썬 문서화"
date:   2019-12-27 16:41:00 +0900
categories: 
tags:
    - spinx
    - 파이썬
    - 문서화
image: https://user-images.githubusercontent.com/8157830/71508129-2a83b180-28ca-11ea-8031-39085cf285e9.png
---

# 문서화는 왜 필요할까
프로그래밍을 처음 배울때 주석이 왜 필요한지는 귀에 딱지가 질 정도로 많이 들었을 것이라고 생각한다.

지금까지 코딩을 해오면서 느낀 개발자들이 공유하는 생각은 주석이 없이 코드로 설명할 수 있는 코드가 좋은 코드라는 점이다.
앞 뒤 말이 모순된다고 생각된다면 뒷 문장을 잘못 이해하고 있을 가능성이 높다고 생각한다.
문장 그대로 코드에 주석이 없어야 좋은 코드라는게 아닌, 주석 없이 코드의 구성 요소(예를 들어 변수명)들만으로도 해당 루틴의 역할을 알 수 있어야 한다라는 뜻이다.

참고: <a href="http://zenr.me/3066" target="_blank">주석이 필요 없는 코드.key</a>

&lt;<a href="http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9788966261031" target="_blank">실용주의 프로그래머</a>&gt;에서도 아래와 같이 말하고 있다. <small>내용과는 별개로 정말 좋은책이니까 한번씩 읽어보면 좋을 것 같다.</small>

> TODO: Write here

주석은 하나하나의 "나무"를 설명하면 안되고 "숲"을 설명할 수 있어야 한다. 항상 생각하고는 있지만 "숲"을 잘 설명하는건 정말 어려운 일 같다.

본론으로 돌아와서 이런 주석을 달 수 있도록 도와주는 도구가 문서화라고 생각한다. 문서화는 작게는 함수, 클래스 크게는 모듈까지 포함해서 왜 필요한 코드이고 어떻게 사용해서 어떤 식으로 동작하는지를 보여줄 수 있다.

# 문서화 방법
## 직접 매뉴얼 작성
가장 기본적이고 원시적인 방법이다.

형식의 제약이 없고 모든 것이 자유롭다. 다만 처음부터 끝까지 모든 과정을 직접 만들어야한다.
결과물은 txt가 될 수도 있고, pdf가 될 수도 있고, html이 될 수도 있다.

생성 과정의 자동화가 되어있지 않아서 코드가 변경될 때 마다 변경점을 직접 찾아 수정해야하고, 코드와 연결되어있지 않아서 변경된 지점을 찾기 쉽지 않다. 변경 사항이 바로 반영되지 않는다면 쓸모없는 문서가 될 것이다.

이 방법은 별로 추천하고싶지 않다. 가능하다면 아래에 나와있는 docstring과 sphinx와 같은 도구를 이용해서 문서화를 자동으로 할 수 있도록 하자.

## Docstring 이용
코드 상단에 따옴표 세 개와 함께 작성하는 문자열이다. 이 문자열에 왜 이 함수(또는 클래스, 모듈)가 필요한지, 어떤 값을 받아서 어떤 결과를 내는지, 함수를 이해하기 위한 기본 지식 등을 적을 수 있다.

예를 들어, 아래와 같이 반올림을 하는 함수가 있다고 하자.

```python
def ceil(a):
    if a - int(a) < 0.5:
        result = int(a)
    else:
        result = int(a) + 1
    return result
```

간단한 함수라 바로 이해가 되겠지만 docstring을 이용하면 코드를 읽는 사람이 좀 더 쉽게 이해할 수 있다.

```python
def ceil(a):
    """a의 소수부를 확인해서 반올림

    매개변수 a는 double type value.
    a의 소수부가 0.5보다 작으면 내림, 크면 올림 연산을 수행.

    a를 반올림한 값을 결과로 반환
    """
    if a - int(a) < 0.5:
        result = int(a)
    else:
        result = int(a) + 1
    return result
```

하지만 이렇게 마구잡이로 docstring을 적으면 자동으로 문서를 생성할 수 없다. 그래서 몇개의 style들이 존재하고 우리는 이 docstring style에 맞춰서 작성하면 된다.

## Docstring 스타일 종류
### Sphinx Style
docstring은 아래와 같은 모양을 가진다.
```python
"""[Summary]

:param [ParamName]: [ParamDescription], defaults to [DefaultParamVal]
:type [ParamName]: [ParamType](, optional)
...
:raises [ErrorType]: [ErrorDescription]
...
:return: [ReturnDescription]
:rtype: [ReturnType]
"""
```
<small>ref: <a href="https://sphinx-rtd-tutorial.readthedocs.io/en/latest/docstrings.html" target="_blank">Spinx docs</a></small>

위에서 작성했었던 `cail()` 함수의 docstring을 새로 작성해보자.

```python
def ceil(a):
    """a의 소수부를 확인해서 반올림

    :param a: 반을림 할 실수
    :type a: double
    :return: a를 반올림한 값을 결과로 반환
    :rtype: int
    """
    if a - int(a) < 0.5:
        result = int(a)
    else:
        result = int(a) + 1
    return result
```
매개변수와 반환값 각각의 type과 의미를 한눈에 확인 할 수 있다.

매개변수가 두 개 이상이라면 아래와 같이 작성하면 된다.
```python
def add(a, b):
    """a와 b를 더한다
    
    :param a: 피연산자 1
    :type a: int
    :param b: 피연산자 2
    :type b: int
    :return: 피연산자 a와 b를 더한 값
    :rtype: int
    """
    return a + b
```

### Numpy Style
### Google Style
autoDocstring -- vscode google style docstring prefix generator

어떤 style을 사용해도 무방하나 이 글에서는 비교적 읽기 쉬운 Google Style로 계속 진행을 하려고 한다.


# Sphinx를 이용한 docs 자동 생성

# Napoleon을 이용한 Google Style Docstring 자동 생성


