
### query 예시


- 키워드를 포함한 리포트 보기
``` sql
-- 1. RankedPerNews 라는 임시 결과 집합(CTE)을 정의합니다.
WITH RankedPerNews AS (
    SELECT
        n.realkey,
        n.time,
        n.title,
        s.name,
        -- [핵심] 뉴스 고유키(realkey)로 그룹을 나누고, 그룹 내에서 종목명(name)을 기준으로 순번을 매깁니다.
        ROW_NUMBER() OVER (PARTITION BY n.realkey ORDER BY s.name ASC) AS rn
    FROM
        news AS n
    -- 뉴스와 뉴스-종목 테이블을 조인합니다.
    JOIN news_stock AS ns ON n.realkey = ns.realkey
    -- 뉴스-종목 테이블과 종목 테이블을 조인합니다.
    JOIN stock AS s ON ns.ticker = s.ticker
    WHERE
        -- 오늘 날짜의 뉴스만 선택합니다.
        n.date = CURDATE()
        -- 지정된 시간대(오전 6시 ~ 9시)의 뉴스만 선택합니다.
        AND n.time BETWEEN '06:00:00' AND '09:00:00'
        AND s.market_cap < 150000
        -- [핵심] 제목에 특정 증권사 리포트 관련 키워드가 포함된 뉴스만 필터링합니다.
        AND n.title REGEXP '하나|메리츠|유안타|IBK|키움|대신|NH|DS투자증권|KB|상상인|한국투자|한투|신한|JP모건|JB금융지주|삼성증권|하이교보|LS|미래에셋|현대차|교보|SK|부국|유진|클릭 e종목'
)
-- 2. 위에서 만든 RankedPerNews 결과 집합에서 최종 데이터를 선택합니다.
SELECT
    realkey,
    time,
    title,
    name AS stock_name
FROM RankedPerNews
WHERE
    -- [핵심] 위에서 매긴 순번(rn)이 1인 데이터만 선택하여, 뉴스당 하나의 대표 종목만 남깁니다.
    rn = 1
	AND title REGEXP '조선|로봇|방산|원전|바이오|마스가|masga|오라클|sofc|식품|알래스카|lng|라면|김밥'
-- 최종 결과를 시간 오름차순(오래된 순)으로 정렬합니다.
ORDER BY time ASC;
```

``` sql
-- 1. RankedPerNews 라는 임시 결과 집합(CTE)을 정의합니다.
WITH RankedPerNews AS (
    SELECT
        n.realkey,
        n.time,
        n.title,
        s.name,
        -- [핵심] 뉴스 고유키(realkey)로 그룹을 나누고, 그룹 내에서 종목명(name)을 기준으로 순번을 매깁니다.
        ROW_NUMBER() OVER (PARTITION BY n.realkey ORDER BY s.name ASC) AS rn
    FROM
        news AS n
    -- 뉴스와 뉴스-종목 테이블을 조인합니다.
    JOIN news_stock AS ns ON n.realkey = ns.realkey
    -- 뉴스-종목 테이블과 종목 테이블을 조인합니다.
    JOIN stock AS s ON ns.ticker = s.ticker
    WHERE
        -- 오늘 날짜의 뉴스만 선택합니다.
        n.date = CURDATE()
        -- 지정된 시간대(오전 6시 ~ 9시)의 뉴스만 선택합니다.
        AND n.time BETWEEN '06:00:00' AND '09:00:00'
        AND s.market_cap < 50000
        -- [핵심] 제목에 특정 증권사 리포트 관련 키워드가 포함된 뉴스만 필터링합니다.
        AND n.title REGEXP '하나|메리츠|유안타|IBK|키움|대신|NH|DS투자증권|KB|상상인|한국투자|한투|신한|JP모건|JB금융지주|삼성증권|하이교보|LS|미래에셋|현대차|교보|SK|부국|유진|클릭 e종목'
)
-- 2. 위에서 만든 RankedPerNews 결과 집합에서 최종 데이터를 선택합니다.
SELECT
    realkey,
    time,
    title,
    name AS stock_name
FROM RankedPerNews
WHERE
    -- [핵심] 위에서 매긴 순번(rn)이 1인 데이터만 선택하여, 뉴스당 하나의 대표 종목만 남깁니다.
    rn = 1
-- 최종 결과를 시간 오름차순(오래된 순)으로 정렬합니다.
ORDER BY time ASC;
```


