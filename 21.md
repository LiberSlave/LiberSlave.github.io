![[Pasted image 20250722075811.png]]
에이치디현대일렉트릭(주) 연결재무제표기준영업(잠정)실적(공정공시)
위는 실적공시의 뉴스 이름임.
사진은 이걸 분석하는 방법.
1. 컨센 대비
2. 전년동기대비
매출액과 영업이익

![[Pasted image 20250722084556.png]]


## DART에 공시 검색하는법
![[Pasted image 20250722172630.png]]
연결재무제표기준영업(잠정)실적(공정공시)

20250722800153 -> 2066, -1886
20250722800121 -> 2091, 2101
20250721800051 -> 3416, 5949
20250722900040 -> 242467, 87925

``` sql
CREATE TABLE disclosure_analyze (
  id           INT AUTO_INCREMENT PRIMARY KEY,
  date         DATE      NOT NULL,
  time         TIME      NOT NULL,
  ticker       VARCHAR(6) NOT NULL,
  name         VARCHAR(20),
  market_cap   INT,  -- (억)
  yoy_opm      DECIMAL(5,1),
  consen_opm   DECIMAL(5,1)
);
```

select * from disclosure_analyze;


