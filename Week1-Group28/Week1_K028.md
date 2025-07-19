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
`OPENAI`를 사용한 예제를 보고, 저도 이 기회에 오픈AI를 사용하는 연습을 해야겠다 생각을 했습니다. <br/>
친절하게 예제에 수행에 도움될 내용들을 상세히 적어주셔서 예제를 따라 <br/>
예제 코드를 분석한 뒤 바로 프로젝트에 의존성 주입을 시작하고 구현을 시작했습니다.

<img width="531" height="131" alt="img" src="https://github.com/user-attachments/assets/6c9b314d-c0c0-4fce-8404-cda09f35ea12" />

바로 `OPENAI`의 `apikey`를 발급 받았고, <br/>
`.env`파일을 생성하여 `apikey`를 저장해두었습니다. <br/>
무료로 제공해주는 크레딧이 있어서 문제없이 사용 가능할거라 판단했습니다.

<img width="530" height="97" alt="img_1" src="https://github.com/user-attachments/assets/618d95fb-dba7-4ec9-8256-74cac7fcb41b" />


<img width="263" height="115" alt="img_2" src="https://github.com/user-attachments/assets/16507184-c9ad-4468-a7c9-916771fd783f" />


`dotenv`를 사용하신 것을 보고 코틀린에도 해당 패키지가 있어서 같이 사용해봤습니다. <br/>
올바르게 `apikey`가 불러와지는 것을 확인했습니다.
