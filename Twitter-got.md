# 트위터 크롤링 (3rd party library)

Twitter에서는 공식적으로 API를 이용해 데이터 수집 환경을 제공한다.

이 서비스는 기본적으로 무료로 제공되지만 일부 기능에 제한이 있다.

따라서 다양한 이유에 의해 1) 유료 서비스를 이용하거나, 2) 3rd party에서 개발된 라이브러리를 사용해야한다.

~~우리는 가난한 학생이므로 3rd party에서 개발된 라이브러리를 사용한다.~~

본 문서에서 소개될 3rd party 라이브러리는 GetOldTweets3이다. [[doc]](https://pypi.org/project/GetOldTweets3/)

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

다음의 값들을 필요에 따라 수정해서 사용한다.

**Twitter API와 달리 속도가 느리고 검색수 제한이 빡빡하므로 until, since를 모두 입력해 사용한다**

- keyword: 검색어
- until: 특정일(UTC 기준) 미만의 트윗만 검색할 수 있다. 지정하고 싶은 경우 날짜를 YYYY-MM-DD 양식으로 작성하여 따옴표로 묶어 입력한다. e.g. "2020-04-16" 단, 7일 보다 오래된 트윗은 검색할 수 없다.
- since: 특정일(UTC 기준) 이후의 트윗만 검색할 수 있다.
- ouput_filename: 수집한 트윗을 저장할 파일 이름, 지정하지 않은 경우 *키워드_after_마지막트윗ID.csv*로 저장된다.

```python
keyword = "주식"
until = None # YYYY-MM-DD일 이전의 트윗만 검색
since = None # YYYY-MM-DD일 이후의 트윗만 검색
output_filename = None # 트윗을 저장할 파일 이름
```

## 3. 트윗 수집

위의 값을 수정한 후 아래 코드를 실행하면 트윗이 수집된다.

안정적인 트윗 수집을 위해 한번에 수집할 트윗의 양을 너무 크지 않은 숫자로 설정하는 것이 좋다. (e.g. 10,000)

```python
max_tweets = 10000 # 10,000개 수집하는데 7분 정도 걸림

print ("[{}] ...트윗 수집 시작...".format(datetime.now().strftime("%H:%M:%S")))
tweetCriteria = got.manager.TweetCriteria()
tweetCriteria.setQuerySearch(keyword)
tweetCriteria.setMaxTweets(max_tweets)
tweetCriteria.setTopTweets(False)
if since:
  tweetCriteria.setSince(since)
if until:
  tweetCriteria.setUntil(until)

tweets = got.manager.TweetManager.getTweets(tweetCriteria)

print ("[{}] ...트윗 수집 종료...".format(datetime.now().strftime("%H:%M:%S")))
print ("총 {:,}개의 트윗이 수집되었습니다.".format(len(tweets)))

tweets = [tweet for tweet in tweets if tweet is not None]
df = pd.DataFrame(list(map(convertTweetToDictGOT, tweets)))
if output_filename:
  df.to_csv(output_filename, index=False)
else:
  df.to_csv("{}_since_{}_until_{}.csv".format(keyword, since, until), index=False, encoding='euc-kr')
```

수집된 트윗은 클라우드 서버에 바로 저장되며, 저장된 트윗은 왼쪽 메뉴의 파일 아이콘을 클릭해 다운로드할 수 있다.

> ![img01](https://github.com/jaehwan-dev/study-in-mis/raw/master/imgs/img01-download%20outputfile.png)

## 코드 리뷰