## 장마감후 쿼리


- 오늘의 특징주(2조 이하)
``` sql
WITH RankedNews AS (
    SELECT
        n.date,
        n.time,
        n.realtime,
        n.title,
        s.name,
        nb.max_10m,
        nsp.market_cap,
        nb.high_10m_time,
        nb.min_10m,
        nb.low_10m_time,
        n.media,
        -- [핵심] realkey(뉴스) 별로 그룹을 묶고, 그 안에서 max_10m이 높은 순서대로 순번(rn)을 매김
        ROW_NUMBER() OVER(PARTITION BY n.realkey ORDER BY nb.max_10m DESC) as rn
    FROM
        news AS n
    JOIN
        news_backtest AS nb ON n.realkey = nb.realkey
    JOIN
        news_stock AS ns ON n.realkey = ns.realkey
    JOIN
        stock AS s ON ns.ticker = s.ticker
    JOIN
        news_snapshot AS nsp ON n.realkey = nsp.realkey AND ns.ticker = nsp.ticker -- ticker까지 조인 조건에 추가
    WHERE
        n.date = CURDATE()
        AND n.title LIKE '%%[특징주]%%'
        AND nsp.market_cap < 20000
)
-- 위에서 매긴 순번(rn)이 1인, 즉 각 뉴스별로 가장 수익률이 높았던 종목 하나만 선택
SELECT
    date,
    time,
    realtime,
    title,
    name,
    max_10m,
    high_10m_time,
    min_10m,
    low_10m_time,
    media
FROM
    RankedNews
WHERE
    rn = 1
ORDER BY
    max_10m ASC,
    realtime ASC;
```

- 오늘의 뉴스(정규장 시간, 공시제외, 2조이하, 평일만, 대상,남성,..... 제외)
``` sql 
WITH UniqueNews AS (
        SELECT
            n.realkey,
            n.title,
            n.date,
            n.time,
            n.realtime,
            s.name,
            nb.max_10m,
            nb.high_10m_time,
            nb.min_10m,
            nb.low_10m_time,
            nsp.high,
            nsp.low,
            nsp.current,    
            nsp.current_rate,
            nsp.high_rate,
            nsp.market_cap,
            nsp.margin_rate,
            nsp.trading_value,
            n.media,
            -- 한 뉴스가 여러 종목에 연결되어 조건에 여러 번 해당될 수 있으므로,
            -- realkey를 기준으로 중복을 제거하기 위해 순번을 매김
            ROW_NUMBER() OVER (PARTITION BY n.realkey ORDER BY n.realtime ASC) as rn_unique_news
        FROM
            news AS n
        JOIN
            news_backtest AS nb ON n.realkey = nb.realkey
        JOIN
            news_stock AS ns ON n.realkey = ns.realkey
        JOIN
            stock AS s ON ns.ticker = s.ticker
        JOIN
            news_snapshot AS nsp ON n.realkey = nsp.realkey AND s.ticker = nsp.ticker
        WHERE
            n.date = CURDATE()
            AND n.realtime BETWEEN '09:03:00' AND '15:10:00'
            AND n.media != '공시'
            AND nsp.market_cap < 20000
            AND WEEKDAY(n.date) < 5
            AND s.name NOT IN ('레이', '대상', '남성', '진영')
    )
    -- 최종 조회: 필터링 및 중복 제거된 뉴스의 상세 정보를 조회
    SELECT
		date,
	    title,
	    time,
	    realtime,
	    name,
	    max_10m,
	    high_10m_time,
	    min_10m,
	    low_10m_time,
	    high,
	    low,
	    current,
	    current_rate,
	    high_rate,
	    market_cap,
	    margin_rate,
	    trading_value,
	    media
    FROM
        UniqueNews
    WHERE
        rn_unique_news = 1 -- 각 realkey 그룹 내의 첫 번째 행만 선택하여 고유한 뉴스만 표시
    ORDER BY
        name ASC, date ASC, max_10m DESC;

```

