# The Cat API 사용법 정리

출처: https://docs.thecatapi.com/ (공식 문서), OpenAPI 스펙 `https://api.thecatapi.com/openapi-json` (v1.7.1)

## 1. 기본 정보

| 항목 | 내용 |
|---|---|
| Base URL | `https://api.thecatapi.com/v1` |
| 인증 | 모든 요청에 `x-api-key: YOUR-API-KEY` 헤더 |
| API 키 발급 | https://thecatapi.com/signup (이메일로 발급) |
| 계정 포털 | https://account.thecatapi.com |
| 업로드 전용 호스트 | `https://upload.thecatapi.com/v1/images/upload-sync` |
| OpenAPI 스펙 | https://api.thecatapi.com/openapi-json |
| API Reference | https://docs.thecatapi.com/reference |

> 참고: OpenAPI 스펙 안의 서버 주소는 `beta-api.thecatapi.com`으로 표기되어 있으나, 공식 문서의 예제는 모두 `api.thecatapi.com`을 사용합니다. 이 문서는 공식 문서 기준입니다.

### 인증 예외 (스펙 기준, 키 없이 호출 가능)
`GET /votes`, `GET /votes/{vote_id}`, `GET /images/search`, `GET /images/{id}`
(단, 키 없이 호출하면 `limit` 등에 제한이 있을 수 있음. 문서상 `limit` 1~100은 API 키가 있을 때 기준)

### 요금제
- **Free**: 학습/취미 프로젝트용. 핵심 필드(이름, 성격, 원산지, 수명, 설명, 이미지)만 제공
- **Startup**: 상업 라이선스, 더 높은 요청 한도, 전체 품종 데이터(특성 점수 1~5, 사실/facts 등), `lang` 다국어
- **Enterprise**: 사진 분석, 건강 기록, 계산기, 건강 팁 등
- 내 키로 쓸 수 있는 기능 확인: `GET /accounts/features`
- 상태 코드/요청 한도/재시도 정책은 문서의 "Errors and limits" 페이지 참고 (구체 수치는 확인하지 못함)

## 2. 빠른 시작

```bash
curl -H "x-api-key: YOUR-API-KEY" https://api.thecatapi.com/v1/breeds/abys
```

### JavaScript (fetch) 예제

```js
const API_KEY = 'YOUR-API-KEY';
const BASE = 'https://api.thecatapi.com/v1';

async function getCats(limit = 5) {
  const res = await fetch(`${BASE}/images/search?limit=${limit}`, {
    headers: { 'x-api-key': API_KEY },
  });
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json(); // [{ id, url, width, height, breeds }]
}
```

## 3. 이미지 (Images)

### 이미지 검색 `GET /images/search`
```bash
curl -H "x-api-key: YOUR-API-KEY" "https://api.thecatapi.com/v1/images/search?limit=2"
```
응답 (실제 호출 결과 기준, 2026-11-09 테스트):
```json
[
  {
    "id": "amc",
    "url": "https://cdn2.thecatapi.com/images/amc.jpg",
    "width": 640,
    "height": 480,
    "created_at": "2026-04-12T08:52:59.979Z",
    "breeds": [],
    "categories": [],
    "colours": [],
    "tags": []
  }
]
```
- 공식 문서 예시에는 `id`, `url`, `width`, `height`, `breeds`만 나오지만, 실제로는 `created_at`, `categories`, `colours`, `tags`도 함께 옵니다.
- 이미지 확장자는 jpg 외에 gif도 섞여 나옵니다 (예: `c2o.gif`). `mime_types=jpg,png`로 제한할 수 있습니다.

| 파라미터 | 설명 |
|---|---|
| `limit` | 개수 (1~100, 기본 1) |
| `page` | 페이지 (0부터 시작) |
| `order` | `ASC` / `DESC` / `RAND` |
| `has_breeds` | 품종 정보가 있는 이미지만 |
| `breed_id`, `breed_ids` | 품종 ID 필터 (`breed_ids`는 쉼표 구분) |
| `include_breeds` | 응답에 품종 상세 포함 |
| `include_categories` | 응답에 카테고리 포함 |
| `colour_id(s)`, `colour_code`, `has_colours`, `include_colours` | 털 색상 관련 |
| `age_months_min`, `age_months_max` | 나이(개월) 범위 |
| `mime_types` | `jpg`, `png`, `gif` (쉼표 구분) |
| `size` | `thumb` / `small` / `med` / `full` |
| `sub_id` | 사용자 구분용 ID |
| `species_id`, `breed_groups`, `exclude_breed_groups`, `lang`, `account_id`, `order_by` | 기타 필터 |

