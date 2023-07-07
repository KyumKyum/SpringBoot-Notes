# Annotations

Spring에서 사용되는 Annotation들을 정리해두었습니다. 공부하는대로 추가할 것입니다.

## @SpringBootApplication

- main에서 실행할 객체, 즉 runApplication<T> 중 T에 들어갈 객체에 해당하는 Bean에 달리는 Annotation이다.
- Starting Point라고 생각하기.

## @RequestMapping

- 기본적인 REST API에 해당하는 Endpoint를 지정할 때 사용되고, Http Method를 지정한다.
- URL을 기준으로 Mapping을 부여한다.
- value, method가 parameter로 사용되며, value는 endpoint, method는 사용될 http method가 들어간다.
  ```kotlin
  @RequestMapping(value = "/api/queryUser", method = RequestMethod.GET)
  @RequestMapping(value = "/api/addUser", method = RequestMethod.POST)
  ```

이는 다음과 같이 @GetMapping(), @PostMapping()등을 활용하여 다음과 같이 나눌 수 있다.

```kotlin
@RequestMapping(value = "/api")
 class TestController {
  @GetMapping(value = "/queryUser")
  fun TestGet(): String {...}

	@PostMapping(value = "/addUSer")
  fun TestPost(): String {...}
}
```

- @RequestMapping은 Class, Method 모두에 사용 가능하고, @{…}Mapping와 같이 세부적으로 나누는 Mapping의 경우는 Method에만 사용이 가능하다.

## @RequestParam()

- Request의 parameter 값을 받아온다. 예를 들어,

```kotlin
@GetMapping("/api/checkId")
fun checkId: String (@RequestParam(”id”) String id) {
  ...
}
```

- 다음과 같은 요청을 수행하기 위한 Url은 해당 parameter값이 없으면 404 Bad Request 오류가 나온다.
- …/api/checkId?id=kyumkyum 와 같이 parameter를 전달하면, 해당 id에 parameter값이 저장된다.
- 여러개의 parameter는 &으로 concat한다.

## @Controller

- Traditional MVC Model로 개발할 시 사용되는 Controller annotation이다. 주로 View를 반환할 때 사용되는 Annotation이다.
  ### @ResponseBody
  - 해당 Controller에서 View가 아니라 Data를 반환해야할 시 사용하는 Annotation이다. 해당 Annotation을 통해서, JSON형태로 data를 반환할 수 있다.
  - 예시 코드
  ```kotlin
  @Controller
  @RequiredArgsController //* Lombok; Create final/@NotNull field's constructor
  @RequestMapping(value = "/api")
  class UserController {

  		val udb:UserDatabaseInstance = UserDatabaseInstance() //* User Database, assume it returns User entity.

      @GetMapping("/") //value=는 생략이 가능하다.
  		 fun getView(Model model): String {
  			val hello = "hello!"
  			model.addAttribute("hello", hello);
  			return "/" //*Return View
      }

  		@GetMapping("/getUser")
  		@ResponseBody fun getUser(Model model, @RequestParam("id") String id): ResponseEntity<User>  {
  			return ResponseEntity.ok(udb.fetchUserById(id)) //* Return data by Json
  		}
  }
  ```

## @ResponseController

- Controller + ResponseBody, 즉 data를 response하는 REST API를 개발할 때 사용한다. 반환 객체를 ResponseEntity로 감싸서 return하여 @controller에서의 번거로운 과정이 필요 없다.
- 예시 코드

```kotlin
@RestController
@RequiredArgsController //* Lombok; Create final/@NotNull field's constructor
@RequestMapping(value = "/api")
class UserController {

    val udb:UserDatabaseInstance = UserDatabaseInstance() //* User Database, assume it returns User entity.

		@GetMapping("/getUser")
		fun getUser (Model model, @RequestParam("id") String id): User {
			return udb.fetchUserById(id) //* Return data by Json
		}

		//* Response with Http status setting.
		//*
		@GetMapping("/getUser/withState")
		fun getUserWithResponseEntity (Model model, @RequestParam("id") String id) :ResponseEntity<User> {
			return ResponseEntity.ok(udb.fetchUserById(id), HttpStatus.OK) //* Return data by Json
		}
}
```

## @Value / @PropertySource

- 기존 node.js project에서 사용하던 .env와 같이, 환경 변수나 외부 설정 (properties)들을 불러올 때 사용할 수 있는 annotation이다.
- 먼저 필요한 값들을 properties라는 이름으로 저장한다. (ex: resources/abc.properties)
- abc.properties에 다음과 같은 property가 있다고 가정해보자.
  ```
  abc="abc"
  ```
- 사용할 Bean위에 @PropertySource annotation과 @Value를 활용하여 다음과 같이 참조한다.
  ```kotlin
  @PropertySource("classpath:abc.properties")
  public class TestController{

    @Value("${abc}")
    private val abc: String;
  }
  ```

## @Configuration / @Bean

- Spring에서 수동으로 Bean을 형성할 때, @configuration 클래서 내부에서 @Bean을 사용하게 된다.

## @autowired

- 필요한 의존 객체에 해당하는 Bean을 찾아 Dependency Injection을 해주는 Annotation.

  > Dependnecy Annotation: [Dependency Annotation](<https://github.com/KyumKyum/Learning_Kotlin/blob/main/Basic%20Concepts/DI%20(Dependency%20Injection).md>)

- 다음과 같이 Constructor, Setter Injection 말고도 Field Injection이라는 방법을 추가해준다.

```kotlin
@RestController
@RequestMapping("/")
class ExampleControler(){

  @Autowired
  private lateinit var exampleService: ExampleService
}
```