- 오늘의 뉴스(정규장 시간, 공시제외, 2조이하, 평일만, 대상,남성,..... 제외) + 중복종목 제외
``` sql 
-- 1단계: realkey 기준으로 고유한 뉴스를 먼저 필터링합니다. (기존과 거의 동일)
WITH UniqueNews AS (
    SELECT
        n.realkey,
        n.title,
        n.date,
        n.time,
        n.realtime,
        s.name,
        nb.max_10m,
        nb.high_10m_time,
        nb.min_10m,
        nb.low_10m_time,
        nsp.high,
        nsp.low,
        nsp.current,
        nsp.current_rate,
        nsp.high_rate,
        nsp.market_cap,
        nsp.margin_rate,
        nsp.trading_value,
        n.media,
        ROW_NUMBER() OVER (PARTITION BY n.realkey ORDER BY n.realtime ASC) as rn_unique_news
    FROM
        news AS n
    JOIN
        news_backtest AS nb ON n.realkey = nb.realkey
    JOIN
        news_stock AS ns ON n.realkey = ns.realkey
    JOIN
        stock AS s ON ns.ticker = s.ticker
    JOIN
        news_snapshot AS nsp ON n.realkey = nsp.realkey AND s.ticker = nsp.ticker
    WHERE
        n.date = CURDATE()
        AND n.realtime BETWEEN '09:03:00' AND '15:10:00'
        AND n.media != '공시'
        AND nsp.market_cap < 20000
        AND WEEKDAY(n.date) < 5
        AND s.name NOT IN ('레이', '대상', '남성', '진영')
),
-- 2단계: 위에서 필터링된 결과를 대상으로, 이번엔 종목(name)별로 max_10m이 가장 높은 뉴스에 순번을 매깁니다.
RankedByStock AS (
    SELECT
        *,
        -- name으로 그룹을 묶고, 그 안에서 max_10m이 높은 순서대로 1, 2, 3... 순번을 매김
        ROW_NUMBER() OVER (PARTITION BY name ORDER BY max_10m DESC) as rn_stock_best
    FROM
        UniqueNews
    WHERE
        rn_unique_news = 1 -- 1단계 필터링 적용
)
-- 최종 조회: 각 종목별로 순번이 1번인, 즉 max_10m이 가장 높은 뉴스만 선택합니다.
SELECT
    date,
    title,
    time,
    realtime,
    name,
    max_10m,
    high_10m_time,
    min_10m,
    low_10m_time,
    high,
    low,
    current,
    current_rate,
    high_rate,
    market_cap,
    margin_rate,
    trading_value,
    media
FROM
    RankedByStock
WHERE
    rn_stock_best = 1 -- 2단계 필터링 적용
ORDER BY
    date ASC, time ASC; 
```


