# 트위터 크롤링 (3rd party library)

Twitter에서는 공식적으로 API를 이용해 데이터 수집 환경을 제공한다.

이 서비스는 기본적으로 무료로 제공되지만 일부 기능에 제한이 있다.

따라서 다양한 이유에 의해 1) 유료 서비스를 이용하거나, 2) 3rd party에서 개발된 라이브러리를 사용해야한다.

~~우리는 가난한 학생이므로 3rd party에서 개발된 라이브러리를 사용한다.~~

본 문서에서 소개될 3rd party 라이브러리는 GetOldTweets3이다.

해당 라이브러리는 가상의 브라우저를 연 후 페이지 스크롤을 통해 계속 트위터를 읽어오는 방식을 사용한다.

따라서 수집 속도가 Twitter API에 비해 현저히 느리다는 단점이 있다.

> ![img02](https://github.com/jaehwan-dev/study-in-mis/blob/master/imgs/img02-api%20vs%20http.PNG)

위의 그림을 보면 API request의 경우 서버에서는 요청을 받아, 그 요청이 적합한지 확인한 후

그 요청에 해당하는 데이터를 JSON이나 XML과 같은 간결한 자료형으로 response한다.

하지만 HTTP request의 경우 가상의 보이지 않는 브라우저를 띄우고,

해당 브라우저가 웹 사이트에 접속하는 것처럼 서버를 속이게 된다.

따라서 서버에서는 웹 사이트에 표시하기 위한 모든 데이터를 포함한 HTML을 response한다.

이런 이유로 브라우저를 이용한 데이터 수집은 필연적으로 속도가 느리며, 결과가 불안정한 경우가 많다.

# GetOldTweets3-키워드 검색을 통한 트윗 수집

> 수집된 결과(예시)

id | date(utc) | permalink | userid_eng | text
--- | --- | --- | --- | ---
1251439228845608960 | 2020-04-18 09:15:37+00:00 | https://twitter.com/jyj_jpris/status/125143922... | jyj_jpris | 아직도 신천지에서 코로나가 나오다니..........그저 이를 갈 뿐이다.
1251439207018422273 | 2020-04-18 09:15:32+00:00 | https://twitter.com/ace27568/status/1251439207... | ace27568 | 코로나를 정쟁에 활용하는 비열한 것들 그러니 너거는 폭망하는 거다.
1251439202434072576 | 2020-04-18 09:15:31+00:00 | https://twitter.com/Haha84128560/status/125143... | Haha84128560 | 일반감기가 아니라 xxxx코로나 아닌가요? 이러다가 졸지에 이부프로펜 퇴출 될듯 ㅎ...

## 1. 라이브러리 설치

tweepy와 달리 GetOldTweets3는 널리사용되는 라이브러리가 아니므로, 환경에 따라 설치할 필요가 있다.

아래 코드를 실행하면 GetOldTweet3 라이브러리의 설치를 확인하고 설치되지 않은 경우 설치한다.

```python
try:
  import GetOldTweets3 as got
except:
  !pip install GetOldTweets3
  import GetOldTweets3 as got
```

## 2. 라이브러리 로드와 객체 생성

GetOldTweets3 사용을 위해서 GetOldTweets3를 비롯해 데이터 처리에 필요한 라이브러리를 로드하고, 필요한 객체를 생성하였다.

수집된 트윗을 처리하기 위한 커스텀 함수도 함께 정의하였다.

```python
import pandas as pd
from datetime import datetime

def convertTweetToDictGOT(tweet):
  _tweet = {"id":tweet.id,
            "date(utc)":tweet.date,
            "permalink":tweet.permalink,
            "userid_eng":tweet.username,
            "text":tweet.text}
  return _tweet
```
