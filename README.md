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

# 🛠 핵심 기술 구현: 
## 외부 API 연동 및 동적 쿼리 엔진 설계
방대한 게임 데이터를 보유한 **IGDB API(Twitch)** 를 효율적으로 호출하고, 사용자 요구사항에 맞춰 데이터를 가공하는 통합 서비스 레이어를 구축했습니다.

#### 1. 외부 API 전용 동적 쿼리 빌더(Query Builder) 구축
IGDB 특유의 쿼리 문법(Apicalypse)에 대응하기 위해, StringBuilder를 활용한 자체 쿼리 빌딩 로직을 구현했습니다.
- 유연한 조건 조합: 이름 검색(~ *query*), 장르(genres = ()), 플랫폼(platforms = ()) 등 다중 조건을 & 연산자로 조합하는 동적 where 절 생성기를 설계했습니다.
- 페이지네이션 및 정렬 자동화: size와 offset을 계산하여 API 단에서 페이지네이션을 처리하고, 최신 출시일(first_release_date desc) 순으로 정렬되도록 최적화했습니다.

#### 2. 안정적인 API 통신 및 보안 관리
- OAuth 2.0 기반 인증 자동화: TwitchTokenService와 연동하여 액세스 토큰을 자동으로 갱신하고, 모든 요청 헤더에 보안 토큰(Bearer Auth)이 주입되도록 설계했습니다.
- 예외 처리 및 로깅: HttpClientErrorException 및 HttpServerErrorException 등 외부 통신 시 발생할 수 있는 에러를 세분화하여 처리하고, 서비스 중단 없이 에러 로그를 남기는 안정적인 구조를 확보했습니다.

#### 3. 데이터 정제 및 매핑 최적화
- 서버 사이드 필터링: API 호출 결과에 대해 Stream API를 활용한 2차 검증(이름 포함 여부 등)을 수행하여 클라이언트에게 가장 정확한 데이터만 전달합니다.
- 효율적인 데이터 변환: Map<Long, String> 구조를 활용하여 플랫폼 ID를 이름으로 빠르게 매핑하는 로직을 통해, 복잡한 연관 데이터 조회 성능을 개선했습니다.

```
/**
 * IGDB 전용 쿼리 문법을 동적으로 생성하여 외부 API 호출 최적화
 */
private String buildIgdbQuery(String query, List<Long> genreIds, List<Long> platformIds, int size, int offset) {
    StringBuilder sb = new StringBuilder();
    sb.append("fields id, name, first_release_date, cover.image_id, genres, platforms;\n");

    List<String> whereConditions = new ArrayList<>();
    if (query != null) whereConditions.add("name ~ *\"" + query + "\"*"); // 와일드카드 검색
    if (genreIds != null) whereConditions.add("genres = (" + join(genreIds) + ")"); // 장르 필터링

    if (!whereConditions.isEmpty()) {
        sb.append("where ").append(String.join(" & ", whereConditions)).append(";\n");
    }
    sb.append("sort first_release_date desc; limit ").append(size).append("; offset ").append(offset).append(";");
    return sb.toString();
}
```
---
## QueryDSL 기반 지능형 동적 검색 엔진
내부 데이터베이스에 저장된 게임 데이터를 효율적으로 검색하기 위해 QueryDSL을 활용한 커스텀 리포지토리를 구현했습니다. 특히, 사용자 편의를 위한 자연어 검색 최적화와 조건별 조인 최적화에 집중했습니다.

#### 1. 사용자 경험을 고려한 '공백 무시' 검색 로직
사용자가 "젤다의전설" 혹은 "젤다의 전설" 중 어떤 방식으로 검색하더라도 동일한 결과를 얻을 수 있도록 자연어 검색 최적화를 진행했습니다.
- 데이터 정규화: Expressions.stringTemplate을 사용하여 DB의 name 컬럼에서 공백을 제거(replace)한 가상 컬럼을 생성했습니다.
- 유연한 매칭: 공백이 제거된 데이터와 사용자의 검색어를 비교하여, 띄어쓰기 여부와 상관없이 정확한 결과를 반환하는 유연한 검색 기능을 구현했습니다.

