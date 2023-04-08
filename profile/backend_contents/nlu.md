# NLU

NLU(Natural Language Understanding)라는 기술이 등장한 취지는 기계에 기계어가 아닌, 사람이 평소 쓰는 자연스러운 표현 그대로 제공해도 기계가 알아들을 수 있도록 하자는 데 있습니다.

NLU 학습에 필요한 요소로는 크게 Intent, Action, Story가 있습니다.

1. Intent는 화자의 발화 의도를 구분합니다.
   - 안녕하세요, 안녕, 하이, 방가 → greet
   - 감사합니다, 감사, 땡쓰, 당케 → thanks
2. Action은 발화에 대한 답변 입니다.
   - 저도 반갑습니다.
   - 별말씀을요
3. Stroy는 Intent와 Action을 맵핑하여 대화의 흐름을 구성합니다.

   ```yaml
   stroy: greet
   steps:
     - intent: greet
     - action: utter_greet
   	.
   	.
   	.
   ```

---

NLU는 발화 의도를 파악할 때 학습된 Intent들 중에서 가장 높은 유사성을 가진 Intent를 발화 의도로 간주합니다.

Intent1, Intent2, Intent3가 있다고 가정할 때, 세 Intent의 유사도 합은 항상 1입니다.

이러한 구조 때문에 Intent 개수가 적을 경우 유사도 정확성에 영향을 미칠 수 있습니다.

본 프로젝트에서는 유사도 측정에 있어서 정확성을 높이기 위해 학습시키고자 하는 Intent외에도 일상 언어를 파악하는 Intent들도 추가하여 학습시켰습니다.
