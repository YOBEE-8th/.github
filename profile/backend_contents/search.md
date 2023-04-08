# 검색

```java
//JpaRecipeRepository
public List<Recipe> pagenationSearchRecipe(PageSearchDto pageSearchDto) {
        int sortLogic = pageSearchDto.getSort();
        Boolean oredrBoolean = pageSearchDto.getOrder();
        int page = pageSearchDto.getPage();
        String keyword = pageSearchDto.getKeyword();

        String query = "select recipe from Recipe recipe LEFT JOIN" +
												" Review r ON recipe.recipeId = r.recipe.recipeId" +
												" LEFT JOIN HashTag h ON h.recipe.recipeId = recipe.recipeId "+
												" where h.tag like '%" + keyword + "%' GROUP BY recipe.recipeId ";
        String order = oredrBoolean ? "ASC" : "DESC";
        switch (sortLogic) {
            case 1:
                query += "ORDER BY COUNT(CASE WHEN r.content IS NOT NULL THEN 1 END)";
                break;
            case 2:
                query += "order by recipe.difficulty ";
                break;
            default:
                query += "order by recipe.recipeLikeCnt ";
                break;
        }
        query += order;

        return em.createQuery(query, Recipe.class)
                .setFirstResult((page) * 20)
                .setMaxResults(20)
                .getResultList();
    }
```

1. PageSearchDto에서 정렬할 방법과 검색할 키워드를 받는다.
   1. 오름차순/내림차순, 좋아요수, 난이도, 리뷰수
   2. Recipe의 HashTag에 키워드를 포함하면 조회되도록했다.
2. Recipe와 ManyToOne 관계인 Review와 HashTag를 LEFT JOIN 하고 GROUP BY 를 이용해 중복을 제거한다
3. 정렬한다
   1. 리뷰수순
      1. 완성된 리뷰만 카운트하기 위해 Review의 content가 null이 아닌 경우만 카운트했다
      2. 난이도 순
      3. 좋아요순
         1. 조회때 마다 좋아요 테이블까지 찾는건 불필요한 것 같아 Recipe Table에 좋아요수를 추가했다.

### 아쉬운점

- keyword를 setParams로 추가하지 못한점
- 완성된 리뷰수도 컬럼으로 추가하지 않은점

```java
public RecipesDto(Recipe recipe, HeaderDto headerDto) {
        this.recipeId = recipe.getRecipeId();
        this.imgUrl = recipe.getResultImage();
        this.title = recipe.getRecipeTitle();
        this.isAI = recipe.isAi();
        this.likeCnt = recipe.getRecipeLikeCnt();
        this.isLike = false;
        for (RecipeLike recipeLike : recipe.getRecipeLikeList()){
            if (recipeLike.getUser().getEmail().equals(headerDto.getUserName())){
                this.isLike = true;
                break;
            }
        }
        this.level = recipe.getDifficulty();
    }

}
```

조회한 Recipe를 JWT에 있는 유저의 이메일정보와 좋아요를 체크한 유저의 이메일을 비교해 좋아요를 눌렀는지 확인한다.

### 아쉬운점

- for문을 안쓰고 query로 좋아요 여부를 체크할 수 있는지 확인안해본것

### 느낀점

- 간단한 조회만 하다가 조건이 많이 붙은 query를 작성하니 자신감이 생긴것 같다.
- SpringDataJpa 편하긴 하지만 query가 근본있는 것같다.
- HashTag 테이블을 left join하여 찾는 과정이 시간이 많이 소모되는것 같아 최대 255자 문자열로 구분자를 이용해 recipe테이블에 column을 추가해서 조회해 봤는데 큰 시간차이가 없었다,
- for문으로 조회를 하다가 query문과 페이지네이션으로 속도를 올렸다.
