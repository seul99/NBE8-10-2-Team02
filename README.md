## 🎮 게임 검색 관리 (Game Search Management)

IGDB API를 기반으로 **게임 이름 + 장르 + 플랫폼 조건을 조합한 검색이 가능한 API**를 구현했습니다.  
외부 API 데이터와 내부 DB 데이터를 분리하여 관리하고,  
QueryDSL을 활용해 **확장 가능한 다중 조건 검색 구조**를 설계했습니다.

---

## 📅 프로젝트 일정
- 기간: 01/15 ~ 01/30  
- 형태: 팀 프로젝트  
- 담당 영역: 게임 검색 도메인 (백엔드)

---

## 🎯 요구 사항 및 배경

### 1. 기술 요구 사항
- IGDB API 연동을 통한 게임 데이터 수집
- 장르 / 플랫폼 기반 다중 조건 검색
- 조건 확장을 고려한 검색 구조
- QueryDSL 기반 동적 쿼리 구현

### 2. 프로젝트 배경 (Background)
게임 검색 서비스 특성상  
- 검색 조건이 계속 늘어날 가능성이 높고  
- 외부 API(IGDB)와 내부 DB 역할을 분리할 필요가 있었습니다.

이에 따라 **검색 조건을 객체로 묶어 관리**하고,  
조건 존재 여부에 따라 쿼리가 유연하게 변경되는 구조를 목표로 설계했습니다.

---

## 👩‍💻 담당 역할
- 게임 검색 조건 설계 (`GameSearchCondition DTO`)
- QueryDSL 기반 조건 검색 로직 구현
- Custom Repository 구조 설계
- IGDB 연동 데이터 기반 장르 / 플랫폼 필터링 구현
- 검색 API 응답 DTO 설계 및 개선

---

## 🧱 설계 및 구현

### 1️⃣ 게임 검색 조건 DTO 분리

#### GameSearchCondition DTO
- 장르 ID / 플랫폼 ID를 List 형태로 관리
- 검색 조건을 하나의 객체로 묶어 전달
- 추후 조건 추가 시 메서드 시그니처 변경 없이 확장 가능

```text
GameSearchCondition
 ├─ List<Long> genreIds
 └─ List<Long> platformIds
```
### 2️⃣ QueryDSL 기반 조건 검색 구현

#### ✔ 설계 포인트
- 단순 파라미터 나열이 아닌  
  **검색 조건 자체를 하나의 도메인 개념으로 관리**
- 조건이 늘어나도 Controller / Service 시그니처 변경 없이 확장 가능

---

#### 🧩 Custom Repository 구조
- `GameQueryRepositoryCustom`
- `GameQueryRepositoryCustomImpl`

```text
Controller
 └─ Service
     └─ GameQueryRepository
         └─ GameQueryRepositoryCustomImpl (QueryDSL)
```
### 🔍 구현 포인트
- `BooleanBuilder` 사용
- 조건이 존재할 경우에만 쿼리 조건 추가
- 장르 / 플랫폼 조건을 **OR가 아닌 AND 조건**으로 처리

👉 조건이 늘어나도  
**쿼리 가독성과 유지보수성을 유지할 수 있도록 설계**

---

## ✨ 주요 API

### 🎯 게임 검색
GET /api/v1/games/search


#### 사용 예시
- 이름만 검색  
  `?query=zelda`

- 이름 + 장르  
  `?query=zelda&genre=12`

- 이름 + 플랫폼  
  `?query=zelda&platform=PC`

- 이름 + 장르 + 플랫폼  
  `?query=zelda&genre=12&genre=5&platform=PS`

#### 반환값
- `id`
- `name`
- `image`
- `firstReleaseDate`
- `genres`
- `(developerName 추후 확장 예정)`

<br>
 
**반환값** <br>
<img width="449" height="143" alt="image" src="https://github.com/user-attachments/assets/124498e9-1feb-4e5b-a0f5-0b05c6666431" />
---

### 🏷️ 장르 목록 조회
GET /api/v1/genres
<br>

**반환값**

<br>

<img width="242" height="284" alt="image" src="https://github.com/user-attachments/assets/12839956-cbdc-46f0-ad60-26b92b8ed637" />

### 🕹️ 플랫폼 목록 조회
GET /api/v1/platforms
<br>

**반환값**
<br>

<img width="199" height="338" alt="image" src="https://github.com/user-attachments/assets/dbf0fd3e-8d78-4aa5-ba94-089e6ddb2a3d" />

---

## ✅ 피드백 반영 사항

### 🏷️ 장르
- 서버 최초 실행 시 **IGDB에서 전체 장르 자동 수집**
- 이후 검색 시 **DB 기준으로 처리**

### 🕹️ 플랫폼
- 버전과 무관하게 검색 가능하도록 **PlatformGroup 개념 도입**
- 드롭다운 선택 시 그룹 단위 필터링
- 플랫폼 그룹은 **추가 / 삭제 가능**

#### 현재 지원 플랫폼
- PS
- Xbox
- Nintendo (Wii 포함)
- PC
- Mobile
- VR

---

## 🔄 개선 및 확장 고려
- 검색 조건 DTO 구조로 인해 조건 확장 용이
- 외부 API 변경이 내부 검색 로직에 미치는 영향 최소화
- 자연어 기반 제목 검색 지원
- 개발사(developer) 조건 추후 추가 예정

---

## 🛠️ 기술 스택
- Java
- Spring Boot
- Spring Data JPA
- QueryDSL
- MySQL
- IGDB API (Twitch OAuth)

---

## 📌 기여 요약
IGDB API 기반 게임 검색 기능을 담당하여  
- QueryDSL을 활용한 **다중 조건 검색**
- **확장 가능한 검색 조건 DTO 구조**를 설계·구현했습니다.

실제 서비스 환경을 고려해  
검색 조건 확장성과 유지보수성을 중심으로 API 구조를 설계했습니다.
