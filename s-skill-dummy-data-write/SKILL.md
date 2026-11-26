---
name: s-skill-dummy-data-write
description: Generate dummy data CSV/XLSX files for new workspace uploads. Creates people, organization, deal/lead, and product data matching a chosen theme.
---

## Invariants

- CSV 인코딩: UTF-8 with BOM (Excel 호환)
- Header 형식: `{Target} - {fieldName}` (space-hyphen-space)
- Detail field header: `{Target}_{Detail}{N} - {fieldName}` (예: `Deal_Note1 - 내용`)
- 콤마 포함 값은 큰따옴표로 감쌈 (CSV)
- 날짜 형식: YYYY-MM-DD
- XLSX 생성에 프로젝트의 `xlsx` 라이브러리 사용

## Workflow (interactive)

### Step 1: Choose File Format
사용자에게 묻는다:
- **CSV**. plain text, 범용
- **XLSX**. Excel 파일, 포맷 지원

### Step 2: Choose Data Type
사용자에게 묻는다:
- **Deal (딜)** 또는 **Lead (리드)**. 어느 타입 생성

### Step 3: Choose Name Style
사용자에게 묻는다:
- Modern Korean (예: 김철수, 삼성전자)
- Medieval Fantasy (예: 아라곤, 곤도르 왕국)
- Wuxia Novel (예: 장무기, 화산파)
- US Silicon Valley (예: John Smith, TechCorp Inc.)

### Step 4: Choose Row Count
사용자에게 묻는다:
- 생성할 row 수 (10, 30, 50, 100 등)

### Step 5: Generate the File

#### Header 형식 규칙
- 기본 형식: `{Target} - {fieldName}` (space-hyphen-space)
- Detail field 형식: `{Target}_{Detail}{Number} - {fieldName}`
  - 예: `Organization_Note1 - 내용`, `People_Note1 - 내용`, `Deal_Note1 - 내용`, `Lead_Note1 - 내용`

#### Header 구조 (Deal 선택)
```
Organization - 이름,Organization - 업종,Organization - 직원수,Organization - 매출(억),Organization_Note1 - 내용,People - 이름,People - 이메일,People - 전화,People - 소스,People - 직함,People - 직책,People - 직무,People - 고객 여정 단계,People_Note1 - 내용,Deal - 이름,Deal - 파이프라인,Deal - 파이프라인 단계,Deal - 금액,Deal_Note1 - 내용,Product - 이름,Product - 상태,Product - 유형,Product - 금액
```

#### Header 구조 (Lead 선택)
```
Organization - 이름,Organization - 업종,Organization - 직원수,Organization - 매출(억),Organization_Note1 - 내용,People - 이름,People - 이메일,People - 전화,People - 소스,People - 직함,People - 직책,People - 직무,People - 고객 여정 단계,People_Note1 - 내용,Lead - 이름,Lead - 파이프라인,Lead - 파이프라인 단계,Lead - 유형,Lead_Note1 - 내용,Product - 이름,Product - 상태,Product - 유형,Product - 금액
```

### Step 6: Save the File
- 저장 위치: 사용자 home 디렉토리 Desktop 또는 지정된 path
- 파일 이름: `dummy_data_{deal|lead}_{style}_{count}_{timestamp}.{csv|xlsx}`

#### XLSX Generation Method
프로젝트의 `xlsx` 라이브러리로 Node.js 스크립트 생성:
```javascript
const XLSX = require('xlsx');
const data = [...]; // 2D array data
const ws = XLSX.utils.aoa_to_sheet(data);
const wb = XLSX.utils.book_new();
XLSX.utils.book_append_sheet(wb, ws, 'Sheet1');
XLSX.writeFile(wb, 'output.xlsx');
```

## Default Workspace Fields

### People Fields
| Field | Header | Type | Options |
|-------|--------|------|---------|
| name | 이름 | String | - |
| email | 이메일 | String | - |
| phoneNumber | 전화 | String | - |
| source | 소스 | SingleSelect | 홈페이지, 박람회, 웨비나, 소개, 광고, 세일즈 아웃바운드 |
| jobPosition | 직함 | String | - |
| jobTitle | 직책 | SingleSelect | 임원진, 관리자급, 실무자 |
| jobDescription | 직무 | SingleSelect | 영업/사업개발, 마케팅/홍보, IT/개발, 인사/총무, 재무/회계, 생산/제조, 연구/개발, 고객지원/서비스, 구매/조달, 경영/사업기획 |
| peopleStage | 고객 여정 단계 | SingleSelect | 미분류, 리드, MQL, SQL, 영업 대상, 구매 고객, 충성 고객 |

### Organization Fields
| Field | Header | Type | Options |
|-------|--------|------|---------|
| name | 이름 | String | - |
| businessClass | 업종 | SingleSelect | 제조/생산, 금융/보험, 유통/물류, IT/소프트웨어, 건설/부동산, 의료/헬스케어, 미디어/엔터테인먼트, 서비스업, 교육, 공공/비영리 |
| employees | 직원수 | Number | - |
| revenue2 | 매출(억) | Number | - |

