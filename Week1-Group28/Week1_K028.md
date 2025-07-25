# K028 Week1 수행 내용

| 이름             | K028 최동현    |
|----------------|-------------|
| 수행한 퀘스트 (수행 중) | 1 →  3  → 4 |

각 미션이 서로 연계하며 이어지는 형식을 가지고 있지만, <br/>
미션 진행 순서는 흐름에 맞춰서 임의로 수정했습니다.

## 미션1. AI로 챌린지 생활 시간표 만들기 📅

### 요구사항
- [x] 목표를 입력하면 시간표를 짜주는 AI 프롬프트를 작성해보고, 결과를 Github 레포지토리에 공유할 것
- 시간표를 적절한 양식으로 출력해야 한다
  - [x] 이미지, 텍스트, 표 등 알아볼 수 있는 자유로운 형식으로 제출
- [x] 하루 또는 일주일동안 이 시간표로 미션에 임해볼 것

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

### `Prompt` 작성하는 클래스
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

<img width="530" height="97" alt="img_1" src="https://github.com/user-attachments/assets/618d95fb-dba7-4ec9-8256-74cac7fcb41b" />

<br/>

<img width="263" height="115" alt="img_2" src="https://github.com/user-attachments/assets/16507184-c9ad-4468-a7c9-916771fd783f" />


`dotenv`를 사용하신 것을 보고 코틀린에도 해당 패키지가 있어서 같이 사용해봤습니다. <br/>
올바르게 `apikey`가 불러와지는 것을 확인했습니다.

### AI에게 프롬포트 요청
```kotlin
fun main() = runBlocking {
    val openAI = OpenAI(
        token = api_key,
    )
    val modelId = ModelId("gpt-4o")

    val schedule = Schedule.DailySchedule.dailyPlan
    val prompt = Schedule.Prompt.writePrompt(schedule)

    val completion = openAI.chatCompletion(
        ChatCompletionRequest(
            model = modelId,
            messages = listOf(
                ChatMessage(role = ChatRole.User, content = prompt)
            )
        )
    )
    println(completion.choices.firstOrNull()?.message?.content)
}
```
- 작성한 일정과 프롬프트를 불러와 `OpenAI` 모델에게 요청을 보내도록 구현하고 시간표를 받아 출력했습니다.

### AI에게 받은 시간표 출력
<img width="3162" height="841" alt="img_6" src="https://github.com/user-attachments/assets/92ce4091-c2c0-4126-9615-c0e928ed8f65" />


