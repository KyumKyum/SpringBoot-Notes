# Cores

Spring Boot의 Cores에 대해 정리한 내용입니다. 기본으로 알고 있으면 도움이 될 만한 지식들입니다. Spring Boot이 어떻게 돌아가는지에 대해서 공부한다고 생각하시면 됩니당.

# Lazy Initialization

- Lazy Initialization에서의 Lazy는 Operating System과 같은 CS분야에서 이야기하는 Lazy의 의미와 같다고 생각하면 된다. Build 시, Spring Bean을 비롯한 모든 요소가 생성 후 실행 되는 것이 아니라, ‘필요할 때에’ 생성이 되는 매커니즘이다. 요청이 있을 때에만 해당 요청에 해당하는 객체가 형성되고 실행된다.
    - Bean: new A()와 같이 객체의 형성/실행의 주체가 우리가 아닌 Spring Boot인 객체. Spring Boot가 관리하는 객체. (IoC: Inversion of Control)
- 해당 Lazy Initialization은 기존 Lazy Approach와 같은 장,단점을 보유하고 있다. 빠른 Build와 효율적인 Memory Mgmt가 가능하다는 장점이 있지만, 오류와 problem이 있는 코드의 경우, start up 때에  (즉 build 시) 발견이 되지 않고, 해당 요청이 있어 init이 될 때에 발견된다는 단점이 있다. (오류가 있을 경우, 발견이 늦게 된다.)
- 기본적으로 disabled이며, 활성화를 하기 위해서는 properties에 추가하면 된다.

```
spring.main.lazy-initialization=true
```

- 위와 같이 enable된 경우, 이후 Bean에 @Lazy(false)를 통해 부분적 disable도 가능하다.

# State

- Spring Application은 availability에 대한 state를 표시할 수 있다.
    - Liveness: Spring Application이 잘 작동 하거나, 오류가 났을 경우 스스로 recover가 가능한지.
    - Readiness: Spring Application이 traffic을 handle할 수 있는 상태인지.
- 이러한 state들은 ApplicationAvailibility를 injecting함으로서 해당 state에 대해 listen & update를 할 수 있다.

# Event & Start up

- Spring Application은 다음과 같은 Start up routine을 가지고 있으며, 해당 routine에서 일어나는 event는 다음과 같다.
    1. `ApplicationStartingEvent`: Application이 시작될 때 sent되는 event이며,, 모든 listener와 initializer들이 실행된다.
    2. `ApplicationEnvironmentPreparedEvent`: Application의 Context가 실행 되기 전, 해당 context가 사용할 enviroment를 준비할 때 sent되는 event이다.
    3. `ApplicationContextInitializedEvent`: ApplicationContext가 준비되었을 때 sent되는 Event이며, 각 context의 initalizer (ApplicationContextInitializers)들은 준비되었지만 Bean definition이 load되기 전.
    4. `ApplicationPreparedEvent`: Bean Definition이 load 되었을 때 sent되는 event이다. 이 event가 호출 된 후에 Context가 Refresh된다.
    5. `ApplicationStartedEvent`: Context Refresh 후 sent되는 event이며, 아직 applicatio이나 command-line runner가 호출되지 않은 상태이다. 
    - Application Runner / Command-Line Runner
        - Spring Application이 실행되긴 했지만, Traffic을 Accept하기 전 (Ready가 되기 전)에 실행되어야 하는 Task들을 여기서 지정해 실행할 수 있다. (Run after SpringApplication has started, but just before completing SpringApplication.run(…))
        - Application Runner는 ApplicationArgument를 arugment로, Command-Line Runner는 String을 argument로 필요할 때 사용된다.
        - 이러한 여러개의 Runner들은 Annotation @Order로 sequence를 지정할 수 있다.
        
    1. `AvailabilityChangeEvent`: 모든 실행이 성공적으로 완료가 되었고, Application이 정상 작동 (live state: `LivenessState.CORRECT`) 되었을 때 sent되는 event이다.
    2. `ApplicationReadyEvent`: 아무런 Application/Command-Line Runner가 호출된 후 sent되는 event이다.
    3. `AvailabilityChangeEvent`: Readiness State가 정상일 때 (`ReadinessState.ACCEPTING_TRAFFIC`), 즉 정상적으로 traffic을 handle할 수 있는 상태가 된 직후에 sent되는 event이다.

---

- `ApplicationFailedEvent`: 위 과정 중 exception이 발생했을 경우 sent되는 event이다.

- 이러한 event는 child context에서 발생을 하여 publish되어도 parent (ancestor) context에서도 publish가 된다. 이와 같은 이유로 같은 종류의 event를 여러번 받을 수 있다.
    - Listener가 이러한 event를 구분하기 위해서는 (본인에게서 발생한건지, 아니면 child에서 발생한건지), ApplicationContextAware를 implement하거나 annotation @Autowired를 통해 발생한 context를 비교하면 된다.

# Debuging

```kotlin
import org.springframework.boot.ApplicationArguments
import org.springframework.stereotype.Component

@Component
class MyBean(args: ApplicationArguments) {

    init {
        val debug = args.containsOption("debug")
        val files = args.nonOptionArgs
        if (debug) {
            println(files)
        }
        // if run with "--debug logfile.txt" prints ["logfile.txt"]
    }

}
```

# Exit

```kotlin
import org.springframework.boot.ExitCodeGenerator
import org.springframework.boot.SpringApplication
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication
import org.springframework.context.annotation.Bean

import kotlin.system.exitProcess

@SpringBootApplication
class MyApplication {

    @Bean
    fun exitCodeGenerator() = ExitCodeGenerator { 42 }

}

fun main(args: Array<String>) {
    exitProcess(SpringApplication.exit(
        runApplication<MyApplication>(*args)))
}
```

- SpringApplication.exit()이 호출되면 return할 exit code를 위와 같이 지정할 수 있으며, 해당 code는 이후에 System.exit()에게 전달되어 status code와 같이 사용될 수 있다.