#### 2. 조건에 따른 동적 조인(Dynamic Join) 최적화
불필요한 데이터 조회를 줄이고 성능을 높이기 위해 검색 조건 유무에 따라 조인 전략을 다르게 가져갔습니다.
- 전략적 조인: 플랫폼 필터가 있는 경우에는 Inner Join을, 없는 경우에는 Left Join을 적용하여 쿼리의 효율성을 높였습니다.
- 데이터 중복 방지: selectDistinct를 사용하여 다대다(N:M) 관계인 장르/플랫폼 조인 시 발생할 수 있는 데이터 중복 문제를 해결했습니다.

```
/**
 * DB 함수 호출 및 동적 조인 전략을 활용한 고도화된 검색 리포지토리
 */
@Override
public List<Game> searchByCondition(GameSearchCondition condition) {
    BooleanBuilder builder = new BooleanBuilder();

    // 1. 공백 무시 자연어 검색 (띄어쓰기 상관없이 검색 가능)
    if (StringUtils.hasText(condition.getQuery())) {
        StringExpression normalizedName = Expressions.stringTemplate("replace({0}, ' ', '')", game.name);
        builder.and(normalizedName.containsIgnoreCase(condition.getQuery().replace(" ", "")));
    }

    // 2. 조건부 조인 전략: 플랫폼 필터 여부에 따라 Inner/Left Join 동적 전환
    var query = queryFactory.selectDistinct(game).from(game).leftJoin(game.gameGenres, gameGenre);
    if (hasPlatformCondition) {
        query.join(game.gamePlatforms, gamePlatform); // 필터 있을 시 Inner Join으로 최적화
    }
    
    return query.where(builder).fetch();
}
```

---
## 하이브리드 데이터 검색 및 매핑 아키텍처
로컬 데이터베이스의 효율성과 외부 API의 방대한 데이터를 동시에 활용하기 위한 Fallback 전략 기반의 검색 서비스를 구축했습니다.

#### 1. 효율적인 하이브리드 검색 전략 (Fallback Mechanism)
사용자 경험 최적화와 서버 리소스 절약을 위해 데이터 조회 우선순위를 설정했습니다.
- 1차 조회 (Local DB): QueryDSL을 활용해 내부 DB에 저장된 고품질 데이터를 우선적으로 검색합니다.
- 2차 조회 (External API): 로컬 DB에 결과가 없을 경우, IgdbService를 통해 외부 API에서 실시간으로 데이터를 페칭하여 검색 결과의 누락을 방지했습니다.

#### 2. 플랫폼 그룹화 및 데이터 노멀라이제이션 (Normalization)
서로 다른 데이터 형식(사용자 입력 vs API 데이터)을 일관성 있게 통합했습니다.
- 플랫폼 코드 매핑: 사용자가 입력한 단순 플랫폼 코드(예: "PS", "PC")를 PLATFORM_MAP을 거쳐 IGDB의 세부 ID 목록으로 변환하는 전처리 로직을 구현했습니다.
- 데이터 정규화: 검색어의 공백이나 대소문자 차이로 인해 결과가 달라지지 않도록 SearchNormalizer를 도입하여 검색 정확도를 높였습니다.

#### 3. 스트림 기반 다중 도메인 데이터 매핑
외부 API에서 가져온 원본 데이터를 사용자에게 보여주기 적합한 형태(Response DTO)로 변환하는 과정을 최적화했습니다.
- 장르 및 플랫폼 통합: 외부 API의 ID 값들을 추출(flatMap)한 뒤, 내부 장르 리포지토리 및 IGDB 플랫폼 맵과 교차 참조하여 이름 기반의 직관적인 데이터로 변환했습니다.
- 효율적인 메모리 매핑: 호출마다 반복되는 조회를 방지하기 위해 Map<Long, String> 구조를 활용하여 변환 성능을 O(1)로 유지했습니다.