품종 필터 + 상세 포함 예:
```bash
curl -H "x-api-key: YOUR-API-KEY" "https://api.thecatapi.com/v1/images/search?limit=2&breed_ids=abys&include_breeds=1"
```

### 기타 이미지 엔드포인트

| 메서드/경로 | 설명 |
|---|---|
| `GET /images/{id}` | 이미지 1건 조회 |
| `GET /images` | 내가 업로드한 이미지 목록 (`?sub_id=` 필터 가능) |
| `POST /images/upload` | 이미지 업로드(비동기). multipart `file`, 또는 JSON `type`(`base64`/`FILE`) + `image` |
| `POST /images/upload-sync` | 업로드 후 분석 완료까지 대기 (`upload.thecatapi.com` 호스트, `auto_crop` 기본 true) |
| `GET /images/{id}/status` | 처리 상태 (예: `"status": "clean"`) |
| `GET /images/{id}/labels` | 품종 식별 라벨 |
| `DELETE /images/{id}` | 업로드한 이미지 삭제 |
| `POST/DELETE /images/{imageId}/public-requests`, `GET .../public-requests/status` | 이미지 공개 요청/취소/상태 |

업로드 예:
```bash
curl -X POST -H "x-api-key: YOUR-API-KEY" -F "file=@/path/cat.jpg" https://api.thecatapi.com/v1/images/upload
```

## 4. 품종 (Breeds)

| 메서드/경로 | 설명 |
|---|---|
| `GET /breeds` | 품종 목록. `limit`(1~1000, 기본 1000), `page`, `order`, `breed_groups`, `exclude_breed_groups`, `lang`, `extinct`, `venom_code` |
| `GET /breeds/search?q=siamese` | 이름(대체 이름 포함) 검색. `q` 1~30자, `limit` 1~100 |
| `GET /breeds/{id}` | 품종 상세 (예: `abys`) |
| `GET /breeds/breed-groups` | 품종 그룹 목록 |
| `GET /breeds/{id}/colours` | 품종별 털 색/무늬 |

```bash
curl -H "x-api-key: YOUR-API-KEY" https://api.thecatapi.com/v1/breeds
curl -H "x-api-key: YOUR-API-KEY" "https://api.thecatapi.com/v1/breeds/search?q=siamese"
curl -H "x-api-key: YOUR-API-KEY" "https://api.thecatapi.com/v1/breeds?breed_groups=Short-haired,Rex"
curl -H "x-api-key: YOUR-API-KEY" "https://api.thecatapi.com/v1/breeds/search?q=rex&exclude_breed_groups=Hairless"
```

다국어(Startup/Enterprise): `/v1/breeds?lang=it`, `/v1/breeds/beng?lang=it`

### 품종 응답 주요 필드 (`/breeds/abys`)
`id`, `name`, `species_id`, `origin`, `country_codes`, `country_code`, `life_span`, `temperament`, `description`, `bred_for`, `perfect_for`, `breed_group`, `history`, `reference_image_id`, `image{id,width,height}`, `venom_codes[]`

실제 호출 예 (`/breeds/search?q=siamese`, 일부):
```json
{
  "id": "siam",
  "name": "Siamese",
  "species_id": "1",
  "life_span": "12-20",
  "temperament": "Active, Agile, Clever, Sociable, Loving, Energetic, Affectionate",
  "origin": "Thailand",
  "country_code": "TH",
  "breed_group": "Short-haired",
  "bred_for": null,
  "perfect_for": null
}
```
- `bred_for`, `perfect_for`는 `null`일 수 있으니 화면에 표시할 때 빈 값 처리가 필요합니다.
- 점수(1~5): `affection_level`, `energy_level`, `intelligence`, `cat_friendly`, `dog_friendly` 등
- 불리언: `hypoallergenic`, `indoor`, `rare` 등
- 전체 스키마(점수/불리언)는 Startup 이상, Free는 핵심 필드만

## 5. 투표 / 즐겨찾기 (Votes / Favourites)

