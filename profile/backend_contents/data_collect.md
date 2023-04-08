# 레시피 데이터 수집

레시피 데이터는 식약처에서 open API 로 제공되는 공공 데이터와

만개의 레시피에서 일부를 크롤링 하여 사용했습니다.

[데이터활용서비스](http://www.foodsafetykorea.go.kr/api/openApiInfo.do?menu_grp=MENU_GRP31&menu_no=661&show_cnt=10&start_idx=1&svc_no=COOKRCP01)

[요리를 즐겁게~ 만개의레시피](https://www.10000recipe.com/)

공공 데이터의 경우 일부만 선별해서 전처리 작업 진행하였습니다.

- 해상도 720p 이상의 이미지를 가지고 있는 레시피만 필터링 진행
- 레시피 재료 정보가 여러 형식으로 다르게 작성되어 있어, 문자열 처리 진행