- 오늘 리포트 결과
``` sql
-- 1. RankedPerNews 라는 임시 결과 집합(CTE)을 정의합니다.
WITH RankedPerNews AS (
    SELECT
        n.realkey,
        n.time,
        n.title,
        s.name,
        nb.max_daily, -- [추가] news_backtest 테이블에서 max_daily 컬럼을 선택합니다.
        -- 뉴스 고유키(realkey)로 그룹을 나누고, 그룹 내에서 종목명(name)을 기준으로 순번을 매깁니다.
        ROW_NUMBER() OVER (PARTITION BY n.realkey ORDER BY s.name ASC) AS rn
    FROM
        news AS n
    -- 뉴스와 뉴스-종목, 종목 테이블을 조인합니다.
    JOIN news_stock AS ns ON n.realkey = ns.realkey
    JOIN stock AS s ON ns.ticker = s.ticker
    -- [추가] news_backtest 테이블을 realkey 기준으로 조인합니다.
    JOIN news_backtest AS nb ON n.realkey = nb.realkey
    WHERE
        -- 오늘 날짜의 뉴스만 선택합니다.
        n.date = CURDATE()
        -- 지정된 시간대(오전 6시 ~ 9시)의 뉴스만 선택합니다.
        AND n.time BETWEEN '06:00:00' AND '09:00:00'
        -- 제목에 특정 증권사 리포트 관련 키워드가 포함된 뉴스만 필터링합니다.
        AND n.title REGEXP '하나|메리츠|유안타|IBK|키움|대신|NH|DS투자증권|KB|상상인|한국투자|한투|신한|JP모건|JB금융지주|삼성증권|하이교보|LS|미래에셋|현대차|교보|SK|부국|유진|클릭 e종목'
)
-- 2. 위에서 만든 RankedPerNews 결과 집합에서 최종 데이터를 선택합니다.
SELECT
    realkey,
    time,
    title,
    name AS stock_name,
    max_daily -- [추가] 최종 결과에 max_daily 컬럼을 포함합니다.
FROM RankedPerNews
WHERE
    -- 위에서 매긴 순번(rn)이 1인 데이터만 선택하여, 뉴스당 하나의 대표 종목만 남깁니다.
    rn = 1
    -- 제목에 '조선', '로봇', 'MASH' 중 하나가 반드시 포함되어야 한다는 조건은 주석처리하고 아래 REGEXP로 통합합니다.
    /*
    AND (
        title LIKE '%조선%'
        OR title LIKE '%로봇%'
        OR title LIKE '%MASH%'
    )
    */
    -- 제목에 '괴리율' 이라는 단어가 포함되지 않은 뉴스만 선택합니다.
    AND title NOT LIKE '%괴리율%'
-- [수정] 최종 결과를 max_daily 오름차순(낮은 값 순서)으로 정렬합니다.
ORDER BY max_daily ASC;

```


- 투자경고, 단기과열, 투자주의, 매매거래정지
``` sql
SELECT
    n.title,
    n.date,
    n.time,
    n.media,
    n.realtime,
    s.name -- stock 테이블의 종목명
FROM
    news n
JOIN
    news_stock ns ON n.realkey = ns.realkey
JOIN
    stock s ON ns.ticker = s.ticker
JOIN
    news_snapshot nsp ON n.realkey = nsp.realkey -- news_snapshot 테이블을 realkey로 조인
WHERE
    n.media = '공시'
    AND n.date = CURDATE()
    AND n.time BETWEEN '20:00:00' AND '20:03:59'
    -- 아래 3개 키워드가 포함된 title은 제외
    AND n.title NOT LIKE '%특정계좌%'
    AND n.title NOT LIKE '%소수계좌%'
    AND n.title NOT LIKE '%단일계좌%'
    -- trading_value가 10을 초과하는 경우만 선택
    AND nsp.trading_value > 100
ORDER BY
    -- title 내용에 따라 정렬 순서 지정
    CASE
        WHEN n.title LIKE '%투자경고종목%' THEN 1
        WHEN n.title LIKE '%단기과열%' THEN 2
        WHEN n.title LIKE '%공매도%' THEN 3
        ELSE 4 -- 그 외 다른 공시들은 마지막에
    END,
    n.time; -- 같은 그룹 내에서는 시간순으로 정렬```

```



## 잡것



- 오늘 뉴스중 시간범위 지정해서 보는 쿼리
``` sql
SELECT time, title
FROM news
WHERE date = CURDATE()
  AND time BETWEEN '13:00:00' AND '14:00:00';
```


- 오늘 뉴스 개수(7000~10000 사이?)
``` sql
SELECT COUNT(*)

FROM news

WHERE date = CURDATE();
```



- 의미있는 키워드인지 찾아보기
``` sql
SELECT
    n.date,      
    n.time,
    n.title,
    n.media,
    s.name AS stock_name
FROM
    news AS n
JOIN
    news_stock AS ns ON n.realkey = ns.realkey
