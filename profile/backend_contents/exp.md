# 0. Contents
1. 경험치

<br>

# 1. 경험치

```java
// user/service/ExperienceService.java

@Service
@RequiredArgsConstructor
public class ExperienceService {

    private final UserRepository userRepository;

    private final ExperienceRepository experienceRepository;

    private final RecipeRepository recipeRepository;

    public void createExperience (User user){
        Experience experience = new Experience();

//        Optional<User> optionalUser = Optional.ofNullable(userRepository.findByEmail(email));
//        if (!optionalUser.isPresent()){
//            throw new EntityNotFoundException("User not present in the database");
//        }
//        User user = optionalUser.get();

        experience.setUser(user);

        experience.setGrilledExp(0);
        experience.setSideExp(0);
        experience.setDessertExp(0);
        experience.setNoodleExp(0);
        experience.setSoupExp(0);

        experienceRepository.save(experience);

    }

    public ResponseUpdateExperienceDto updateExperience (UpdateExperienceDto updateExperienceDto, String email){

        ResponseUpdateExperienceDto responseUpdateExperienceDto = new ResponseUpdateExperienceDto();

        Long recipeId = updateExperienceDto.getRecipeId();

        Recipe recipe = recipeRepository.findById(recipeId);

        int difficulty = recipe.getDifficulty();

        String category = recipe.getCategory();

        Optional<User> optionalUser = Optional.ofNullable(userRepository.findByEmail(email));
        if (!optionalUser.isPresent()){
            throw new EntityNotFoundException("User not present in the database");
        }
        User user = optionalUser.get();

        Optional<Experience> optionalExperience = Optional.ofNullable(experienceRepository.findByUser(user));

        if (!optionalExperience.isPresent()){
            throw new EntityNotFoundException("Experience not present in the database");
        }

        Experience updatedExperience = optionalExperience.get();

        int nowGrilled = updatedExperience.getGrilledExp();
        int nowSide = updatedExperience.getSideExp();
        int nowNight = updatedExperience.getDessertExp();
        int nowNoodle = updatedExperience.getNoodleExp();
        int nowSoup = updatedExperience.getSoupExp();

        if (category.equals("구이/볶음")) {

            updatedExperience.setGrilledExp(nowGrilled+(difficulty*5));
            updatedExperience.setSideExp(nowSide);
            updatedExperience.setDessertExp(nowNight);
            updatedExperience.setNoodleExp(nowNoodle);
            updatedExperience.setSoupExp(nowSoup);

        } else if (category.equals("반찬")) {

            updatedExperience.setGrilledExp(nowGrilled);
            updatedExperience.setSideExp(nowSide+(difficulty*5));
            updatedExperience.setDessertExp(nowNight);
            updatedExperience.setNoodleExp(nowNoodle);
            updatedExperience.setSoupExp(nowSoup);
        }
        else if (category.equals("디저트")) {
            updatedExperience.setGrilledExp(nowGrilled);
            updatedExperience.setSideExp(nowSide);
            updatedExperience.setDessertExp(nowNight+(difficulty*5));
            updatedExperience.setNoodleExp(nowNoodle);
            updatedExperience.setSoupExp(nowSoup);
        }
        else if (category.equals("면")) {
            updatedExperience.setGrilledExp(nowGrilled);
            updatedExperience.setSideExp(nowSide);
            updatedExperience.setDessertExp(nowNight);
            updatedExperience.setNoodleExp(nowNoodle+(difficulty*5));
            updatedExperience.setSoupExp(nowSoup);

        } else if (category.equals("국/찌개")) {
            updatedExperience.setGrilledExp(nowGrilled);
            updatedExperience.setSideExp(nowSide);
            updatedExperience.setDessertExp(nowNight);
            updatedExperience.setNoodleExp(nowNoodle);
            updatedExperience.setSoupExp(nowSoup+(difficulty*5));
        }

        experienceRepository.save(updatedExperience);

        responseUpdateExperienceDto.setGrilledExp(updatedExperience.getGrilledExp());
        responseUpdateExperienceDto.setDessertExp(updatedExperience.getDessertExp());
        responseUpdateExperienceDto.setNoodleExp(updatedExperience.getNoodleExp());
        responseUpdateExperienceDto.setSoupExp(updatedExperience.getSoupExp());
        responseUpdateExperienceDto.setSideExp(updatedExperience.getSideExp());

        responseUpdateExperienceDto.setUpCategory(category);
        responseUpdateExperienceDto.setUpExp(difficulty*5);

        return responseUpdateExperienceDto;

    }

    public ResponseGetExperienceDto getExperience (String email){

        ResponseGetExperienceDto responseGetExperienceDto = new ResponseGetExperienceDto();

        Optional<User> optionalUser = Optional.ofNullable(userRepository.findByEmail(email));
        if (!optionalUser.isPresent()){
            throw new EntityNotFoundException("User not present in the database");
        }
        User user = optionalUser.get();

        Optional<Experience> optionalExperience = Optional.ofNullable(experienceRepository.findByUser(user));

        if (!optionalExperience.isPresent()){
            throw new EntityNotFoundException("Experience not present in the database");
        }

        Experience nowExperience = optionalExperience.get();

        responseGetExperienceDto.setGrilledExp(nowExperience.getGrilledExp());
        responseGetExperienceDto.setDessertExp(nowExperience.getDessertExp());
        responseGetExperienceDto.setNoodleExp(nowExperience.getNoodleExp());
        responseGetExperienceDto.setSoupExp(nowExperience.getSoupExp());
        responseGetExperienceDto.setSideExp(nowExperience.getSideExp());

        return responseGetExperienceDto;

    }

}
```