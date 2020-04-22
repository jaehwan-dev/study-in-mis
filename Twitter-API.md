# 트위터 크롤링 (API)

트위터에서 제공하는 공식 API를 이용해 트위터에서 데이터를 수집해보자.

트위터에서는 다양한 API를 제공하지만, 그 중에 키워드 검색을 통한 트윗 수집을 수행해본다.

(키워드 검색 이 외에도 개별 사용자의 타임라인 수집 등이 가능)

### 들어가며

Twitter에서는 공식적으로 API를 이용해 데이터 수집 환경을 제공한다.

이 서비스는 기본적으로 무료로 제공되지만 일부 기능에 제한이 있다.

예를 들어 키워드 검색을 통한 트위터 수집은 최근 7일간의 데이터로 제한되어있다. [[docs]](https://developer.twitter.com/en/docs/tweets/search/overview)

***API나 라이브러리를 사용할 때, 제공되는 document는 사용에 필요한 거의 모든 정보가 포함되어 있으므로 꼭 꼼꼼히 읽어보자!***

또한 15분당 요청할 수 있는 request의 횟수도 제한되어 있다. (180 times per user auth, 450 times per app auth)

따라서 그 이전 기간의 트위터를 수집하기 위해서는 1) 유료 서비스를 이용하거나, 2) 3rd party에서 개발된 라이브러리를 사용해야한다.

---

# Twitter API-키워드 검색을 통한 트윗 수집

> 수집된 결과(예시)

id | date(utc) | userid | username | text | retweet_from
--- | --- | --- | --- | --- | ---
1251439261863174144 | 2020-04-18 09:15:45 | 170993468 | 새로운 대한민국 | RT @khwook9: 이거 큰일 입니다 중대원전부 빨리격리 해야할듯 하네요 젊은친... | 1251382335079018496
1251439261485686784 | 2020-04-18 09:15:45 | 1246861575102722048 | 전분 | RT @OO_basic: 엥? 좆주빈 재산 몰수해서 왜 코로나 사태 진정에다가 쓰자... | 1251077873303199749
1251439256934858752 | 2020-04-18 09:15:44 | 439407254 | 취향이 운명이다~ | RT @hohonim70: 340만명 입국했는데 직원 확진 0명.. '코로나 방역관... | 1251280802110779392

## 1. Keys and tokens

트위터 API 사용을 위해서는 Key와 Token이 필요하다.

이는 트위터 측에서 API를 호출한 앱(혹은 사용자)이 누구인지 확인하기 위한 값이다.

https://developer.twitter.com/en/ 에 접속해 개발자 계정을 등록하고 App을 생성하면 API 사용에 필요한 key와 token을 받을 수 있다.

발급받은 key와 token 값을 아래 셀에 입력하자.

```python
consumer_key = "..."
consumer_secret = "..."
```

## 2. 라이브러리 로드와 객체 생성

트위터 API 사용을 위해서는 tweepy 라이브러리를 사용해야한다.

tweepy를 비롯해 데이터 처리에 필요한 라이브러리를 로드하고, 필요한 객체를 생성하였다.

수집된 트윗을 처리하기 위한 커스텀 함수도 함께 정의하였다.

```python
# 필요한 라이브러리 로드와 객체, 커스텀 함수 생성
import tweepy
import pandas as pd
from datetime import datetime

auth = tweepy.AppAuthHandler(consumer_key, consumer_secret)
api = tweepy.API(auth, wait_on_rate_limit=True, wait_on_rate_limit_notify=True)

def convertTweetToDict(tweet):
  if "retweeted_status" in tweet._json:
    _tweet = {"id":tweet.id_str, "date(utc)":tweet.created_at,
              "userid":tweet.user.id_str, "username":tweet.user.name,
              "text":tweet.text, "retweet_from":tweet.retweeted_status.id_str}
  else:
    _tweet = {"id":tweet.id_str, "date(utc)":tweet.created_at,
              "userid":tweet.user.id_str, "username":tweet.user.name,
              "text":tweet.text, "retweet_from":None}
  return _tweet
```

다음의 값들을 필요에 따라 수정해서 사용한다.

- keyword: 검색어
- until: 특정일 이전의 트윗만 검색할 수 있다. 지정하고 싶은 경우 날짜를 YYYY-MM-DD 양식으로 작성하여 따옴표로 묶어 입력한다. e.g. "2020-04-16" 단, 7일 보다 오래된 트윗은 검색할 수 없다.
- ouput_filename: 수집한 트윗을 저장할 파일 이름, 지정하지 않은 경우 *키워드_after_마지막트윗ID.csv*로 저장된다.

```python
keyword = "코로나"
until = None # YYYY-MM-DD일 이전의 트윗만 검색
output_filename = None # 트윗을 저장할 파일 이름
```

## 3. 트윗 수집

위의 값을 수정한 후 아래 코드를 실행하면 트윗이 수집된다.

```python
tweetCount = 0
rpp = 100
max_tweets = 45000
max_id = -1

print ("[{}] ...트윗 수집 시작...".format(datetime.now().strftime("%H:%M:%S")))

tweets = [None] * max_tweets
while (tweetCount < max_tweets):
  if until:
    if (max_id <= 0):
      new_tweets = api.search(q=keyword, result_type="recent", count=rpp,
                              until=until, include_entities=False)
    else:
      new_tweets = api.search(q=keyword, result_type="recent", count=rpp,
                              until=until, include_entities=False, max_id=str(max_id-1))
  else:
    if (max_id <= 0):
      new_tweets = api.search(q=keyword, result_type="recent", count=rpp,
                              include_entities=False)
    else:
      new_tweets = api.search(q=keyword, result_type="recent", count=rpp,
                              include_entities=False, max_id=str(max_id-1))

  if new_tweets:
    tweets[tweetCount:tweetCount+rpp] = new_tweets
    tweetCount += len(new_tweets)
    max_id = new_tweets[-1].id
  else:
    print ("[{}] ...트윗 수집 종료...".format(datetime.now().strftime("%H:%M:%S")))
    print ("총 {:,}개의 트윗이 수집되었습니다.".format(tweetCount))
    break
else:
  print ("[{}] ...트윗 수집 종료...".format(datetime.now().strftime("%H:%M:%S")))
  print ("총 {:,}개의 트윗이 수집되었으며 아직 수집할 트윗이 더 남아있습니다".format(tweetCount))
  print ("max_id: {}".format(tweets[-1].id))

tweets = [tweet for tweet in tweets if tweet is not None]
df = pd.DataFrame(list(map(convertTweetToDict, tweets)))
if output_filename:
  df.to_csv(output_filename, index=False)
else:
  df.to_csv("{}_after_{}.csv".format(keyword, max_id), index=False, encoding='euc-kr')
```

수집된 트윗은 클라우드 서버에 바로 저장되며, 저장된 트윗은 왼쪽 메뉴의 파일 아이콘을 클릭해 다운로드할 수 있다.

> ![img01](https://github.com/jaehwan-dev/study-in-mis/blob/master/imgs/img01-download%20outputfile.png)

## 코드 리뷰