### 즐겨찾기
```bash
# 목록
curl -H "x-api-key: YOUR-API-KEY" https://api.thecatapi.com/v1/favourites
# 생성
curl -X POST -H "Content-Type: application/json" -H "x-api-key: YOUR-API-KEY" \
  -d '{"image_id": "KWdLHmOqc"}' https://api.thecatapi.com/v1/favourites
# 조회 / 삭제
curl -H "x-api-key: YOUR-API-KEY" https://api.thecatapi.com/v1/favourites/FAVOURITE_ID
curl -X DELETE -H "x-api-key: YOUR-API-KEY" https://api.thecatapi.com/v1/favourites/FAVOURITE_ID
```
- 본문에 `sub_id`를 넣으면 어떤 사용자가 즐겨찾기했는지 기록, 목록 조회 시 `?sub_id=`로 필터
- 목록 파라미터: `limit`, `page`, `order`, `image_id`, `sub_id`, `is_deleted`, `attach_image`(이미지 정보 포함) 등

### 투표
```bash
curl -X POST -H "Content-Type: application/json" -H "x-api-key: YOUR-API-KEY" \
  -d '{"image_id": "KWdLHmOqc", "value": 1}' https://api.thecatapi.com/v1/votes
curl -H "x-api-key: YOUR-API-KEY" https://api.thecatapi.com/v1/votes
curl -H "x-api-key: YOUR-API-KEY" https://api.thecatapi.com/v1/votes/VOTE_ID
```
- 본문: `image_id`(필수), `value`(필수, `1` 또는 `-1`), `sub_id`, `country_code`, `ip_address`
- **투표는 삭제할 수 없음.** 되돌리려면 반대 `value`로 다시 투표

## 6. 기타 엔드포인트

| 분류 | 경로 | 비고 |
|---|---|---|
| 카테고리 | `GET /categories`, `GET /categories/{id}` | |
| 고양이 상식(Facts) | `GET /facts`, `/facts/random`, `/facts/breed/{breedId}`, `/facts/breed/{breedId}/random`, `/facts/species/{speciesId}/random`, `/facts/{id}` | Startup 이상. `limit`, `offset`, `lang`, `include_sources` |
| 건강 팁 | `GET /health-tips`, `/health-tips/breed/{breedId}`, `/health-tips/{id}` | Enterprise |
| 품종 규제 | `GET /breeds/bans/countries`, `/breeds/bans/country`, `/breeds/bans/breed/{breedId}` | |
| 반려동물(Pets) | `/pets` 하위 (등록, 초상화 생성, 체형/나이/체중 분석, 방문 기록 등) | Enterprise |
| 계산기 | `POST /calculators/energy-requirement`, `/calculators/weight-loss-plan` | Enterprise |
| 웹훅 | `/webhooks`, `/webhooks/topics`, `/webhooks/{id}`, `.../rotate-secret` | |
| 계정 | `GET /accounts/features`, `/accounts/quota`, `/accounts/capabilities` | |
| Sub ID | `GET /sub-ids` | `start_date`, `end_date` (ISO 8601) |

## 7. 웹페이지(HTML/JS) 개발 시 참고

- 가장 단순한 사용: `GET /images/search?limit=N` 으로 고양이 사진 목록을 받아 `url`을 `<img src>`에 사용
- 품종 정보를 함께 보려면 `include_breeds=1` 또는 `has_breeds=1`과 조합
- **API 키는 브라우저 코드에 노출됨.** 학습/개인용이면 괜찮지만, 공개 배포 시에는 별도 서버(프록시)에서 키를 숨기는 것이 안전
- 사용자별 즐겨찾기/투표는 `sub_id`로 구분
- 에러 처리: `res.ok` 확인 후 상태 코드별 안내 (401: 키 오류, 429: 한도 초과 등 — 정확한 코드는 공식 "Errors and limits" 문서 확인)

## 8. 실제 테스트 결과 (2026-11-09)

프로젝트의 `.env`(`CAT_KEY`)로 호출한 결과입니다. 키 값은 기록하지 않았습니다.

| 엔드포인트 | 결과 |
|---|---|
| `GET /images/search?limit=3` | 200, 이미지 3장 |
| `GET /breeds/search?q=siamese` | 200, Siamese 정보 |
| `GET /breeds/abys` | 200, Abyssinian 상세 |
| `GET /accounts/features` | 200, `[]` (Startup/Enterprise 기능 없음 → 무료 플랜으로 추정) |

- `.env` 변수 이름은 `CAT_KEY`입니다.
- 순수 HTML/JS 페이지는 `.env`를 읽을 수 없으므로, 키를 코드에 넣으면 브라우저에 노출됩니다.

## 9. 확인하지 못한 내용
- 요청 한도 수치, 에러 코드 표 ("Errors and limits" 페이지)
- 일부 요청 본문 스키마 (`CreateFavouriteDto` 등 — 스펙이 잘려 확인 불가. 위 예제의 `image_id`, `sub_id` 사용은 공식 예제로 확인됨)