```
/**
 * 로컬 DB 검색 -> 실패 시 외부 API 검색으로 이어지는 
 * 하이브리드 검색 로직 및 데이터 매핑 전용 서비스
 */
public List<GameSearchResponse> search(GameSearchCondition condition) {
    // 1. 검색어 정규화 및 플랫폼 코드 매핑 (전처리)
    condition.setQuery(normalize(condition.getQuery()));
    mapPlatformCodesToIds(condition);

    // 2. Local DB(QueryDSL) 우선 검색
    List<Game> localGames = gameSearchRepository.searchByCondition(condition);
    if (!localGames.isEmpty()) return localGames.stream().map(GameSearchResponse::from).toList();

    // 3. Fallback: 외부 API(IGDB) 검색 및 연관 데이터(장르, 플랫폼) 매핑
    List<IgdbGameSummaryDto> igdbGames = igdbService.search(condition);
    return mapToResponse(igdbGames);
}
```

---

## 📌 기여
IGDB API 기반 게임 검색 기능을 담당하여  
- QueryDSL을 활용한 **다중 조건 검색**
- **확장 가능한 검색 조건 DTO 구조**를 설계·구현했습니다.

실제 서비스 환경을 고려해  
검색 조건 확장성과 유지보수성을 중심으로 API 구조를 설계했습니다.

---

## 💻 발표 자료
![1](https://github.com/user-attachments/assets/3000d214-64c3-4bec-9b7b-964e167437a6)
![2](https://github.com/user-attachments/assets/a725d545-4bc4-439f-9bba-290460441820)
![3](https://github.com/user-attachments/assets/f20b9b50-7baa-4103-b7a0-ef0f7a4688eb)
![4](https://github.com/user-attachments/assets/552ee796-cb01-4bee-bae5-798203a09513)
![5](https://github.com/user-attachments/assets/de936457-4f29-440c-b822-606d3b0ecf67)
![6](https://github.com/user-attachments/assets/e45098fc-5cee-4452-9714-fabecb947800)
![7](https://github.com/user-attachments/assets/e6746885-e42b-468f-8164-675a0bbbeb9f)
![8](https://github.com/user-attachments/assets/cf4e14db-0e1d-41d4-a0d3-1080fc82eaed)
![9](https://github.com/user-attachments/assets/20aee9c3-d57f-4449-b919-d66ed593fdb8)
![10](https://github.com/user-attachments/assets/8e5831d1-9dd7-4800-bf3b-4d5119607bae)
![11](https://github.com/user-attachments/assets/aceab412-d5fd-4376-8a83-5dfe8021ea72)
![12](https://github.com/user-attachments/assets/0ac79558-c09e-45d5-a070-d1c1abd3e0a2)
![13](https://github.com/user-attachments/assets/df4d87a1-97d7-4d03-9804-b3d84110c865)
![14](https://github.com/user-attachments/assets/72897ae4-7247-487e-8ef3-a6847a6dcc87)
![15](https://github.com/user-attachments/assets/b1aea4d6-97a5-40c1-843b-e05286d56bc6)
![16](https://github.com/user-attachments/assets/974940ce-cbb3-4eb2-afed-eb074625536e)
![17](https://github.com/user-attachments/assets/4ec4e645-e263-456c-b4d9-d4bf6a726085)
![18](https://github.com/user-attachments/assets/0ae4610c-1cb3-4874-a3ed-1afc6efc99ba)
![19](https://github.com/user-attachments/assets/26f79400-c645-4622-b19b-b928b40e0b6f)
![20](https://github.com/user-attachments/assets/a80c2b11-aaba-43e9-a405-bad463fb9e9e)
![21](https://github.com/user-attachments/assets/f3c010fa-dbdc-4395-9b61-03d163e6093b)
![22](https://github.com/user-attachments/assets/58085fcf-ea72-40d9-b39d-20a69a7bbfae)
![23](https://github.com/user-attachments/assets/3bee5a23-4ed1-4d43-9a4a-451dfdbabf01)
![24](https://github.com/user-attachments/assets/33003d55-0286-4d7c-bdcf-1fbe8f007a54)
![25](https://github.com/user-attachments/assets/4e7f6c2c-586a-492a-8696-8476e4fe38a7)
![26](https://github.com/user-attachments/assets/26a92124-01ad-4a62-8e2c-465e8cc93bc1)
![27](https://github.com/user-attachments/assets/0e5e7054-e693-42e0-a7a7-5a3f19aee064)
![28](https://github.com/user-attachments/assets/2a77e8b6-6905-4dbc-a2a0-c0800564d3f3)



















