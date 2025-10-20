
나는 현재 AWS의 Lambda, DynamoDB, EventBridge서비스를 이용해서 뉴스데이터를 내가 자는시간에 매일 매일 서버에 저장하고 있다. 내가 일어나면 dynamoDB에 저장된 뉴스를 가져와서 파이썬으로 분석후 시각화한것을 아침마다 받아 볼 수 있다. 

dynamodb에 write를 할때(데이터를 저장) 비용이 발생하는데 이 비용을 효율적으로 관리하기 위해 현재 내 데이터 저장 워크플로우를 살펴 보기로 했다. 

| Amazon DynamoDB APN2-TimedStorage-ByteHrs                     |                            | USD 0.00 |
| ------------------------------------------------------------- | -------------------------- | -------- |
| $0.00 per GB-Month of storage for first 25 free GB-Months     | 0.267 GB-Mo                | USD 0.00 |
| Amazon DynamoDB PayPerRequestThroughput                       |                            | USD 0.25 |
| $0.1355 per million read request units (Asia Pacific (Korea)) | 244,274.5 ReadRequestUnits | USD 0.03 |
| $0.68 per million write request units (Asia Pacific (Korea))  | 328,776 WriteRequestUnits  | USD 0.22 |
이번달(20일간)은 33만번의 write가 있었고 (33만행의 데이터 저장함) 0.22달러의 지출이 예상된다.
근데 실제 저장된 데이터의 개수를 확인해보니 16만개 였다. 
``` python
import boto3

import pandas as pd

import logging

logger = logging.getLogger()

from boto3.dynamodb.conditions import Key

from datetime import datetime, timedelta

  

def fetch_news_by_date_range_from_aws(start_date, end_date):

    """

    AWS DynamoDB에서 날짜 범위의 뉴스 데이터를 가져오는 함수

    Args:

        start_date (str): 시작 날짜 (형식: "YYYY-MM-DD")

        end_date (str): 종료 날짜 (형식: "YYYY-MM-DD")

    Returns:

        pd.DataFrame: 해당 날짜 범위의 뉴스 데이터가 담긴 DataFrame

    """

    logger.info(f"날짜 범위로 뉴스 데이터 조회: {start_date} ~ {end_date}")

    try:

        # DynamoDB 리소스와 테이블 설정

        dynamodb = boto3.resource('dynamodb', region_name='ap-northeast-2')

        table = dynamodb.Table('gut_test_2')

        # 날짜 범위 계산 (start_date부터 end_date까지의 모든 날짜)

        start = datetime.strptime(start_date, "%Y-%m-%d")

        end = datetime.strptime(end_date, "%Y-%m-%d")

        date_range = [(start + timedelta(days=i)).strftime("%Y-%m-%d")

                      for i in range((end - start).days + 1)]

        all_items = []

        # 각 날짜별로 쿼리 실행하여 결과 합치기

        for date in date_range:

            response = table.query(

                IndexName='date-pubDate-index',

                KeyConditionExpression=Key('date').eq(date)

            )

            items = response.get('Items', [])

            all_items.extend(items)

            logger.info(f"{date} 날짜의 뉴스 {len(items)}개 조회 완료")

            # 페이지네이션 처리 (결과가 1MB를 초과하는 경우)

            while 'LastEvaluatedKey' in response:

                response = table.query(

                    IndexName='date-pubDate-index',

                    KeyConditionExpression=Key('date').eq(date),

                    ExclusiveStartKey=response['LastEvaluatedKey']

                )

                items = response.get('Items', [])

                all_items.extend(items)

                logger.info(f"{date} 날짜의 뉴스 추가 {len(items)}개 조회 완료")

        logger.info(f"총 {len(all_items)}개 뉴스 데이터 조회 완료 ({start_date} ~ {end_date})")

        # DataFrame으로 변환

        df = pd.DataFrame(all_items)

        # pubDate 문자열을 datetime으로 변환

        if not df.empty and 'pubDate' in df.columns:

            df['pubDate'] = pd.to_datetime(df['pubDate'], format="%Y-%m-%d %H:%M:%S")

            logger.info("pubDate 컬럼을 datetime 형식으로 변환 완료")

        return df

    except Exception as e:

        logger.error(f"날짜 범위 {start_date} ~ {end_date} 뉴스 데이터 조회 중 오류 발생: {e}")

        return pd.DataFrame()  # 빈 DataFrame 반환


news_df = fetch_news_by_date_range_from_aws("2025-06-01", "2025-06-20")

print(f"총 {len(news_df)}개 뉴스 데이터 로드 완료")

```

총 166324개 뉴스 데이터 로드 완료

즉, 현재 비효율이 발생하고 있고 절반으로 비용을 줄일 수 있는것이다. 
원인은 겹치는 데이터를 덮어쓰듯이 저장하기 때문으로 보인다. 
현재 eventbridge로 20분 간격으로 lambda함수를 호출하여 네이버뉴스api를  이용하여 뉴스를 db에 저장하고 있다. 이때문에 20분이 지나고 뉴스데이터를 다시 받아올때 겹치는 데이터가 존재할 수 있다. (websocket이 아니라 http기 때문에)


plotly로 시간대별 뉴스의 개수를 분석하여 비효율을 개선하고자 했다. 
![[Pasted image 20250621111619.png]]
![[Pasted image 20250621111644.png]]

나는  현재 20분간격(아침3시간, 저녁3시간)으로 api에 1000개씩 데이터를 가져오게 request를 넣고 있는데 20분동안 저장된 실제 뉴스데이터의 개수의 최대값은 500~600 언저리 이다. 즉, 20분동안 뉴스가 1000개나 발행되지는 않고 있는것이다. 여기서 40~50%의 비효율이 발생함을 확인했다. 
따라서 1000개에서 500개로 데이터를 저장하도록 lambda함수의 코드를 바꾸기로 했다. 