| 시간        | 활동                                        | 내용 설명                                                                                       |
|-------------|---------------------------------------------|------------------------------------------------------------------------------------------------|
| 07:00 - 08:00 | 모닝 리추얼                                | 일어나서 간단한 스트레칭과 명상 등으로 정신을 맑게 하세요. 하루 준비 운동과 계획 점검을 합니다.                     |
| 08:00 - 09:30 | 기초 개념 학습: 메모리의 기본 구조 이해       | 메모리가 어떻게 구성되고 동작하는지 기본 개념 학습을 시작합니다. 유튜브 강의나 온라인 튜토리얼을 활용하세요.         |
| 09:30 - 09:45 | 쉬는 시간                                   | 짧은 산책이나 커피 타임으로 머리를 식히세요.                                                   |
| 09:45 - 11:15 | 심화 학습: 프로세스 및 메모리 구조          | 프로세스와 메모리의 상호작용을 이해하기 위해 프로세스 주소 공간 등의 개념을 학습합니다. 슬라이드나 학습 자료 활용.   |
| 11:15 - 11:30 | 쉬는 시간                                   | 가볍게 스트레칭하면서 휴식을 취하세요.                                                         |
| 11:30 - 12:00 | 퀴즈 및 복습                                | 앞서 배운 내용을 간단한 퀴즈로 복습하며 이해도를 점검하세요. 온라인 퀴즈 툴을 사용해도 좋습니다.                        |
| 12:00 - 13:00 | 점심 시간                                   | 점심 식사는 충분한 영양을 섭취하는 것이 중요합니다. 휴식을 취하며 이후 학습을 위한 컨디션을 준비하세요.                 |
| 13:00 - 14:30 | 운영체제별 메모리 관리 연구(1)               | 각 운영체제에서의 메모리 관리 방식을 조사하고 비교하세요. 문서를 참조하며 차이점을 서면으로 정리해 보세요.            |
| 14:30 - 14:45 | 쉬는 시간                                   | 상쾌한 공기를 마시며 짧은 산책으로 체력을 회복하세요.                                           |
| 14:45 - 16:15 | 운영체제별 메모리 관리 연구(2)               | 다양한 OS에서의 메모리 관리 전략에 대한 토픽을 심화 학습합니다. 관련 포럼이나 Q&A 사이트를 활용하여 질문하세요.        |
| 16:15 - 16:30 | 쉬는 시간                                   | 간단한 간식을 먹으며 에너지를 보충하세요.                                                       |
| 16:30 - 18:00 | 실습: 가상 머신을 통한 실습                  | VirtualBox나 VM Ware를 이용해 여러 운영체제를 체험하며 메모리 관리 기능을 실습합니다.            |
| 18:00 - 19:00 | 저녁 식사 및 휴식                             | 충분히 휴식을 취하며 몸과 마음을 재충전하세요.                                                   |
| 19:00 - 21:00 | 문제 해결 및 연습                           | 앞서 배운 내용을 바탕으로 문제를 해결해보고, Mock Test를 진행하세요. 토론이 가능하다면 스터디 그룹을 형성하세요.        |
| 21:00 - 21:15 | 쉬는 시간                                   | 휴식시간을 갖고, 맑은 정신으로 다시 시작하세요.                                                  |
| 21:15 - 23:00 | 추가학습/복습                                | 하루 동안의 학습 내용을 복습하고, 이해가 안 된 부분을 다시 한번 학습합니다. 온라인 자료나 도서를 참고하세요.          |
| 23:00 - 23:30 | 요약 정리 및 자기 평가                        | 오늘 학습한 내용을 요약하고, 이해도와 성과를 스스로 평가하세요. 내일 학습 계획도 미리 세워보세요.                      |
| 23:30 - 00:00 | 자유 시간                                   | 남은 시간은 자신의 취미 생활이나 휴식을 하며 보내세요.                                          |
| 00:00 - 03:00 | 심야 학습(선택 사항)                         | 여유가 있으면 오늘의 학습 내용을 확장하거나 더 많은 학습을 시도해보세요. 충분한 수면은 잊지 마세요.                   |

### AI에게 요청한 전체 계획을 요약한 코멘트
최동현님, 순차적으로 기초 개념에서부터 심화되는 학습을 진행하며 <br/>
프로세스 메모리 구조와 운영체제별 메모리 관리에 대해 이해할 수 있도록 계획했습니다. <br/>
휴식 시간을 적절히 배치하여 학습 집중도를 높였으며, 실습을 통해 배운 내용을 적용해 보도록 했습니다. <br/>
이후 목표 달성을 위해 이러한 학습 패턴을 꾸준히 유지하시면 좋겠습니다. <br/>
학습 중 혼동되는 부분이 있을 경우, 추가 자료를 찾아보거나 전문가에게 질문하는 것도 좋은 방법이니 참고하세요. 화이팅입니다!


## 미션 3. 잔소리 봇에 컨셉 부여하기
잔소리를 해주는 프롬프트를 만들 때 컨셉을 추가해달라는 미션도 있었습니다.

### 요구사항
- [x] 잔소리 봇의 말투 옵션을 선택할 수 있어야 해요.
- [x] 어떤 말투 옵션이 좋을지도 생각해보아요.
  - (예 : 엄마, 남자친구)
  - 남자친구/여자친구) 자기야. 아직도 학습 중이야? 오늘은 나랑 언제 전화할 수 있어 ? 😞
- [x] AI 응답이 선택한 말투를 잘 사용하는지 테스트 후 결과를 캡처해요.
- [ ] 조사 후 웹훅으로 슬랙 연동해보아요.(선택)

```kotlin
// Prompt 관련 함수
val concept = "감정보다는 이성과 루틴 강조하는 다정하지만 현실적인 여자친구"
  ...
너의 컨셉은 ${concept}
컨셉에 몰입해서 작성을 해줘.
```
위 요청을 `Prompt`에 추가해서 컨셉을 부여했습니다.

### 출력 결과
### 하루 일과 시간표

