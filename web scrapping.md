# Web Scrapping

> 엄격히 말하면 웹 크롤링은 여러 페이지를 active하게 옮겨다니며 데이터를 수집하는 행위를 말하기 때문에   
> 단일 페이지에서 데이터를 수집하는 웹 스크래핑이라는 용어를 사용한다

우리가 흔히 웹 브라우저를 통해 보는 페이지는 HTML(Hyper Text Markup Language)로 작성되어있다.

과거에는 하나의 HTML 파일 안에 디자인, 데이터, 구조 모든 것을 담았으나

최근의 웹 표준은 이들의 기능을 엄격히 분리하는 것을 추구하고 있다.

문제는 API를 통한 데이터 호출과 달리 웹 페이지에서 우리가 원하는 데이터를 추출하는 것은 굉장히 귀찮은 일이라는 것이다.

웹 스크래핑이 어려운 이유를 예로 들자면 다음과 같다.

볼링 동호회 회원의 이름과 최고 점수를 기록한다고 생각해보자.

- [기록원 1 - 나나]
> - '김보성'회원의 최고 점수는 120점이다.
> - '정명석'회원의 최고 점수는 150점이다.
> - '정상현'회원의 최고 점수는 180점이다.

다른 기록원 레이나는 다음과 같이 기록했다.

- [기록원 2 - 레이나]
> - 김보성: 120점
> - 정명석: 150점
> - 정상현: 180점

이 두 기록에서 이름과 점수를 뽑아내려면 다른 규칙을 적용해야한다.

즉, 웹 페이지가 작성된 규칙을 꼼꼼히 확인하는 작업이 필수적이다.

(만약 웹 페이지가 제공되는 규칙이 변경된다면 그 전에 작성해둔 코드는 쓸모가 없어진다)

웹 스크래핑을 더 어렵게 만드는 것은 요즘은 리소스 최적화를 위해 동적 로딩을 적용한 페이지가 많아졌다는 점이다.

예전에는 HTML과 CSS, 파라미터만 분석하면 됐다면 요즘은 페이지 내에서 발생하는 network connection까지 분석해야하는 경우가 많다.

## 1. Static Webpage Scrapping-pandas.read_html()

가장 쉬운 형태의 웹 페이지 스크래핑을 해보자.

https://sports.news.naver.com/esports/index.nhn 에 접속하면 Riot Games의 LOL의 프로리그 순위를 확인할 수 있다.

이 페이지에서 각 팀의 전적 기록을 pandas dataframe으로 가져오는 코드를 작성해보자.

```
import pandas as pd

url = 'https://sports.news.naver.com/esports/index.nhn'
dfs = pd.read_html(url)
dfs[0]
```

- [실행 결과]
> ![img03](https://github.com/jaehwan-dev/study-in-mis/blob/master/imgs/img03-pd.read_html.JPG)

단 세 줄의 짧은 코드로 생각보다 쉽게 데이터가 불러와졌다.

이제 어떤 과정을 통해 이런 결과를 얻을 수 있었는지 파해쳐보자.

브라우저의 메뉴에 개발자 도구를 활용하면 다양한 정보를 얻을 수 있다.

> ![img04](https://github.com/jaehwan-dev/study-in-mis/blob/master/imgs/img04-developer%20tool.JPG)

https://sports.news.naver.com/esports/index.nhn 의 HTML 코드를 볼 수 있는데, 이 코드들 중에 `<table>` 태그에 집중하자.

> ![img05](https://github.com/jaehwan-dev/study-in-mis/blob/master/imgs/img05-HTML%20tag.JPG)

`pandas.read_html()` 메쏘드는 HTML 코드를 입력받아 해당 코드의 `<table>` 구조체를 모두 dataframe 형태로 바꾸어 list 타입으로 반환한다.

만약 필요로 하는 데이터가 `<table>` 태그에 포함되어 있다면 간편하게 데이터를 수집할 수 있다.

## 2. Static Webpage Scrapping-HTML parsing

