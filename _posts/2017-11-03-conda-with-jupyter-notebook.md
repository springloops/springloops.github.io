---
author: springloops
comments: true
date: 2017-11-03
layout: post
title: 'Installing Anaconda and Jupyter notebook'
categories:
- Python
tags:
- jupyter
- anaconda
---

# Installing Anaconda and jupyter notebook


## Anaconda


Go [anaconda downloads](https://www.continuum.io/downloads)

* 자신의 환경과 맞는 버전을 설치.
* 기본 conda 환경에 설치된 패키지 조회 

```bash
conda list
```
![conda_list.png](/master/imgs/_for_posts/2017-11-03-conda-with-jupyter-notebook/conda_list.png)

### conda 환경 생성

conda create -n [virtualenv_name] [python=version]

```bash
conda create -n py3 python=3
```


### conda 환경 조회

```bash
conda env list
```

![conda_env_list.png](/master/imgs/_for_posts/2017-11-03-conda-with-jupyter-notebook/conda_env_list.png)


### 가상 환경 접속

source activate [virturalenv_name]

```bash
source activate py3
```

### 가상환경 내 패키지 설치하기

conda install [package]

```bash
conda install jupyter notebook
```

### 가상 환경내 설치된 패키지 확인

```bash
conda env list
```

### 가상 환경 설정 내보내기

공유 등을 위해..

```bash
conda env export > environment.yaml
```

### 환경 설정 파일 (environment.yaml) 기반으로 환경 생성하기

환경 설정 파일 (*.yaml) 의 첫번째 라인에 있는 name 에 정의된 값으로 환경 이름이 결정된다.

```bash
conda env create -f environment.yaml
```

### 가상 환경 나오기

```bash
source deactivate
```

### 가상 환경 삭제하기

conda env remove -n [virtualenv_name]

```bash
conda env remove -n py3
```


jupyter notebook 을 실행한 상태에서 slideshow 만들기

jupyter nbconvert 명령 실행
```bash
jupyter nbconvert --to slides 2017-11-03-conda-with-jupyter-notebook.ipynb  --to slides --post serve
```


## jupyter notebook

오프라인에서 사용가능한 slideshow 만들기

### reveal.js 설치

* 원하는 위치에 다운로드
```bash
git clone https://github.com/hakimel/reveal.js.git
```

* jupyter notebook cell 에 slide 설정


1. View > Cell Toolbar > Slideshow 선택
![menu_slideshow.png](/master/imgs/_for_posts/2017-11-03-conda-with-jupyter-notebook/menu_slideshow.png)

2. Slide Type 설정
![slide_select_box.png](/master/imgs/_for_posts/2017-11-03-conda-with-jupyter-notebook/slide_select_box.png)

3. jupyter nbconvert 명령 실행
```bash
jupyter nbconvert --to slides 2017-11-03-conda-with-jupyter-notebook.ipynb --reveal-prefix=/data/play/reveal.js
```



```python

```