JOIN
    stock AS s      ON ns.ticker   = s.ticker
WHERE
    n.date BETWEEN DATE_SUB(CURDATE(), INTERVAL 30 DAY) 
               AND CURDATE()
    AND n.realkey IN (
        SELECT realkey
        FROM news_stock
        GROUP BY realkey
        HAVING COUNT(*) <= 1
    )
    AND (n.title LIKE '%연결재무제표%')
    AND n.title NOT LIKE '%\%%' 


ORDER BY
    n.date DESC,    
    n.time DESC;

```

-  키워드 들어간 모든 뉴스검색
``` sql
SELECT
    n.time,
    n.title
FROM
    news AS n
WHERE
    n.date = CURDATE()
    AND n.title LIKE '%연결재무제표%'
ORDER BY
    n.time DESC;
```



1. title에 %있는것 제외
2. 관련종목개수 정확히 1~2만 존재 (종목 2개면 1개의 뉴스만 표시. <- 중복 표시 방지)
``` sql
SELECT
    n.time,
    n.title,
    GROUP_CONCAT(DISTINCT s.name ORDER BY s.name SEPARATOR ', ') AS stock_names
FROM news AS n
JOIN news_stock AS ns ON n.realkey = ns.realkey
JOIN stock AS s ON ns.ticker = s.ticker
WHERE n.date = DATE_SUB(CURDATE(), INTERVAL 4 DAY)
  AND n.realkey IN (
      SELECT realkey
      FROM news_stock
      GROUP BY realkey
      HAVING COUNT(*) BETWEEN 1 AND 2
  )
  AND n.title NOT LIKE '%\%%'
GROUP BY n.realkey, n.time, n.title
ORDER BY n.time DESC;

```


1. title에 %있는것 제외
2. 관련종목개수 정확히 1~2만 존재 (종목 2개면 1개의 뉴스만 표시. <- 중복 표시 방지)
3. 무드의 키워드.
``` sql

SELECT
    n.time,
    n.title,
    s.name AS stock_name
FROM
    news AS n
JOIN
    news_stock AS ns ON n.realkey = ns.realkey
JOIN
    stock AS s ON ns.ticker = s.ticker
WHERE
    n.date = DATE_SUB(CURDATE(), INTERVAL 4 DAY)
    -- 관련종목이 1개 이상, 2개 이하인 realkey만
    AND n.realkey IN (
        SELECT realkey
        FROM news_stock
        GROUP BY realkey
        HAVING COUNT(*) BETWEEN 1 AND 2
    )
    AND n.title NOT LIKE '%\%%'
    AND n.title REGEXP '특징주|속보|단독|세계|국내|최초|임상|3상|성공|fda|美|中|개발|인수|승인|매각|M&A|경영권|수주|최대주주|공급|역대|추진|협력|치료제|백신|완전관해|기술이전|정책|합병|효능|발표|정부|투자|상장|대통령|0배|보다|억원|조원|코인|계약|1보|국산화|무근|독점|글로벌'
ORDER BY
    n.time DESC;
```


1. title에 %있는것 제외
2. 관련종목개수 정확히 1~2만 존재 (종목 2개면 1개의 뉴스만 표시. <- 중복 표시 방지)
3. 시총 6000억 이하하
``` sql
SELECT
    n.time,
    n.title,
    GROUP_CONCAT(DISTINCT s.name ORDER BY s.name SEPARATOR ', ') AS stock_names
FROM news AS n
JOIN news_stock AS ns ON n.realkey = ns.realkey
JOIN stock AS s ON ns.ticker = s.ticker
WHERE n.date = DATE_SUB(CURDATE(), INTERVAL 5 DAY)
  AND n.realkey IN (
      SELECT realkey
      FROM news_stock
      GROUP BY realkey
      HAVING COUNT(*) BETWEEN 1 AND 2
  )
  AND n.title NOT LIKE '%\%%'
  AND s.market_cap < 6000
GROUP BY n.realkey, n.time, n.title
ORDER BY n.time DESC;
```


