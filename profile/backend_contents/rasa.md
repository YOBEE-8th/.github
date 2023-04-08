# Rasa

Rasa는 텍스트 및 음성 기반 대화를 자동화하는 오픈 소스 머신 러닝 프레임워크 입니다.

사용자의 명령이나 질문에 대한 발화의도 파악를 파악하고 그에 맞는 답변을 반환하는 용도로 활용하였습니다.

본 프로젝트에서는 pre-trained 한국어 모델(ko_pipeline)을 사용하였으며, 우리 앱의 목적에 맞게 사용할 수 있도록 튜닝하였습니다.

![Untitled](https://github.com/YOBEE-8th/.github/blob/main/profile/backend_contents/img/rasa.png)

## 설명

```python
# type: 페이지 이동
    if 'page' in rasa_response[0]['text']:
        result_type = '0'
        result_message = '페이지 이동 명령'
        if rasa_response[0]['text'] == "first_page":
            result_text = '0'
            result_message += "( << )"
        elif rasa_response[0]['text'] == "previous_page":
            result_text = '1'
            result_message += "( < )"
        elif rasa_response[0]['text'] == "next_page":
            result_text = '2'
            result_message += "( > )"
        elif rasa_response[0]['text'] == "last_page":
            result_text = '3'
            result_message += "( >> )"
        else:
            result_text = '4' # current_page
            result_message += "( ● )"
```

```bash
Q: 다음 페이지로 이동해줘
Rasa: next_page
```

- Rasa에서 발화의도 파악 후 내놓은 답변은 next_page 입니다.
- 답변에 ‘page’ 라는 문자열이 포함되므로
  - result_type은 0으로 설정합니다.
  - result_text는 2으로 설정합니다.
  - result_message는 “페이지 이동 명령 ( > )”으로 설정합니다.

타이머 설정

```python
# type: 타이머 설정
    elif 'timer' in rasa_response[0]['text']:
        result_type = '2'
        result_message = "타이머 설정 명령"
        result_text = "timer"
```

```bash
Q: 타이머 설정해줘
Rasa: timer
```

- Rasa에서 발화의도 파악 후 내놓은 답변은 timer 입니다.
- 답변에 ‘timer’ 라는 문자열이 포함되므로
  - result_type은 2으로 설정합니다.
  - result_text은 ‘timer’로 설정합니다.
  - result_message는 “타이머 설정 명령”으로 설정합니다.

재료 질문

```python
 # type: 재료 질문
    elif '재료:' in rasa_response[0]['text']:
        ingredient = rasa_response[0]['text'].replace('재료:','')
        ingredient = ingredient.replace(' ', '')
        result_type = '1'
        result_message = "재료질문 답변"

        # Spring 서버 API 주소
        spring_url = "http://${배포서버ip:port}/api/v1/AI/recipe/"+ str(recipeId) +"/ingredient"
        # HTTP 요청 헤더 설정
        headers = {"Authorization": token}
        # 요청 보내기
        spring_response = requests.get(spring_url, headers=headers)
        # 응답 데이터 가져오기
        spring_response = spring_response.json()
        ingredients = spring_response['data']

        for ingredient_name, ingredient_weight in ingredients.items():
            ingredient_name = ingredient_name.replace(' ', '')

            if ingredient in ingredient_name:
								.
								.
								.
                result_text = ingredient_name + ' ' + ingredient_weight +' 넣어주세요'
                break
        else:
            result_text = ingredient + ' 들어가지 않아요.'
```

```bash
Q: 소금 얼마나 넣어야 돼?
Rasa: 재료:소금
```

- Rasa에서 발화의도 파악 후 내놓은 답변은 재료:소금 입니다.
- 답변에 ‘재료:’ 라는 문자열이 포함되므로
  - result_type은 1으로 설정합니다.
  - 재료: 뒤에 오는 단어(소금)가 해당 레시피에서 재료로 쓰이는지 확인합니다.
    - 소금이 재료로 쓰인다면, result_text은 ‘소금 5그램 넣어주세요’로 설정합니다.
    - 소금이 재료로 쓰이지 않으면, result_text는 ‘소금 들어가지 않아요.’로 설정합니다.
  - result_message는 “재료질문 답변”으로 설정합니다.

학습 외 질문

```python
# type: 학습 외 질문
    else:
        result_type = '1'
        result_message = "ChatGPT 답변"

        spring_url = "http://${배포서버ip:port}/api/v1/recipe/unresolved"
        headers = {"Authorization": token, "Content-Type": "application/json"}
        data = {"recipeId": recipeId, "content": message}
        spring_response = requests.post(spring_url, json=data, headers=headers)
        spring_response = spring_response.json()

        # ChatGPT 설정
        .
				.
				.

        result_text = completions.choices[0].text.replace('\n','')
```

```python
Q: 오늘 날씨 어때?
Rasa: 죄송해요. 답변할 수 없는 질문입니다.
ChatGPT: 오늘은 화창한 날씨입니다.
```

- Rasa에서 발화의도 파악 후 내놓은 답변은 죄송해요. 답변할 수 없는 질문입니다. 입니다.
- 학습 외 질문으로 분류되므로
  - result_type은 1으로 설정합니다.
  - result_text는 오늘 날씨 어때? 에 대한 ChatGPT의 답변으로 설정합니다.
  - result_message는 "ChatGPT 답변"으로 설정합니다.
  - 학습 외 질문은 추가 학습을 위해 미해결 질문 테이블에 추가합니다.
