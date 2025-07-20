# K028 Week1 수행 내용

| 이름             | K028 최동현                                             |
|----------------|----------|
| 수행한 퀘스트 (수행 중) | 1  |

## 미션1. AI로 챌린지 생활 시간표 만들기 📅

### 요구사항
- [ ] 목표를 입력하면 시간표를 짜주는 AI 프롬프트를 작성해보고, 결과를 Github 레포지토리에 공유할 것
- 시간표를 적절한 양식으로 출력해야 한다
  - [ ] 이미지, 텍스트, 표 등 알아볼 수 있는 자유로운 형식으로 제출
- [ ] 하루 또는 일주일동안 이 시간표로 미션에 임해볼 것

### 수행내역
AI에게 프롬프트를 만들어서 시간표를 받을 때, <br/>
간단하게 `Chat-GPT` 같은 사이트에 직접 접속해서 간단하게 수행해도 되는 미션이지만 <br/>
`OPENAI`를 사용한 예제를 보고 리서치해 본 결과 Kotlin에도 `OPENAI` 관련 라이브러리가 있어서 <br/>
저도 이 기회에 오픈AI를 사용하는 연습을 해야겠다 생각을 했습니다. <br/>
친절하게 예제에 수행에 도움될 내용들을 상세히 적어주셔서 예제를 따라 <br/>
예제 코드를 분석한 뒤 바로 프로젝트에 의존성 주입을 시작하고 구현을 시작했습니다.

<img width="552" height="245" alt="img_4" src="https://github.com/user-attachments/assets/2c15042b-0cf3-41b0-b089-81284b66497d" />


- `dotenv` : `.env` 파일에서 환경변수를 쉽게 불러오는 라이브러리
- `Ktor Client` : Kotlin의 공식 비동기 HTTP 클라이언트
- `kotlinx-json` :  Kotlinx JSON 직렬화 라이브러리
- `openai` : Kotlin에서 OpenAI API 접근할 수 있도록 도와주는 클라이언트 라이브러리 (공식X)
- `SLF4J` : 로그 관련 라이브러리

바로 `OPENAI`의 `apikey`를 발급 받았고, <br/>
`.env`파일을 생성하여 `apikey`를 저장해두었습니다. <br/>
무료로 제공해주는 크레딧이 있어서 문제없이 사용 가능할거라 판단했습니다.


<img width="530" height="97" alt="img_1" src="https://github.com/user-attachments/assets/80cf4527-b0bd-496d-8183-a02f8a31b964" />

<br/>

<img width="263" height="115" alt="img_2" src="https://github.com/user-attachments/assets/d65cac55-08d9-4002-8789-e9e74972f98c" />


`dotenv`를 사용하신 것을 보고 코틀린에도 해당 패키지가 있어서 같이 사용해봤습니다. <br/>
올바르게 `apikey`가 불러와지는 것을 확인했습니다.

### `OpenAI` 연동 테스트
```kotlin
fun main() = runBlocking {
    val openAI = OpenAI(
        token = api_key,
    )

    val modelId = ModelId("gpt-4o")

    val completion = openAI.chatCompletion(
        ChatCompletionRequest(
            model = modelId,
            messages = listOf(
                ChatMessage(role = ChatRole.User, content = "안녕, 내 요청이 정상적으로 들어간다면 네이버부스트캠프! 라고 해줘")
            )
        )
    )
    println(completion.choices.firstOrNull()?.message?.content)
}
```
- 요구사항에서 `gpt-4o` 모델을 사용해달라고 하여, 해당 모델로 테스트를 진행 했습니다.

#### 테스트 결과
<img width="450" height="262" alt="img_5" src="https://github.com/user-attachments/assets/7bf13b12-544b-4a28-a9dc-52a10d0ba8be" />


정상적으로 AI 모델이 답을 반환하는 것을 확인했습니다.

### `Prompt` 작성하는 클래스 작성
```kotlin
import java.time.LocalDate
import java.time.LocalTime
import java.time.format.DateTimeFormatter

object Schedule {

    data class DailySchedule(
        val userName: String,
        val learningObjectives: List<String>,
        val wakeTime: LocalTime,
        val sleepTime: LocalTime,
        val lunchTime: LocalTime,
        val memo: String,
    ) {
        companion object {
            val dailyPlan = DailySchedule(
                userName = "K028 최동현",
                learningObjectives = listOf("프로세스 메모리 구조", "운영체제별 메모리 관리 방식"),
                wakeTime = LocalTime.parse("07:00"),
                sleepTime = LocalTime.parse("03:00"),
                lunchTime = LocalTime.parse("12:00"),
                memo = "비전공자라 추천 학습 순서가 있으면 좋겠어."
            )
        }
    }

    object Prompt {
        fun writePrompt(schedule: DailySchedule): String {
            val today = LocalDate.now().format(DateTimeFormatter.ofPattern("yyyy년 M월 d일 (E)", java.util.Locale.KOREAN))
            return """
                날짜: $today
                너는 사용자의 학습 계획을 짜주는 AI 도우미야.

                사용자 정보:
                - 이름: ${schedule.userName}
                - 학습 목표: ${schedule.learningObjectives.joinToString(", ")}
                - 기상 시간: ${schedule.wakeTime}
                - 취침 시간: ${schedule.sleepTime}
                - 점심 시간: ${schedule.lunchTime}
                - 참고 사항: ${if (schedule.memo.isBlank()) "없음" else schedule.memo}

                요청 사항:
                - 학습 목표를 달성하기 위한 하루 일과 시간표를 촘촘하게 짜줘.
                - 점심 시간은 1시간을 고정적으로 배치해줘.
                - 학습목표를 어떤 식으로 학습하면 좋을지 각 시간 블록마다 무엇을 하면 좋을지 설명을 해줘.
                - 쉬는 시간은 적절히 포함하되, 학습 집중도를 최대로 높일 수 있게 구성해줘.
                - 말투는 친절하지만 지나치게 부드럽지 않게. 똑 부러지게 알려줘.

                결과:
                - 시간대별 할 일을 나열해줘. 보기 좋게 표처럼 정리해도 좋아.
                - 마지막에 전체 계획을 요약하는 코멘트를 추가해줘.
            """.trimIndent()
        }
    }
}
```

### 출력 결과
```kotlin

```
=======
<img width="530" height="97" alt="img_1" src="https://github.com/user-attachments/assets/618d95fb-dba7-4ec9-8256-74cac7fcb41b" />


<img width="263" height="115" alt="img_2" src="https://github.com/user-attachments/assets/16507184-c9ad-4468-a7c9-916771fd783f" />


`dotenv`를 사용하신 것을 보고 코틀린에도 해당 패키지가 있어서 같이 사용해봤습니다. <br/>
올바르게 `apikey`가 불러와지는 것을 확인했습니다.