| 시간          | 내용                                                         | 설명                                                                                     |
|---------------|------------------------------------------------------------|----------------------------------------------------------------------------------------|
| 07:00 - 08:00 | 기상 및 아침 정리                                              | 상쾌하게 하루를 시작하며 정신을 깨우자. 가벼운 스트레칭이나 명상도 좋아.                     |
| 08:00 - 10:00 | 기본 개념 이해: 메모리 구조 및 역할                             | 컴퓨터 메모리의 기본 구조와 역할에 대해 공부해. 유튜브 강의나 초보자용 서적으로 시작하면 좋을 것 같아. |
| 10:00 - 10:15 | 짧은 휴식                                                     | 머리를 잠시 식혀 줘. 잠깐 산책해도 좋고, 눈을 감고 쉬어도 좋아.                                    |
| 10:15 - 12:00 | 심화 학습: 프로세스 메모리 및 메모리 주소 공간                   | 프로세스가 메모리를 관리하는 방식에 대해 좀 더 깊게 알아보자. 자료구조와 주소 공간에 집중해봐.      |
| 12:00 - 13:00 | 점심 식사                                                     | 운이 네게 밥을 줄 때까지 기다리지 말고, 스스로 좋은 식사를 챙기자.                                |
| 13:00 - 15:00 | 휴식 및 개인 시간                                              | 아무 걱정 없이 좋아하는 활동을 즐기며 다시 에너지를 얻는 데 집중해보자.                               |
| 15:00 - 17:00 | 운영체제별 메모리 관리 방식 I: Windows                          | 윈도우 운영체제에서 메모리가 어떻게 관리되는지 학습해봐. 공식 문서나 강좌를 활용하면 좋아.                 |
| 17:00 - 17:15 | 짧은 휴식                                                     | 간단한 간식이나 음료와 함께 잠시 쉬면서 리프레시해.                                                     |
| 17:15 - 19:00 | 운영체제별 메모리 관리 방식 II: Linux                           | 리눅스에서의 메모리 관리 기법을 배워보자. 다른 OS와 어떻게 다른지 비교하는 것도 잊지 말고.             |
| 19:00 - 20:00 | 저녁 식사                                                     | 저녁을 잘 먹고 내일을 위한 에너지를 쌓아 두자. 특별한 보상이 있는 식사로 보답해도 좋아.                      |
| 20:00 - 22:00 | 복습 및 요약                                                   | 오늘 학습한 내용을 정리하고 주요 개념을 복습하자. 컨셉을 이해하지 못한 부분은 다시 확인하자.               |
| 22:00 - 23:00 | 심화 탐색: 고급 메모리 관리 테마                               | 메모리 교착 상태나 페이지 교환 같은 좀 더 복잡한 개념을 탐구해봐. 추가 자료를 찾아보는 것도 추천해.          |
| 23:00 - 01:00 | 개인 취미 및 휴식                                              | 크게 무리하지 않는 선에서, 너가 좋아하는 활동을 하며 지친 머리를 풀어줘.                                |
| 01:00 - 03:00 | 정리 및 취침 준비                                              | 오늘의 학습을 천천히 정리하고 내일 준비를 하며 마무리하자. 리프레시한 마음으로 하루를 끝내자.              |


### 화요일 미션 시간표 요청
<img width="2451" height="554" alt="img_7" src="https://github.com/user-attachments/assets/67abb091-cee6-44c8-b049-fb2ab43cf891" />

### 목요일 미션 시간표 요청

<img width="2060" height="723" alt="img_8" src="https://github.com/user-attachments/assets/f73d8c4c-8b37-423a-a1e0-49f919efd7bd" />


### 수행 후기
아무래도 미션의 내용에 대해서 전 날에 알 수가 없다 보니 <br/>
시간표를 미리 출력 받아서 그 시간표 대로 수행하기는 어려웠습니다. <br/>
프롬프트를 주말에 복습하기 위한 용도로 수정하여 이용하면 나쁘지 않을 것 같다는 생각이 들었습니다. <br/>
오픈 AI에 `API Key`를 사용하여 연동도 해보고, <br/>
직접 연동한 LLM 모델에 프롬프트를 작성하여 시간표도 작성 받아 보니 <br/>
상당히 의미도 있고 재미있는 미션이었습니다!
