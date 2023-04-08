# 레시피

## 데이터 추가

### 만개레시피 크롤링

```java
from bs4 import BeautifulSoup
import requests
import pprint

token = "TOKEN"
server = "server address"
while 1:
    process = input("1:데이터 크롤링 2:삭제 3: 종료")

    if process == "1":
        base_url = "https://www.10000recipe.com"
        search = "/recipe/list.html?q="
        while 1:
            keyword  = input("검색어 입력:")

            search_url = base_url + search + keyword

            r = requests.get(search_url)

            soup = BeautifulSoup(r.text, "html.parser")

            items = soup.select(".common_sp_list_li")
            recipes = []
            for item in items:
                recipeinfo = dict()
                recipeinfo["name"] = item.select_one(".common_sp_caption_tit.line2").text
                url = item.select_one("a.common_sp_link")
                recipe_url = base_url + url.attrs['href']
                recipe = requests.get(recipe_url)
                print(recipe_url)
                recipe_soup = BeautifulSoup(recipe.text, "html.parser")
                main = recipe_soup.select_one("#main_thumbs").attrs['src']
                recipeinfo["id"] = recipe_url.split("/")[-1]
                recipeinfo["main"] = main
                info = ['0', '0', '0']

                if recipe_soup.select_one(".view2_summary_info1"):
                    info[0] = recipe_soup.select_one(".view2_summary_info1").text.split("인분")[0]

                if recipe_soup.select_one(".view2_summary_info2"):
                    if "분" in recipe_soup.select_one(".view2_summary_info2").text:
                        info[1] = recipe_soup.select_one(".view2_summary_info2").text.split("분")[0]
                        info[1] = info[1]
                    elif "시간" in recipe_soup.select_one(".view2_summary_info2").text:
                        info[1] = recipe_soup.select_one(".view2_summary_info2").text.split("시간")[0]
                        info[1] = str(int(info[1]) * 60)
                if recipe_soup.select_one(".view2_summary_info3"):
                    diff = recipe_soup.select_one(".view2_summary_info3").text
                    if diff == "중급":
                        info[2] = '3'
                    elif diff == "초급":
                        info[2] = '2'
                    elif diff == "아무나":
                        info[2] = '1'
                    elif diff == "상급":
                        info[2] = '4'
                recipeinfo["info"] = ','.join(info)

        # 재료
                if recipe_soup.select_one(".ready_ingre3"):
                    ingredient = []
                    for j_soup in  recipe_soup.select(f"#divConfirmedMaterialArea > ul"):
                        for k_soup in j_soup.select("li"):
                            edited = k_soup.text.split("\n구매\n")
                            temp = ""
                            if len(edited) > 1:
                                temp += edited[0].strip() + "::"
                                temp +=  edited[1].strip() + ""
                            else:
                                temp += edited[0].split("  ")[0] + " "
                                temp += edited[0].split("  ")[-1].strip()
                            ingredient.append(temp)
                    recipeinfo["ingredient"] = ingredient


                # 과정
                step = []
                for i in range(40):
                    detail = recipe_soup.select_one(f"#stepdescr{i}.media-body")
                    temp = ""
                    if detail:
                        temp +=' '.join(detail.text.split("\n")) + "---"
                    if recipe_soup.select_one(f"#stepimg{i} > img"):
                        temp += recipe_soup.select_one(f"#stepimg{i} > img").attrs['src']
                    if temp:
                        step.append(temp)
                recipeinfo["step"] = step
                #recipes.append(recipeinfo)
                pprint.pprint(recipeinfo)
                text = input("추가할까요? 1:네 2:아니오 3:다른 검색어 4:종료")
                if text == "1":
                    category = input("카테고리 입력(국/찌개, 반찬, 구이/볶음, 면, 디저트):")
                    recipeinfo["category"] = category

                    hashTagList = recipeinfo["name"] +"#"+ category + "#" + input("hash tag 입력: #으로 구분합니다 레시피 이름과 카테고리는 자동으로 추가됩니다")
                    recipeinfo["hashTagList"] = hashTagList

                    res = requests.post("http://{server}:8090/api/v1/recipe/getData2", json=recipeinfo)
                    print("status: " + str(res.status_code))
                elif text == "3" or text == "4":
                    break
            if text == "4":
                break
    elif process == "2":
        while 1:
            idOrName = int(input("1:id로 삭제 2:name으로 삭제 3: 종료"))
            if idOrName == 1:
                id = input("id입력:")
                res = requests.get(f"http://{server}:8090/api/v1/recipe/remove/id/{id}", headers={"Authorization":token})
                print("status: " + str(res.status_code))

            elif idOrName == 2:
                name = input("name입력:")
                res = requests.get(f"http:/{server}:8090/api/v1/recipe/remove/name/{name}", headers={"Authorization":token})
                print("status: " + str(res.status_code))
            elif idOrName == 3:
                break
    elif process == "3":
        break
```

- BeautifulSoup를 이용해 크롤링
- 만개레시피에서 검색과 해당 레시피를 서버API호출하여 저장할 수 있다.
- hash tag을 자신이 추가할 수 있다
- 추가한 데이터가 맘에 들지않으면 id나 이름으로 조회하여 삭제 할 수 있다.

### 공공데이터

- 식품의약품안전처의 조리식품의 레시피DB를 조회하는 OPEN-API를 사용했다.
- 해당 데이터에서 이미지, 조리과정등을 저장하고 재료정보는 재가공하여 DB에 저장했다.

## 레시피 조리과정

- SPRING DATA JPA를 이용해 해시태크, 각 과정에서 이미지와 설명을 조회했다.

## 아쉬운점

- 크롤링 속도가 느렸다.
