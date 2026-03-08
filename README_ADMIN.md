# 🚀 Admin Access Logger - 빠른 시작 가이드

**완전 자동화된 GitHub Pages + Supabase 접속 기록** ✨

---

## ⚡ 3가지 간단한 설정

### 1️⃣ admin.html 수정 (229-230줄)
```javascript
const SUPABASE_URL = 'https://YOUR_PROJECT_ID.supabase.co';
const SUPABASE_KEY = 'YOUR_ANON_KEY';
```
Supabase 대시보드 → Settings → API → 복사 & 붙여넣기

### 2️⃣ keep-alive.yml 수정 (YOUR_SUPABASE_URL, YOUR_SUPABASE_ANON_KEY)
동일한 값 입력 (4곳)

### 3️⃣ Git Push
```bash
git add -A
git commit -m "setup: supabase admin panel"
git push origin main
```

---

## ✅ 확인

- **로컬**: admin.html 브라우저에서 열기 → 접속 기록 저장 확인
- **배포**: `kw0n.me/admin` 방문 → 기록 표시 확인  
- **자동화**: GitHub → Actions 탭 → "Keep GitHub Pages Alive" 매주 실행 확인

---

## 📚 자세한 가이드

[SUPABASE_SETUP.md](SUPABASE_SETUP.md) 참고

---

**이게 다입니다! 🎉**
