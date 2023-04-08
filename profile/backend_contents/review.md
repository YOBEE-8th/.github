# 레시피 리뷰

```java
public interface ReviewRepository extends JpaRepository<Review, Long> {

    List<Review> findByrecipeAndContentNotNullOrderByCreatedAt(Recipe recipe);

    List<Review> findByUser(User user);

    List<Review> findByUserAndRecipeOrderByCreatedAt(User user, Recipe recipe);

    Review findByreviewImage(String reviewImage);

    Long countByrecipeAndContentNotNull(Recipe recipe);
}
```

- CRUD기능과 레시피로 조회하는 기능이 있다.
- 완성된 리뷰만 조회하기 위해 Content가 not null인 레시피만 조회한다.
- 리뷰사진은 S3에 저장하고 해당사진의 url은 DB에 저장한다.
- 사진 recipe{recipeId}\_ms단위의 현재시간.jpg 형식으로 이름을 변경했다.
