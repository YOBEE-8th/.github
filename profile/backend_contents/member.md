# 0. Contents
1. 로그아웃
2. 회원탈퇴

<br>

# 1. 로그아웃

```java
// user/service/UserService.java

// 로그아웃

public UserTypeDto logout(String email){

        Optional<User> optionalUser = Optional.ofNullable(userRepository.findByEmail(email));
        if (!optionalUser.isPresent()){
            throw new EntityNotFoundException("User not present in the database");
        }
        User user = optionalUser.get();

        user.setFcmToken(null);
        UserTypeDto userTypeDto = new UserTypeDto();
        userTypeDto.setType(user.getType());

        userRepository.save(user);

        return userTypeDto;
    }

```

<br>

# 2. 회원탈퇴

```java
// user/service/UserService.java

// 회원탈퇴

public EmailDto withdrawal(String email){

        Optional<User> optionalUser = Optional.ofNullable(userRepository.findByEmail(email));
        if (!optionalUser.isPresent()){
            throw new EntityNotFoundException("User not present in the database");
        }
        User user = optionalUser.get();
        EmailDto emailDto = new EmailDto();
        emailDto.setEmail(user.getEmail());
        String url = user.getProfileImage();

        if (!(url.equals(default_img)) && userRepository.findByProfileImage(url).size() == 1) {
            s3Service.delete(user.getProfileImage().split("/")[user.getProfileImage().split("/").length - 1]);
        }

        List<RecipeLike> recipeLikeList = recipeLikeRepository.findByUser(user);
        for (RecipeLike recipeLike : recipeLikeList){
            Recipe recipe = recipeLike.getRecipe();
            int cnt = recipe.getRecipeLikeCnt();
            recipe.setRecipeLikeCnt(cnt - 1);
            recipeRepository.save(recipe);
        }

        List<ReviewLike> reviewLikeList = reviewLikeRepository.findByUser(user);
        for(ReviewLike reviewLike : reviewLikeList){
            Review review = reviewLike.getReview();
            int cnt = review.getReviewLikeCnt();
            review.setReviewLikeCnt(cnt - 1);
            reviewRepository.save(review);
        }

        userRepository.delete(user);

        return emailDto;
    }
```