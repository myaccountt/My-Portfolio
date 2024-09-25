# GA 데이터를 이용한 분석 및 대시보드 작성

Text: Google Merchandise Store의 Google Analytics 데이터를 사용하여 대시보드를 작성하고 분석
수행 기간: 2024-05 ~ 2024-07
수행인원: 1
사용 툴: SQL, Python, Tableau, Excel
배경: 포트폴리오 작성을 위한  1인 개인 프로젝트입니다.

![https://blog.kakaocdn.net/dn/6yjiN/btsITg0JvYJ/0A0kQlKFAEc7ANHNUKvuJ0/img.png](https://blog.kakaocdn.net/dn/6yjiN/btsITg0JvYJ/0A0kQlKFAEc7ANHNUKvuJ0/img.png)

Google analytics 데이터 분석을 리뷰합니다.

- 분석 목표 : 데이터를 분석하여 매출 증대를 위한 아이디어 제언
- 사용 데이터 :데이터는 구글 빅쿼리에 기본 데이터로 제공되고 있는 "google_analytics_sample"을 사용하였습니다.
- 2016년 8월 1일~2017년 8월 1일에 Google Merchandise Store에서 발생한 거래 내역을 google analytics를 이용하여 수집한 데이터입니다.

![https://blog.kakaocdn.net/dn/b3qiD1/btsHCnrwDdv/qs2Fhy1kkA1ATpyHwt1jBK/img.png](https://blog.kakaocdn.net/dn/b3qiD1/btsHCnrwDdv/qs2Fhy1kkA1ATpyHwt1jBK/img.png)

구글 SDK를 설치하고, 애플리케이션 기본 자격 증명 작업을 수행한 후 해당 데이터를 파이썬과 연동시켰습니다.

각 필드에 관한 설명은 [https://support.google.com/analytics/answer/3437719?hl=ko](https://support.google.com/analytics/answer/3437719?hl=ko)를 참조

![https://blog.kakaocdn.net/dn/63pOh/btsITiDonUr/spzwupFUEc4RjRmZgZoIW0/img.png](https://blog.kakaocdn.net/dn/63pOh/btsITiDonUr/spzwupFUEc4RjRmZgZoIW0/img.png)



```bash
query = """
WITH table1 AS (
    SELECT
      visitId,
      product.v2ProductName AS ProductName,
      product.ProductSKU AS ProductSKU,
      MAX(IF(hits.eCommerceAction.action_type = '1', 1, 0)) AS `제품 리스트 클릭`,
      MAX(IF(hits.eCommerceAction.action_type = '2', 1, 0)) AS `제품 상세보기`,
      MAX(IF(hits.eCommerceAction.action_type = '3', 1, 0)) AS `장바구니 담기`,
      MAX(IF(hits.eCommerceAction.action_type = '5', 1, 0)) AS `결제`,
      MAX(IF(hits.eCommerceAction.action_type = '6', 1, 0)) AS `결제 완료`
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201703*`,
         UNNEST(hits) AS hits,
         UNNEST(hits.product) AS product
    WHERE hits.eCommerceAction.action_type != '0'
      AND hits.eCommerceAction.action_type != '4'
      AND hits.eCommerceAction.action_type != '7'
      AND hits.eCommerceAction.action_type != '8'
    GROUP BY visitId, ProductName, product.ProductSKU
    ORDER BY visitId, product.ProductSKU
),

table2 AS (
    SELECT
      visitId,
      hits.eCommerceAction.action_type AS action_type,
      product.v2ProductName AS ProductName,
      product.ProductSKU AS ProductSKU,
      date,
      hits.hour,
      geoNetwork.country,
      product.productQuantity AS productQuantity,
      product.productPrice,
      hits.transaction.transactionRevenue AS transactionRevenue,
      hits.appInfo.landingScreenName,
      hits.appInfo.exitScreenName,
      trafficSource.source,
      ROW_NUMBER() OVER (
          PARTITION BY visitId, product.ProductSKU, product.v2ProductName
          ORDER BY hits.eCommerceAction.action_type DESC
      ) AS rn
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201703*`,
         UNNEST(hits) AS hits,
         UNNEST(hits.product) AS product
    WHERE hits.eCommerceAction.action_type != '0'
      AND hits.eCommerceAction.action_type != '4'
      AND hits.eCommerceAction.action_type != '7'
      AND hits.eCommerceAction.action_type != '8'
),

filtered_table2 AS (
    SELECT
      visitId, action_type, ProductName, ProductSKU, date, hour, country, productQuantity,
      productPrice, transactionRevenue, landingScreenName, exitScreenName, source
    FROM table2
    WHERE rn = 1
    ORDER BY visitId, ProductSKU
)

SELECT
  t1.visitId,  t1.ProductName,  t1.ProductSKU,  t1.`제품 리스트 클릭`,  t1.`제품 상세보기`,  t1.`장바구니 담기`,  t1.`결제`,  t1.`결제 완료`,
  t2.date,  t2.action_type, t2.hour,  t2.country,  t2.productQuantity,  t2.productPrice, t2.transactionRevenue,  t2.landingScreenName,  t2.exitScreenName,  t2.source
FROM
  table1 t1
LEFT JOIN
  filtered_table2 t2
ON
  t1.visitId = t2.visitId
  AND t1.ProductName = t2.ProductName
  AND t1.ProductSKU = t2.ProductSKU;
"""
f_data = client.query(query).result().to_dataframe()
```

| visitId | ProductName | ProductSKU | 제품 리스트 클릭 | 제품 상세보기 | 장바구니 담기 | 결제 | 결제 완료 | date | action_type | hour | country | productQuantity | productPrice | transactionRevenue | landingScreenName | exitScreenName | source |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1489612431 | Google Men's Lightweight Microfleece Jacket Black | GGOEGAAX0573 | 1 | 1 | 0 | 0 | 0 | 20170315 | 제품 상세보기 | 14 | United States | 0 | 74990000 | 0 | shop.googlemerchandisestore.com/home | shop.googlemerchandisestore.com/google+redesign/apparel/mens/mens+outerwear | google |
| 1489616619 | 25L Classic Rucksack | GGOEGAAX0795 | 1 | 1 | 0 | 0 | 0 | 20170315 | 제품 상세보기 | 15 | United States | 0 | 99990000 | 0 | shop.googlemerchandisestore.com/home | shop.googlemerchandisestore.com/google+redesign/bags/backpacks//quickview | google |
| 1489616619 | Google Alpine Style Backpack | GGOEGBRJ037299 | 1 | 1 | 0 | 0 | 0 | 20170315 | 제품 상세보기 | 15 | United States | 0 | 99990000 | 0 | shop.googlemerchandisestore.com/home | shop.googlemerchandisestore.com/google+redesign/bags/backpacks//quickview | google |

위 코드를 사용하여, 방문자마다 확인한 제품을 기본 키로 하는 데이터 프레임을 생성하였습니다.
해당 데이터를 제품으로 필터링하여 제품별 방문자의 행동에 대한 통계를 대시보드로 작성할 수 있습니다. 

![Untitled](https://github.com/user-attachments/assets/f8a44364-aa10-42df-99f7-41caca38d72c)

위 데이터를 사용하여 작성한 Excel 대시보드입니다. 날짜와 최종 행동유형, 제품명으로 대시보드를 필터링 할 수 있습니다. 

해당 데이터를 통해 확인할 수 있는 인사이트는 다음과 같습니다.

- 방문자 중 약 60%가 미국에서 방문합니다.
- 40% 정도의 방문자가 오전 8시~오후2시 시간대에 방문합니다.
- 90%에 가까운 방문자가 직접 혹은 구글을 통해 방문합니다.
- 결제 단계에서 결제 완료를 하지 않은 고객은 결제 페이지보다 고객 정보 페이지에서 이탈한 경우가 많아 해당 페이지에 대한 개선이 필요해 보입니다.

![https://blog.kakaocdn.net/dn/bqyJzs/btsISeuXCue/wwB4GkMjvKxntL5tZJgFRk/img.png](https://blog.kakaocdn.net/dn/bqyJzs/btsISeuXCue/wwB4GkMjvKxntL5tZJgFRk/img.png)

- 매출을 상승시키기 위한 방법으로 장바구니에 물건을 넣고 이탈한 고객이 해당 물품을 잊어버리고 다른 판매처에서 물품을 구매하기 전에 해당 제품을 구매하도록 유도하는 방법이 있습니다.
- 이러한 방법으로 리마인드 메시지 혹은 리타겟팅 광고를 통해 해당 제품을 장바구니에 넣은 것을 상기시키는 방법이 있는데 해당 방법의 최적 타이밍이 언제인지 확인해 보겠습니다.

![https://blog.kakaocdn.net/dn/s7vfE/btsIRym6pV5/heZkpDQTsU56WE9PpXg9hK/img.png](https://blog.kakaocdn.net/dn/s7vfE/btsIRym6pV5/heZkpDQTsU56WE9PpXg9hK/img.png)



```bash
query = """
WITH table1 AS (
    SELECT
        fullVisitorId, visitNumber, visitId, date, hits.eCommerceAction.action_type, hits.hour, hits.minute, product.productSKU, product.v2ProductCategory, product.v2ProductName, product.productPrice,
        ROW_NUMBER() OVER (
            PARTITION BY visitId, product.productSKU
            ORDER BY hits.eCommerceAction.action_type DESC
        ) AS rn
        #방문자와 제품SKU를 파티션으로 묶은 후 방문자가 본 제품SKU별 최종 행동유형이 무엇인지 파악하기 위함
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201*`,
    UNNEST(hits) AS hits,
    UNNEST(hits.product) AS product
    WHERE visitNumber > 1 AND hits.eCommerceAction.action_type != '0'
),

filtered_table1 AS (
    SELECT
        fullVisitorId, visitNumber, visitId, date, action_type,
        hour, minute, productSKU, v2ProductCategory, v2ProductName, productPrice
    FROM table1
    WHERE rn = 1
)

SELECT * FROM filtered_table1
ORDER BY visitId, productSKU
"""
action_data = client.query(query).result().to_dataframe()
```

```bash
action_cart_data = action_data.loc[action_data['action_type']=='3'].reset_index()
#최종 행동유형이 "제품 장바구니에 담기"인 경우
action_pur_data = action_data.loc[action_data['action_type']=='6'].reset_index()
#최종 행동유형이 "제품 결제 완료"인 경우

mer_df = pd.merge(action_cart_data, action_pur_data, how='outer', on='fullVisitorId')
mer_df = mer_df[(mer_df['date_x'] < mer_df['date_y']) & (mer_df['visitId_x'] != mer_df['visitId_y']) & (mer_df['productSKU_x'] == mer_df['productSKU_y'])]
#같은 고객, 다른 날짜,  다른 세션 식별자, 같은 제품SKU인 경우를 필터링

mer_df['date_y'] = pd.to_datetime(mer_df['date_y'])
mer_df['date_x'] = pd.to_datetime(mer_df['date_x'])
mer_df['date_dif'] = mer_df['date_y'] - mer_df['date_x']
#한 고객이 제품을 장바구니에 넣은 후 세션을 종료하고, 다시 방문해 같은 제품을 결제 완료 했을 경우, 재방문 후 결제완료까지 걸리는 기간을 계산

mer_df = mer_df.drop_duplicates(subset='index_x', keep='first')
#같은 물품을 2번 구입했을 경우 index_x가 2번 나타나기에 장바구니에 넣은 후 이후 재방문하여 결제완료를 했을 경우 그 기간이 빠른 경우만 남김
```

| index_x | fullVisitorId | visitNumber_x | visitId_x | date_x | action_type_x | hour_x | minute_x | productSKU_x | v2ProductCategory_x | v2ProductName_x | productPrice_x | index_y | visitNumber_y | visitId_y | date_y | action_type_y | hour_y | minute_y | productSKU_y | v2ProductCategory_y | v2ProductName_y | productPrice_y | date_dif |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 23829 | 0003450834640354121 | 4 | 1473414794 | 2016-09-09 00:00:00 | 3 | 2 | 57 | GGOEGFKQ020399 | Home/Shop by Brand/Google/ | Google Laptop and Cell Phone Stickers | 1990000 | 41556 | 8 | 1476825602 | 2016-10-18 00:00:00 | 6 | 14 | 53 | GGOEGFKQ020399 | Office | Google Laptop and Cell Phone Stickers | 1990000 | 39 |
| 126046 | 0014262055593378383 | 5 | 1500099419 | 2017-07-14 00:00:00 | 3 | 23 | 20 | GGOEGBPB021199 | ${escCatTitle} | Google Slim Utility Travel Bag | 11990000 | 126116 | 6 | 1500139462 | 2017-07-15 00:00:00 | 6 | 10 | 45 | GGOEGBPB021199 | Bags | Google Slim Utility Travel Bag | 8390000 | 1 |
| 64990 | 0024314485741511650 | 3 | 1481992053 | 2016-12-17 00:00:00 | 3 | 8 | 28 | GGOEGBRB013899 | Home/Bags/Backpacks/ | Google Laptop Backpack | 79990000 | 68275 | 4 | 1482891024 | 2016-12-27 00:00:00 | 6 | 18 | 26 | GGOEGBRB013899 | Bags | Google Laptop Backpack | 79990000 | 10 |
| 21421 | 0037518757923116572 | 2 | 1472916929 | 2016-09-03 00:00:00 | 3 | 8 | 45 | GGOEGDHB072099 | Home/Drinkware/ | Google Insulated Stainless Steel Bottle | 15990000 | 27270 | 4 | 1474002793 | 2016-09-15 00:00:00 | 6 | 22 | 30 | GGOEGDHB072099 | (not set) | Google Insulated Stainless Steel Bottle | 15990000 | 12 |
| 60469 | 0047509547902079820 | 2 | 1481310471 | 2016-12-09 00:00:00 | 3 | 11 | 19 | GGOEGFKQ020399 | Home/Accessories/ | Google Laptop and Cell Phone Stickers | 1990000 | 75974 | 5 | 1485544215 | 2017-01-27 00:00:00 | 6 | 12 | 2 | GGOEGFKQ020399 | Office | Google Laptop and Cell Phone Stickers | 1590000 | 49 |
| 60475 | 0047509547902079820 | 2 | 1481310471 | 2016-12-09 00:00:00 | 3 | 11 | 19 | GGOEGPJC203399 | Home/Accessories/ | Crunch Noise Dog Toy | 4990000 | 75977 | 5 | 1485544215 | 2017-01-27 00:00:00 | 6 | 12 | 2 | GGOEGPJC203399 | Accessories | Crunch Noise Dog Toy | 3990000 | 49 |
| 5310 | 0060402135554655326 | 2 | 1470762130 | 2016-08-09 00:00:00 | 3 | 10 | 11 | GGOEGAYR023499 | Home/Lifestyle/ | Straw Beach Mat | 9990000 | 6002 | 3 | 1470848645 | 2016-08-10 00:00:00 | 6 | 10 | 17 | GGOEGAYR023499 | (not set) | Straw Beach Mat | 9990000 | 1 |
| 5311 | 0060402135554655326 | 2 | 1470762130 | 2016-08-09 00:00:00 | 3 | 10 | 25 | GGOEGBMJ013399 | Home/Lifestyle/ | Sport Bag | 4990000 | 6006 | 3 | 1470848645 | 2016-08-10 00:00:00 | 6 | 10 | 17 | GGOEGBMJ013399 | (not set) | Sport Bag | 4990000 | 1 |
| 5312 | 0060402135554655326 | 2 | 1470762130 | 2016-08-09 00:00:00 | 3 | 10 | 10 | GGOEGCBB074199 | Home/Electronics/ | Google Car Clip Phone Holder | 6990000 | 35305 | 6 | 1475444991 | 2016-10-02 00:00:00 | 6 | 14 | 57 | GGOEGCBB074199 | (not set) | Google Car Clip Phone Holder | 5590000 | 54 |
| 5314 | 0060402135554655326 | 2 | 1470762130 | 2016-08-09 00:00:00 | 3 | 10 | 9 | GGOEGFQB013799 | Home/Electronics/ | Compact Selfie Stick | 8990000 | 6010 | 3 | 1470848645 | 2016-08-10 00:00:00 | 6 | 10 | 17 | GGOEGFQB013799 | (not set) | Compact Selfie Stick | 8990000 | 1 |

위 코드를 사용하여 고객별 제품에 대한 최종 행동유형과 방문 날짜를 추출합니다.
고객이 제품을 장바구니에 넣고 이탈한 후 다시 돌아와 제품을 결제 완료하였을 경우, 해당 기이 얼마나 되는지 확인합니다.

제품을 장바구니에 넣고 이탈한 고객의 대부분은 하루 뒤에 다시 방문하여 해당 제품을 구매하는데, 이를 통하여 제품을 장바구니에 넣고 이탈한 고객에게 하루 뒤에 리마인드 메시지를 발신하는 게 좋다고 판단할 수 있습니다.

![https://blog.kakaocdn.net/dn/0QJIO/btsITl7VQ1c/pM1Qou6STPcZ3XMHRGNLC1/img.png](https://blog.kakaocdn.net/dn/0QJIO/btsITl7VQ1c/pM1Qou6STPcZ3XMHRGNLC1/img.png)

Google Merchadise Store 웹페이지 홈 화면엔 제품 프로모션을 제공하고 있습니다.
프로모션을 클릭하는 사람이 진짜로 해당 카테고리의 제품을 구매하는지 확인해 보고자 합니다.

![https://blog.kakaocdn.net/dn/bLP74J/btsITf7KBFU/nP9WUxXPeeKfZW4KwFaQBk/img.png](https://blog.kakaocdn.net/dn/bLP74J/btsITf7KBFU/nP9WUxXPeeKfZW4KwFaQBk/img.png)



```bash
query = """
WITH table1 AS (
    SELECT
      visitId,
      ARRAY_AGG(DISTINCT promoName) AS promoName,
      MAX(IF(CAST(promoIsView AS STRING) = 'true', 1, 0)) AS promoIsView,
      MAX(IF(CAST(promoIsClick AS STRING) = 'true', 1, 0)) AS promoIsClick
    FROM (
      SELECT
        visitId,
        promotion.promoName AS promoName,
        promotionActionInfo.promoIsView AS promoIsView,
        promotionActionInfo.promoIsClick AS promoIsClick
      FROM
        `bigquery-public-data.google_analytics_sample.ga_sessions_201*`,
        UNNEST(hits) AS hits,
        UNNEST(hits.promotion) AS promotion
      WHERE CAST(promotionActionInfo.promoIsView AS STRING) = 'true'
    )
    GROUP BY visitId

    UNION ALL

    SELECT
      visitId,
      ARRAY_AGG(DISTINCT promoName) AS promoName,
      MAX(IF(CAST(promoIsView AS STRING) = 'true', 1, 0)) AS promoIsView,
      MAX(IF(CAST(promoIsClick AS STRING) = 'true', 1, 0)) AS promoIsClick
    FROM (
      SELECT
        visitId,
        promotion.promoName AS promoName,
        promotionActionInfo.promoIsView AS promoIsView,
        promotionActionInfo.promoIsClick AS promoIsClick
      FROM
        `bigquery-public-data.google_analytics_sample.ga_sessions_201*`,
        UNNEST(hits) AS hits,
        UNNEST(hits.promotion) AS promotion
      WHERE CAST(promotionActionInfo.promoIsClick AS STRING) = 'true'
    )
    GROUP BY visitId
),

table2 AS (
    SELECT
        visitId,
        ARRAY_AGG(DISTINCT IF(action_type = '1', v2ProductCategory, NULL) IGNORE NULLS) AS `제품 리스트 클릭`,
        ARRAY_AGG(DISTINCT IF(action_type = '2', v2ProductCategory, NULL) IGNORE NULLS) AS `제품 상세보기`,
        ARRAY_AGG(DISTINCT IF(action_type = '3', v2ProductCategory, NULL) IGNORE NULLS) AS `장바구니 담기`,
        ARRAY_AGG(DISTINCT IF(action_type = '4', v2ProductCategory, NULL) IGNORE NULLS) AS `장바구니 삭제`,
        ARRAY_AGG(DISTINCT IF(action_type = '5', v2ProductCategory, NULL) IGNORE NULLS) AS `결제`,
        ARRAY_AGG(DISTINCT IF(action_type = '6', v2ProductCategory, NULL) IGNORE NULLS) AS `결제완료`
    FROM (
      SELECT
        visitId,
        product.v2ProductCategory AS v2ProductCategory,
        hits.eCommerceAction.action_type AS action_type
      FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201*`,
      UNNEST(hits) AS hits,
      UNNEST(hits.product) AS product
    )
    GROUP BY visitId
)

SELECT * FROM table1 t1
LEFT JOIN table2 t2
ON t1.visitId = t2.visitId;
"""
visit_promo2 = client.query(query).result().to_dataframe()
```

| visitId | promoName | promoIsView | promoIsClick | visitId_1 | 제품 리스트 클릭 | 제품 상세보기 | 장바구니 담기 | 장바구니 삭제 | 결제 | 결제완료 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1478879638 | ['YouTube Brand'] | 0 | 1 | 1478879638 | [] | [] | [] | [] | [] | [] |
| 1478904825 | ['Google Brand'] | 0 | 1 | 1478904825 | [] | [] | [] | [] | [] | [] |
| 1478864525 | ['YouTube Brand'] | 0 | 1 | 1478864525 | [] | [] | [] | [] | [] | [] |
| 1478868243 | ['Drinkware'] | 0 | 1 | 1478868243 | [] | [] | [] | [] | [] | [] |
| 1478890973 | ['YouTube Brand' 'Office'] | 0 | 1 | 1478890973 | [] | [] | [] | [] | [] | [] |
| 1478892727 | ['Womens T-Shirts' 'Drinkware'] | 0 | 1 | 1478892727 | [] | [] | [] | [] | [] | [] |
| 1475356512 | ['Andriod Brand' 'Office'] | 0 | 1 | 1475356512 | [] | [] | [] | [] | [] | [] |
| 1475357079 | ['Google Brand' 'Office' 'Andriod Brand' 'YouTube Brand'] | 0 | 1 | 1475357079 | ['Home/Shop by Brand/YouTube/'] | ['Home/Shop by Brand/YouTube/'] | [] | [] | [] | [] |
| 1475320538 | ['Apparel' 'Womens T-Shirts'] | 0 | 1 | 1475320538 | ['Home/Drinkware/' 'Home/Office/' "Home/Apparel/Women's/"] | ['Home/Drinkware/' "Home/Apparel/Women's/" 'Home/Office/'] | ["Home/Apparel/Women's/" 'Home/Drinkware/' '${escCatTitle}' 'Home/Office/'] | ['(not set)'] | [] | [] |

```bash
visit_promo2['제품 리스트 클릭 여부'] = ''
visit_promo2['제품 상세보기 여부'] = ''
visit_promo2['장바구니 담기 여부'] = ''
visit_promo2['장바구니 삭제 여부'] = ''
visit_promo2['결제 여부'] = ''
visit_promo2['결제완료 여부'] = ''

for i in range(len(visit_promo2)):
    for j in range(len(visit_promo2['promoName'][i])):
        for l in range(len(visit_promo2['결제완료'][i])):
            if len(str(re.search(str(visit_promo2['promoName'][i][j]), str(visit_promo2['결제완료'][i][l])))) > 30:
                visit_promo2['결제완료 여부'][i] = visit_promo2['결제완료 여부'][i] + \
                    '[' + str(visit_promo2['promoName'][i][j]) + ']'
            else:
                pass
        for l in range(len(visit_promo2['제품 리스트 클릭'][i])):
            if len(str(re.search(str(visit_promo2['promoName'][i][j]), str(visit_promo2['제품 리스트 클릭'][i][l])))) > 30:
                visit_promo2['제품 리스트 클릭 여부'][i] = visit_promo2['제품 리스트 클릭 여부'][i] + \
                    '[' + str(visit_promo2['promoName'][i][j]) + ']'
            else:
                pass
        for l in range(len(visit_promo2['제품 상세보기'][i])):
            if len(str(re.search(str(visit_promo2['promoName'][i][j]), str(visit_promo2['제품 상세보기'][i][l])))) > 30:
                visit_promo2['제품 상세보기 여부'][i] = visit_promo2['제품 상세보기 여부'][i] + \
                    '[' + str(visit_promo2['promoName'][i][j]) + ']'
            else:
                pass
        for l in range(len(visit_promo2['장바구니 담기'][i])):
            if len(str(re.search(str(visit_promo2['promoName'][i][j]), str(visit_promo2['장바구니 담기'][i][l])))) > 30:
                visit_promo2['장바구니 담기 여부'][i] = visit_promo2['장바구니 담기 여부'][i] + \
                    '[' + str(visit_promo2['promoName'][i][j]) + ']'
            else:
                pass
        for l in range(len(visit_promo2['장바구니 삭제'][i])):
            if len(str(re.search(str(visit_promo2['promoName'][i][j]), str(visit_promo2['장바구니 삭제'][i][l])))) > 30:
                visit_promo2['장바구니 삭제 여부'][i] = visit_promo2['장바구니 삭제 여부'][i] + \
                    '[' + str(visit_promo2['promoName'][i][j]) + ']'
            else:
                pass
        for l in range(len(visit_promo2['결제'][i])):
            if len(str(re.search(str(visit_promo2['promoName'][i][j]), str(visit_promo2['결제'][i][l])))) > 30:
                visit_promo2['결제 여부'][i] = visit_promo2['결제 여부'][i] + \
                    '[' + str(visit_promo2['promoName'][i][j]) + ']'
            else:
                pass
```

| visitId | promoName | promoIsView | promoIsClick | visitId_1 | 제품 리스트 클릭 | 제품 상세보기 | 장바구니 담기 | 장바구니 삭제 | 결제 | 결제완료 | 제품 리스트 클릭 여부 | 제품 상세보기 여부 | 장바구니 담기 여부 | 장바구니 삭제 여부 | 결제 여부 | 결제완료 여부 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1478879638 | ['YouTube Brand'] | 0 | 1 | 1478879638 | [] | [] | [] | [] | [] | [] |  |  |  |  |  |  |
| 1478904825 | ['Google Brand'] | 0 | 1 | 1478904825 | [] | [] | [] | [] | [] | [] |  |  |  |  |  |  |
| 1478864525 | ['YouTube Brand'] | 0 | 1 | 1478864525 | [] | [] | [] | [] | [] | [] |  |  |  |  |  |  |
| 1478868243 | ['Drinkware'] | 0 | 1 | 1478868243 | [] | [] | [] | [] | [] | [] |  |  |  |  |  |  |
| 1478890973 | ['YouTube Brand' 'Office'] | 0 | 1 | 1478890973 | [] | [] | [] | [] | [] | [] |  |  |  |  |  |  |
| 1478892727 | ['Womens T-Shirts' 'Drinkware'] | 0 | 1 | 1478892727 | [] | [] | [] | [] | [] | [] |  |  |  |  |  |  |
| 1475356512 | ['Andriod Brand' 'Office'] | 0 | 1 | 1475356512 | [] | [] | [] | [] | [] | [] |  |  |  |  |  |  |
| 1475357079 | ['Google Brand' 'Office' 'Andriod Brand' 'YouTube Brand'] | 0 | 1 | 1475357079 | ['Home/Shop by Brand/YouTube/'] | ['Home/Shop by Brand/YouTube/'] | [] | [] | [] | [] |  |  |  |  |  |  |
| 1475320538 | ['Apparel' 'Womens T-Shirts'] | 0 | 1 | 1475320538 | ['Home/Drinkware/' 'Home/Office/' "Home/Apparel/Women's/"] | ['Home/Drinkware/' "Home/Apparel/Women's/" 'Home/Office/'] | ["Home/Apparel/Women's/" 'Home/Drinkware/' '${escCatTitle}' 'Home/Office/'] | ['(not set)'] | [] | [] | [Apparel] | [Apparel] | [Apparel] |  |  |  |

```bash
for category in ['Apparel','Office','Drinkware']:
    cc = visit_promo2[['visitId','promoName', 'promoIsClick', '결제완료 여부']]
    cc.loc[~cc['결제완료 여부'].str.contains(category, case=False, na=False), '결제완료 여부'] = '0'
    cc.loc[cc['결제완료 여부'].str.contains(category, case=False, na=False), '결제완료 여부'] = '1'
    variable = pd.merge(cc, variable, how='left')
    cct = visit_promo2[['visitId','promoName', 'promoIsClick', '결제완료 여부']]

    cct.loc[~cct['결제완료 여부'].str.contains(category, case=False, na=False), '결제완료 여부'] = '0'
    cct.loc[cct['결제완료 여부'].str.contains(category, case=False, na=False), '결제완료 여부'] = '1'

    cct['targetpromoClick'] = '0'
    cct.loc[(cct['promoName'].astype(str).str.contains(category, case=False, na=False)) & (cct['promoIsClick']==1), 'targetpromoClick'] = '1'#길이 6912

    c = cct.loc[cct['targetpromoClick'] == '1']
    cct_filtered = cct.drop(c.index.tolist())

    cct = pd.concat([c,cct_filtered.sample(n=min(len(c), len(cct_filtered)), random_state=1002)])

    print(len(cct.loc[(cct['결제완료 여부']=='1') & (cct['targetpromoClick']=='1')]) / len(cct.loc[cct['targetpromoClick']=='1']))
    print(len(cct.loc[(cct['결제완료 여부']=='1') & (cct['targetpromoClick']=='0')]) / len(cct.loc[cct['targetpromoClick']=='0']))

    contingency_table = pd.crosstab(cct['targetpromoClick'], cct['결제완료 여부'])
    chi2, p_value, dof, expected = chi2_contingency(contingency_table)
    print(f"{chi2:.2f}")
    print(f"{p_value:.4f}")
	print(f"{expected}")
```

프로모션에 대한 클릭과 해당 카테고리 제품에 대한 구매에 대한 연관성에 대한 검정은 카이제곱 독립성 검정을 통해 수행합니다.

프로모션을 클릭한 경험이 있는 고객이 어떤 카테고리 제품의 프로모션을 클릭했는지, 또한 어떤 카테고리 제품에 대한 결제 완료를 수행하였는지 나타내는 데이터 프레임을 생성하고, 해당 데이터를 통해 카이제곱 독립성 검정을 수행합니다.

![https://blog.kakaocdn.net/dn/by4LoR/btsISJanXEW/Oh3gldwckrgxXWTScbAr9K/img.png](https://blog.kakaocdn.net/dn/by4LoR/btsISJanXEW/Oh3gldwckrgxXWTScbAr9K/img.png)

프로모션 및 구매완료 횟수가 높은 의류, 사무용품, 드링크 웨어에 대해 검정을 수행했으며, 그 결과 의류와 사무용품은 프로모션 클릭과 구매 사이에 연관성은 존재하지 않았고, 드링크 웨어의 경우 해당 프로모션을 클릭한 사람이 제품을 구매할 확률이 유의하게 높다는 것을 확인할 수 있었습니다.

이를 통하여 드링크 웨어 카테고리의 제품을 프로모션에 노출 시키는 것이 매출 상승에 효과가 있다고 판단할 수 있습니다.

![Untitled 1](https://github.com/user-attachments/assets/712e8f69-e230-4bd8-aed3-45b178b527f9)



```bash
query = """
SELECT
    fullVisitorId,
    visitId,
    visitNumber,
    hits.hitNumber AS hitNumber,
    CASE
        WHEN hits.eCommerceAction.action_type = '0' THEN NULL
        ELSE product.v2ProductName
    END AS ProductName,

    CASE
        WHEN hits.eCommerceAction.action_type = '0' THEN NULL
        ELSE product.v2ProductCategory
    END AS ProductCategory,
    CASE
        WHEN hits.eCommerceAction.action_type = '0' THEN NULL
        ELSE product.productVariant
    END AS productVariant,
    CASE
        WHEN hits.eCommerceAction.action_type = '0' THEN NULL
        ELSE product.productBrand
    END AS productBrand,
    CASE
        WHEN hits.eCommerceAction.action_type = '0' THEN NULL
        ELSE product.ProductSKU
    END AS ProductSKU,
    CASE
        WHEN hits.eCommerceAction.action_type = '0' THEN NULL
        ELSE product.productQuantity
    END AS productQuantity,
    CASE
        WHEN hits.eCommerceAction.action_type = '0' THEN NULL
        ELSE product.productPrice END AS productPrice,
    hits.eCommerceAction.action_type AS action_type,
    date,
    hits.hour,
    geoNetwork.country,
    geoNetwork.region,
    geoNetwork.city,
    hits.transaction.transactionRevenue AS transactionRevenue,
    hits.appInfo.screenName AS screenName,
    hits.appInfo.landingScreenName,
    hits.appInfo.exitScreenName,
    device.browserSize AS browserSize,
    device.deviceCategory AS deviceCategory,
    trafficSource.adContent AS adContent,
    trafficSource.adwordsClickInfo.gclId AS gclId,
    trafficSource.adwordsClickInfo.page AS page,
    trafficSource.adwordsClickInfo.slot AS slot,
    trafficSource.campaign AS campaign,
    trafficSource.isTrueDirect AS isTrueDirect,
    trafficSource.keyword AS keyword,
    trafficSource.medium AS medium,
    trafficSource.source AS source,
    trafficSource.referralPath AS referralPath
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,UNNEST(hits) AS hits, UNNEST(hits.product) AS product
union all
SELECT
    fullVisitorId,
    visitId,
    visitNumber,
    hits.hitNumber AS hitNumber,
    CASE
        WHEN hits.eCommerceAction.action_type = '0' THEN NULL
        ELSE product.v2ProductName
    END AS ProductName,

    CASE
        WHEN hits.eCommerceAction.action_type = '0' THEN NULL
        ELSE product.v2ProductCategory
    END AS ProductCategory,
    CASE
        WHEN hits.eCommerceAction.action_type = '0' THEN NULL
        ELSE product.productVariant
    END AS productVariant,
    CASE
        WHEN hits.eCommerceAction.action_type = '0' THEN NULL
        ELSE product.productBrand
    END AS productBrand,
    CASE
        WHEN hits.eCommerceAction.action_type = '0' THEN NULL
        ELSE product.ProductSKU
    END AS ProductSKU,
    CASE
        WHEN hits.eCommerceAction.action_type = '0' THEN NULL
        ELSE product.productQuantity
    END AS productQuantity,
    CASE
        WHEN hits.eCommerceAction.action_type = '0' THEN NULL
        ELSE product.productPrice END AS productPrice,
    hits.eCommerceAction.action_type AS action_type,
    date,
    hits.hour,
    geoNetwork.country,
    geoNetwork.region,
    geoNetwork.city,
    hits.transaction.transactionRevenue AS transactionRevenue,
    hits.appInfo.screenName AS screenName,
    hits.appInfo.landingScreenName,
    hits.appInfo.exitScreenName,
    device.browserSize AS browserSize,
    device.deviceCategory AS deviceCategory,
    trafficSource.adContent AS adContent,
    trafficSource.adwordsClickInfo.gclId AS gclId,
    trafficSource.adwordsClickInfo.page AS page,
    trafficSource.adwordsClickInfo.slot AS slot,
    trafficSource.campaign AS campaign,
    trafficSource.isTrueDirect AS isTrueDirect,
    trafficSource.keyword AS keyword,
    trafficSource.medium AS medium,
    trafficSource.source AS source,
    trafficSource.referralPath AS referralPath
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,UNNEST(hits) AS hits, UNNEST(hits.product) AS product
"""
d_data = client.query(query).result().to_dataframe()
d_data.drop_duplicates()
d_data = d_data.drop_duplicates()
```

태블로를 이용한 대시보드를 작성해 보겠습니다. 태블로는 특정 테이블에 대한 고유값 카운트가 가능하기 때문에 엑셀 대시보드 작성 때처럼 형태를 변형하지 않고 RAW 데이터 형태 그대로 데이터를 불러오겠습니다.

![Untitled 2](https://github.com/user-attachments/assets/8e3d53e5-a289-42de-a31a-f1ab46a8866b)

태블로를 활용하여 작성한 대시보드입니다. 

이를 이용하여 분석을 수행해 보겠습니다.

![Untitled 3](https://github.com/user-attachments/assets/4b268257-6096-4812-985c-03734c2835bb)

데스크탑을 사용하는 고객은 주소를 직접 입력하거나 즐겨찾기를 통해 사이트를 방문하며, 평균 방문 횟수가 결제 금액이 높은 충성 고객이 많고, 결제 완료까지 가는 비율이 높습니다.

모바일을 사용하는 고객은 구글 검색을 통하여 방문하며, 평균 결제금액과 방문 횟수가 상대적으로 적은 첫 방문 고객이 많으며, 결제 완료까지 가는 비율이 낮습니다.

![Untitled 4](https://github.com/user-attachments/assets/61af5569-b107-420e-85cc-2d7d8770c28f)

고객의 유입 매체별 분석입니다. 대부분 고객은 오전 8시~오후2시 시간대에 사이트를 방문하지만, CPC(클릭 시 비용이 지불되는 광고를 통한 트래픽)의 경우 저녁 시간대 방문이 많기 때문에 저녁 시간대 광고 노출을 늘리는 것이 좋다고 판단됩니다.

링크를 통한 유입(Referral)의 경우 결제 완료 단계로 가는 비율이 2%로 매우 낮고 매출액도 낮기 때문에 만약 리퍼럴 마케팅을 위한 비용을 지불하고 있을 경우 마케팅을 하지 않는게 비용 측면에서 더 나을 수 있습니다. 특히나 Referral의 경우 유튜브를 통한 유입이 65%지만 이 중 결제 완료 단계까지 가는 경우는 0.002% 수준으로 매우 낮은 수준이었습니다.

![Untitled 5](https://github.com/user-attachments/assets/308724fc-c780-456d-8827-73c473ae959d)

2017년 7월 13일의 방문자 수는 다른 날과 비교해서 많지 않지만 매출액은 비정상 적으로 높은 것을 알 수 있습니다.

확인 결과 일부 의류 제품의 가격이 비정상적으로 높게 책정되었는데, 이는 Google Analytics의 오류로 인한 이상치라고 판단됩니다.

더욱이 토요일과 일요일마다 방문자 수가 급감하는 주기성이 확인됩니다. 
이는 직접 방문자와 검색 방문자가 주말에 급감하기 때문인데, 주말 프로모션을 진행하거나, 주말에 소비하는 경향이 있는 고객에게 타겟 광고를 게시하는 것 식의 전략이 필요할 것으로 생각됩니다.
