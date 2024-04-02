---
title: 'about'
date: 2021-1-3 16:21:13
lang: 'ko'
---

# 임지영

## Projects

### VIBE 사용자 이벤트 수집기

뮤직 웹앱 Naver VIBE를 클론하고, 프로덕트의 사용성 향상, 이용자 행동 패턴 분석을 위한 사용자 이벤트를 수집하는 프로젝트
[소스 코드](https://class101.net/ko/shortforms/65a6637dad9e233e3fc59209)

![mini vibe screenshot](https://imgur.com/VJiOi6W.png)

기간: 2020.11 - 2020.12
- 아토믹 디자인 패턴을 도입하여 재사용 가능한 컴포넌트 개발 및 스토리북 도입
- context API를 활용하여 각 컴포넌트의 식별자 정보를 전달하는 Wrapper 컴포넌트 구현
- 이벤트 수집을 위한 커스텀 훅 구현
- Mongoose discriminator/schema를 이용하여 요청된 로그 데이터에 대해 이벤트 구조를 검증하는 모델 정의, 이벤트 로그 API 구현
- typeORM를 통해 엔티티 정의 및 API 구현

#React.js #express.js #Next.js #TypeScript #MongoDB #Storybook


### [GitHub의 Issue 클론 프로젝트](https://github.com/YimJiYoung/IssueTracker-22)

![issue tracker screenshot](https://i.imgur.com/C8b5Db4.png)

기간: 2020.10 - 2020.11
- Container-Presenter 패턴을 도입하여 로그인, Label 목록 페이지의 컴포넌트 구현
- 인증을 위한 커스텀 훅 구현, 로그인된 유저 정보 context API로 관리
- sequelize를 통해 이슈 생성, 마일스톤, 레이블 관련 REST API 구현
- API 테스트 코드 작성
- 데일리 스크럼, 주별 스프린트 계획, 회고 등으로 애자일 개발 프로세스 경험
- 코드 리뷰를 통해 피드백을 주고 받아 팀원들이 함께 성장할 수 있도록 도모

#React.js #express.js #sequelize #React-router 