### Deal (딜) Fields
| Field | Header | Type | Options / Default |
|-------|--------|------|-------------------|
| name | 이름 | String | - |
| pipeline | 파이프라인 | Pipeline | 세일즈 파이프라인 |
| pipelineStage | 파이프라인 단계 | PipelineStage | 첫 미팅 준비, 고객 니즈 파악, 솔루션 비교 검토, 최종 협상, 계약 승인 절차, 성사, 실패 |
| price | 금액 | Number | - |

### Lead (리드) Fields
| Field | Header | Type | Options / Default |
|-------|--------|------|-------------------|
| name | 이름 | String | - |
| pipeline | 파이프라인 | Pipeline | 리드 파이프라인 |
| pipelineStage | 파이프라인 단계 | PipelineStage | 새 리드, 연락 시도 중, 연락 완료, 영업 대상 확정, 영업 대상 보류 |
| type | 유형 | SingleSelect | 신규, 업셀, 재시도 |

### Product Fields
| Field | Header | Type | Options |
|-------|--------|------|---------|
| name | 이름 | String | - |
| status | 상태 | SingleSelect | active, inactive (활성, 비활성으로 표시) |
| type | 유형 | SingleSelect | 일반, 구독 (월간), 구독 (연간) |
| price | 금액 | Number | - |

### Note Fields
| Field | Header | Type | Description |
|-------|--------|------|-------------|
| content | 내용 | String | 30자 이하, 테마에 맞는 내용 |

#### Note Header 형식
- `{Target}_Note{N} - 내용` 형식 사용
- 예: `Lead_Note1 - 내용`, `Deal_Note1 - 내용`, `People_Note1 - 내용`, `Organization_Note1 - 내용`

## Example Data by Name Style

### Modern Korean
- Organizations: 삼성테크, 현대솔루션, SK이노베이션, LG디지털, 카카오엔터프라이즈
- People: 김철수, 이영희, 박민수, 정지원, 최수진
- Products: 기업용 솔루션, 클라우드 서비스, 데이터 분석 플랫폼

### Medieval Fantasy
- Organizations: 곤도르 왕국, 로한 기마단, 엘프 연합, 드워프 대장간, 마법사 길드
- People: 아라곤, 레골라스, 김리, 간달프, 프로도
- Products: 미스릴 갑옷, 엘프 활, 드워프 도끼, 마법 지팡이

### Wuxia Novel
- Organizations: 화산파, 무당파, 소림사, 아미파, 곤륜파
- People: 장무기, 소용녀, 곽정, 황용, 양과
- Products: 천잠사 의복, 용천검, 비급 무공서, 영단묘약

### US Silicon Valley
- Organizations: TechCorp Inc., Innovation Labs, Digital Solutions, Cloud Systems, AI Dynamics
- People: John Smith, Sarah Johnson, Michael Chen, Emily Davis, David Kim
- Products: Enterprise SaaS, Cloud Platform, AI Analytics, DevOps Suite

## Data Generation Rules

1. **Organization과 People는 1:1 매핑**
   - 각 row가 organization 하나와 person 하나 포함

2. **Deal/Lead는 person에 연결**
   - 각 row가 unique Deal 또는 Lead 생성

3. **Product는 선택**
   - 50% 확률로 product 포함

4. **Note는 무작위로 생성**
   - 각 object (Organization, People, Deal/Lead)에 30-50% 확률로 Note 포함
   - 내용은 30자 이하, 테마에 맞는 자연스러운 메모
   - 예시:
     - Korean: "다음 주 미팅 예정", "결재 승인 대기 중"
     - Fantasy: "마법 계약서 검토 필요", "드래곤 출몰 주의"
     - Wuxia: "내공 수련 중", "비급 전수 논의"
     - Silicon Valley: "Demo scheduled next week", "Awaiting budget approval"

5. **Field 값 생성**
   - String field: 테마에 맞는 값 생성
   - SingleSelect field: 옵션에서 무작위 선택
   - Number field: 적절한 범위의 무작위 숫자
   - Pipeline/stage: won/lost stage 제외하고 무작위 선택

6. **이메일 생성 규칙**
   - Korean: {이름}@{회사도메인}.co.kr
   - Fantasy/Wuxia: {이름}@{세계관도메인}.com
   - Silicon Valley: {firstname}.{lastname}@{company}.com

7. **전화번호 생성 규칙**
   - Korean: 010-XXXX-XXXX
   - International: +1-XXX-XXX-XXXX

## Notes

- Pipeline stage는 default 워크스페이스 값 사용
- 인코딩, header 형식 등 제약은 위 Invariants 섹션 참조

## Example Interaction

```
User: /s-skill-dummy-data-write

Claude: I'll generate dummy data. A few questions first.

1. Which file format?
   - CSV
   - XLSX (Excel)

2. Which data type?
   - Deal (딜)
   - Lead (리드)

3. Which name style?
   - Modern Korean
   - Medieval Fantasy
   - Wuxia Novel
   - US Silicon Valley

4. How many rows?
   - 10
   - 30
   - 50
   - 100
```
