# Admin 접속 기록 시스템 - Supabase 설정 가이드 (GitHub Pages + GitHub Actions)

## 🏗️ 아키텍처

```
GitHub Pages (admin.html) ←→ Supabase (PostgreSQL)
                                     ↑
                     GitHub Actions (1주일마다 자동 요청)
                            └─ 일시중지 방지
```

**특징:**
- 💻 별도 서버 불필요 (GitHub Pages만 사용)
- 🔄 직접 Supabase 연동 (JavaScript SDK)
- 🤖 GitHub Actions가 1주일마다 자동 ping 요청 (일시중지 방지)
- 📊 실시간 데이터 동기화
- 📝 Git push로 자동 배포

---

## 1️⃣ Supabase 프로젝트 생성

1. [Supabase.com](https://supabase.com) 방문 후 GitHub로 로그인
2. 새 프로젝트 생성
   - Project name: `autodrive-admin`
   - Database password: 안전한 비밀번호 설정
   - Region: `Asia - Singapore` (빠른 속도)

---

## 2️⃣ 데이터베이스 테이블 생성

1. Supabase 대시보드 → **SQL Editor** 클릭
2. 다음 쿼리 실행:

```sql
-- access_logs 테이블 생성
CREATE TABLE access_logs (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  time TEXT NOT NULL,
  ip TEXT NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 인덱스 생성 (성능 향상)
CREATE INDEX idx_access_logs_created_at ON access_logs(created_at DESC);

-- Row Level Security
ALTER TABLE access_logs ENABLE ROW LEVEL SECURITY;

-- 정책: 모든 사용자가 읽기 가능
CREATE POLICY "Anyone can read logs"
  ON access_logs FOR SELECT
  USING (true);

-- 정책: 모든 사용자가 쓰기 가능
CREATE POLICY "Anyone can insert logs"
  ON access_logs FOR INSERT
  WITH CHECK (true);

-- 정책: 모든 사용자가 삭제 가능
CREATE POLICY "Anyone can delete logs"
  ON access_logs FOR DELETE
  USING (true);
```

---

## 3️⃣ Supabase API 키 복사

1. Supabase 대시보드 → **Settings** → **API**
2. 다음 정보 복사:
   - **Project URL**: `https://xxxxxxxxxxxx.supabase.co`
   - **anon public key**: `eyJhbGciOiJIUzI1NiIs...`

---

## 4️⃣ admin.html 설정

[page/admin.html](admin.html)을 열고 다음을 수정:

**줄 229-230:**
```javascript
const SUPABASE_URL = 'https://YOUR_SUPABASE_URL.supabase.co';
const SUPABASE_KEY = 'YOUR_SUPABASE_ANON_KEY';
```

예시:
```javascript
const SUPABASE_URL = 'https://abcdefgh.supabase.co';
const SUPABASE_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...';
```

---

## 5️⃣ GitHub Actions 설정 (자동 일시중지 방지)

[.github/workflows/keep-alive.yml](.github/workflows/keep-alive.yml)을 열고 수정:

**YOUR_SUPABASE_URL 수정** (4곳):
```bash
# 찾기: YOUR_SUPABASE_URL
# 바꾸기: xxxxxxxxxxxx.supabase.co (https://제외)
```

**YOUR_SUPABASE_ANON_KEY 수정** (2곳):
```bash
# 찾기: YOUR_SUPABASE_ANON_KEY
# 바꾸기: eyJhbGciOiJIUzI1NiIs...
```

---

## 6️⃣ Git Push & 배포

```bash
cd page
git add -A
git commit -m "setup: Supabase admin panel with GitHub Actions keep-alive"
git push origin main
```

GitHub Pages에서 자동 배포됨!

---

## 7️⃣ 검증

### 로컬 테스트
- `admin.html`을 브라우저에서 열기
- 접속 기록이 자동 저장되는지 확인

### 배포 확인
- `kw0n.me/admin` 방문
- 접속 기록이 표시되는지 확인
- CSV 다운로드, 기록 초기화 기능 테스트

### GitHub Actions 확인
1. GitHub 레포 → **Actions** 탭
2. `Keep GitHub Pages Alive` workflow 확인
3. 매주 월요일 00:00 UTC에 자동 실행됨

---

## 📊 기능

✅ **자동 접속 기록**
- 페이지 방문 시 시간 + IP 자동 저장
- 포맷: `26-03-08 15:31 PM / 203.0.113.42`

✅ **실시간 동기화**
- 5초마다 최신 데이터 갱신
- 다중 사용자 접근 시 실시간 반영

✅ **데이터 관리**
- 📥 CSV로 내보내기
- 🗑️ 기록 초기화
- 📊 통계 표시 (총 접속 / 오늘 접속)

✅ **자동 일시중지 방지**
- GitHub Actions가 1주일마다 요청 전송
- GitHub Pages 서버 활성 상태 유지

---

## 🔧 트러블슈팅

### admin.html에서 데이터가 안 나타남
```
❌ 원인: Supabase URL/Key가 잘못됨
✅ 해결: admin.html의 SUPABASE_URL, SUPABASE_KEY 재확인
```

### 접속 기록이 저장되지 않음
```
❌ 원인: access_logs 테이블이 없음 또는 RLS 설정 오류
✅ 해결: Supabase SQL Editor에서 위의 테이블 생성 쿼리 다시 실행
```

### GitHub Actions가 실행 안 됨
```
❌ 원인: YAML 문법 오류 또는 설정 누락
✅ 해결: 
   1. .github/workflows/keep-alive.yml 문법 확인
   2. YOUR_SUPABASE_URL, YOUR_SUPABASE_ANON_KEY 정확히 입력
   3. GitHub Actions 탭에서 workflow 상태 확인
```

### CORS 오류
```
❌ 원인: Supabase RLS 정책 설정 오류
✅ 해결: 위의 RLS 정책 쿼리가 정확히 실행되었는지 확인
```

---

## 📚 추가 정보

- [Supabase 공식 문서](https://supabase.com/docs)
- [Supabase JavaScript SDK](https://supabase.com/docs/reference/javascript)
- [GitHub Actions Scheduling](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onschedule)

---

**성공! 🎉**

이제 `kw0n.me/admin`에서 실시간 접속 기록을 확인하고, GitHub Actions가 자동으로 서비스를 활성 상태로 유지합니다.

