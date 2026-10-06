알아챔 by HLEX
> 건강한 변화는 알아차림에서 시작됩니다.
손톱 물어뜯기, 입술 뜯기, 피부 뜯기 같은 무의식적 신체 반복행동(BFRB)을 기록하고,
사용자가 직접 여러 대체 행동을 실험하며 자신에게 가장 효과적인 방법을 찾아가는
개인 맞춤형 헬스케어 프로토타입입니다.
팀/프로젝트명: HLEX (Healthier Life Experience)
앱 이름: 알아챔
진행 형태: 1인 캡스톤 프로젝트 (개발 기간 약 1개월)
---
1. 문제 정의
손톱이나 입술을 뜯는 사람은 대부분 행동이 끝난 뒤에야 알아차린다. 기존 습관관리
앱들은 행동 횟수를 세거나 감지 후 경고하는 데 집중하지만, 같은 행동이라도
원인(건조함, 불안, 지루함, 집중 등)이 사람마다 다르기 때문에 똑같은 대처법이
모두에게 효과적이지 않다.
알아챔은 행동을 막는 앱이 아니라, 알아차림 → 상황 파악 → 대체 행동 실험 →
효과 평가 → 개인 맞춤 방법 발견이라는 과정을 지원한다.
2. 핵심 차별점
기존 습관관리 앱	알아챔
행동 횟수 기록 / 연속 일수 표시	알아차린 횟수와 시도한 횟수를 누적 (실패해도 초기화 없음)
센서/카메라로 감지 후 경고	사용자가 스스로 알아차리고 기록
모두에게 같은 대처법 제시	상황·느낌별로 여러 대체 행동을 직접 실험하고 효과를 비교
> ⚠️ **정직한 범위 고지**: 현재 추천 로직은 "느낌 → 대체 행동" 매핑과 사용자의
> 과거 실험 결과를 반영하는 **규칙 기반(rule-based) 시스템**입니다. 문서상
> "AI 활용"이라는 표현을 쓴 적이 있지만, 머신러닝 모델이 아니라 조건 분기 로직이며,
> 데이터가 더 축적되면 개인화 모델로 확장할 수 있는 구조로 설계했습니다.
3. 핵심 사용자
1차 사용자: 손톱·입술·피부를 무의식적으로 뜯거나 만지는 사람
초기 집중 사용자: 공부/스마트폰 사용 중 해당 행동이 나타나는 10대 후반~20대
(특히 과제·시험으로 장시간 집중하는 대학생)
4. 주요 기능
온보딩 — 개선할 행동 선택 → 주요 발생 상황 선택 → 현실적인 주간 목표 설정
원터치 알아차림 기록 — 행동 선택 즉시 저장, 활동/느낌/강도는 선택 입력
순간 개입 코치 — 방금 느낀 감정에 맞는 대체 행동을 즉시 제안
대체 행동 실험실 — 실험 전/후 충동 강도를 기록하고 효과를 누적 비교
나의 HLEX 지도 — 가장 잦은 시간대·상황·느낌, 가장 효과적인 방법을 자동 계산
예방 모드 — 위험 시간대 전에 집중 모드를 시작하고 종료 후 체크인
계정/데이터 관리 — 이메일 로그인·회원가입, 데이터 내보내기(JSON), 전체 초기화
5. 기술 스택
프론트엔드: 순수 HTML / CSS / Vanilla JS (프레임워크 없는 단일 파일, `index.html`)
백엔드: Supabase (PostgreSQL + Auth + Row Level Security)
배포: GitHub Pages / Netlify / Vercel 중 택1 (정적 호스팅이면 충분)
6. 실행 방법
6-1. Supabase 프로젝트 준비
supabase.com에서 무료 프로젝트를 생성합니다.
SQL 에디터에서 아래 스키마를 실행합니다. (`index.html` 상단 주석에도 동일하게 포함되어 있습니다.)
```sql
create table profiles (
  id uuid references auth.users primary key,
  username text unique,
  onboarded boolean default false,
  behaviors text[],
  primary_behavior text,
  situations text[],
  goal text,
  created_at timestamptz default now()
);

create table records (
  id uuid primary key,
  user_id uuid references auth.users not null,
  ts timestamptz not null,
  behavior text, activity text, feeling text, intensity int
);

create table experiments (
  id uuid primary key,
  user_id uuid references auth.users not null,
  ts timestamptz not null,
  behavior text, feeling text, method text,
  before_val int, after_val int, feedback text
);

create table prevention_sessions (
  id uuid primary key,
  user_id uuid references auth.users not null,
  ts timestamptz not null,
  situation text, occurred text
);

alter table profiles enable row level security;
alter table records enable row level security;
alter table experiments enable row level security;
alter table prevention_sessions enable row level security;

create policy "own profile" on profiles for all using (auth.uid() = id);
create policy "own records" on records for all using (auth.uid() = user_id);
create policy "own experiments" on experiments for all using (auth.uid() = user_id);
create policy "own prevention" on prevention_sessions for all using (auth.uid() = user_id);
```
프로젝트 설정 > API 메뉴에서 Project URL과 anon public key를 복사합니다.
6-2. 코드에 키 연결
`index.html` 상단의 다음 두 줄을 본인 값으로 교체합니다.
```js
const SUPABASE_URL = 'YOUR_SUPABASE_PROJECT_URL';
const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY';
```
> anon key는 클라이언트에 노출되도록 설계된 공개용 키이며, 실제 데이터 보호는
> 위에서 설정한 Row Level Security 정책이 담당합니다. 단, `service_role` 키는
> 절대 클라이언트 코드나 저장소에 포함하지 않습니다.
아이디 기반 로그인
이메일 대신 아이디로 로그인할 수 있도록, 내부적으로 아이디를
`아이디@hlex.local` 형태의 가짜 이메일로 변환해 Supabase Auth에 전달합니다.
이 방식을 쓰려면 Supabase 대시보드 Authentication → Providers → Email
메뉴에서 Confirm email 옵션을 꺼두어야 합니다. (실제로 받을 수 없는
주소라 인증 메일이 전달되지 않기 때문입니다.)
6-3. 배포
정적 파일 한 개이므로 빌드 과정이 없습니다. GitHub Pages, Netlify, Vercel 중
편한 방식으로 올리면 바로 접속 가능한 링크가 생성됩니다.
7. 프로젝트 범위와 한계 (Known Limitations)
캡스톤 프로젝트로서의 범위와 한계를 다음과 같이 명확히 밝힙니다.
검증 기간의 한계: 현재 파일럿은 소규모·단기간 테스트로, "개인별 최적 대체
행동을 찾아준다"는 핵심 가치 주장을 통계적으로 입증하기에는 표본과 기간이
충분하지 않습니다. 본 프로토타입은 메커니즘이 작동하는지 보여주는 데모이며,
장기 효과를 증명하는 결과로 해석하지 않습니다.
인과관계 미검증: 실험 전후 충동 강도 감소가 대체 행동 자체의 효과인지,
단순히 "기록하며 잠시 멈춘 효과(novelty effect)"인지 현재 설계로는 구분할 수
없습니다.
기존 연구와의 연결 필요: BFRB(신체집중반복행동) 분야에는 Habit Reversal
Training, ComB(Comprehensive Behavioral Model) 등 검증된 치료 접근이 있습니다.
현재 대체 행동 추천은 이런 연구를 참고해 설계했으나, 공식적인 임상적 근거로
제시하지는 않습니다.
의료 기기 아님: 이 앱은 자기 관리를 돕는 도구이며 진단, 치료, 전문 의료
서비스의 대체재가 아닙니다. 출혈·감염 의심·심리적 고통이 큰 경우 전문가 상담을
안내합니다.
8. 향후 로드맵
홈 화면 위젯, 스마트워치 연동
손톱/입술 회복 사진 기록
사용자 맞춤 대체 행동 직접 등록
규칙 기반 추천을 데이터 기반 개인화 모델로 확장
전문가(피부과·정신건강의학과)와의 공유용 요약 리포트
9. 개인정보 보호 원칙
꼭 필요한 정보만 수집
기록 삭제 및 전체 데이터 초기화 기능 제공
사진은 기본적으로 기기 내 저장, 카메라 영상은 저장하지 않음
사용자가 원할 때 데이터를 JSON으로 내보낼 수 있음
10. 라이선스
이 저장소의 코드는 학습·포트폴리오 목적의 캡스톤 프로젝트 결과물입니다.
별도 라이선스 명시가 필요하면 MIT License 추가를 권장합니다.
