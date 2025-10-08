# Claude on Rails Template

이 리포지토리는 "claude-on-rails 프로젝트 생성을 위한 템플릿"입니다. Rails 8 최소 구성을 기본으로 하며 Importmap/Propshaft, Hotwire(Turbo/Stimulus), Tailwind CSS(v3), Lucide 아이콘 시스템과 기본 CI(깃허브 액션)를 포함합니다.

## 템플릿 사용법

- GitHub UI에서: 리포지토리 페이지의 “Use this template” 버튼을 눌러 새 저장소를 생성하세요.
- gh CLI에서: 아래처럼 템플릿을 지정해 새 저장소를 만듭니다.
  ```bash
  gh repo create <OWNER>/<NEW_REPO> --private --template to-goingon/new-claude-on-rails-project --clone
  cd <NEW_REPO>
  ```

### 템플릿에서 생성한 새 프로젝트 초기화 절차

```bash
bundle install
bin/rails db:prepare
bin/dev
```

### 템플릿 사용 후 권장 변경 사항

- 앱 이름/타이틀: `app/views/layouts/application.html.erb`의 기본 타이틀("New Claude On Rails Project")을 프로젝트명으로 변경
- README: 프로젝트 목적/도메인에 맞게 본 문서를 커스터마이징
- CI: `.github/workflows/ci.yml`에서 필요 없는 잡 제거/수정
- 배포: Kamal 사용 시 레지스트리/서버 설정과 `config/deploy.yml` 추가

### 주요 스택

- **Rails**: ~> 8.0.2
- **Ruby**: 3.4.3 (`.ruby-version`)
- **DB**: SQLite (개발/테스트/프로덕션 모두 `storage/*.sqlite3`)
- **자산 파이프라인**: Propshaft + Importmap (Node/npm 불필요)
- **Hotwire**: `turbo-rails`, `stimulus-rails`
- **Tailwind CSS v3**: `tailwindcss-rails 3.3.2` + `tailwindcss-ruby 3.4.17`
- **아이콘**: rails_icons gem (~> 1.4) + Lucide 아이콘 라이브러리
- **품질/보안**: Brakeman, RuboCop(omakase)
- **기타**: Kamal(배포 준비), Thruster(Puma 가속), Solid Cache/Queue/Cable 세트

### 사전 요구 사항

- Ruby 3.4.3
- SQLite 3.8+
- Git, OpenSSL, 빌드 도구(일반적인 Linux build-essential)
- Foreman은 `bin/dev` 실행 시 자동 설치됨(루비 젬)

### 설치 및 실행

1. 의존성 설치

```bash
bundle install
```

2. 데이터베이스 준비

```bash
bin/rails db:prepare
```

3. 개발 서버 실행(웹 + Tailwind 워처 동시 실행)

```bash
bin/dev
```

- `Procfile.dev`
  - `web: bin/rails server`
  - `css: bin/rails tailwindcss:watch`
- `bin/dev`는 Foreman 미설치 시 자동 설치 후 `Procfile.dev`를 실행합니다.

### Tailwind CSS(v3) 구성

- 버전: 코어 3.4.17 (`tailwindcss-ruby`), 래퍼 3.3.2 (`tailwindcss-rails`)
- 입력 CSS: `app/assets/stylesheets/application.tailwind.css`
  ```css
  @tailwind base;
  @tailwind components;
  @tailwind utilities;
  ```
- 설정: `config/tailwind.config.js`
  - `content` 경로: `./app/views/**/*`, `./app/helpers/**/*`, `./app/javascript/**/*` 등
  - 폰트 확장 예시 포함(`Inter var`)
- 빌드 산출물: `app/assets/builds/tailwind.css`
- 레이아웃 포함: `app/views/layouts/application.html.erb`
  ```erb
  <%= stylesheet_link_tag "tailwind", "inter-font", "data-turbo-track": "reload" %>
  <%= stylesheet_link_tag :app, "data-turbo-track": "reload" %>
  ```
- 개발 중 수동 빌드(옵션)
  ```bash
  bin/rails tailwindcss:build
  ```

### 애플리케이션 구조 메모

- 라우팅: 기본 root 미설정, 헬스체크 `GET /up` 활성화
- 자산: Propshaft 사용으로 `app/assets` 하위 정적 자산을 단순 제공
- Importmap: JS 의존성은 NPM 없이 `importmap`으로 관리

### 테스트/품질

- 로컬 테스트

```bash
bin/rails test
bin/rails test:system   # 시스템 테스트(Chrome 필요)
```

- GitHub Actions CI: `.github/workflows/ci.yml`
  - `scan_ruby`: Brakeman 정적 분석
  - `scan_js`: `bin/importmap audit`
  - `lint`: RuboCop(omakase) 실행
  - `test`: DB 준비 후 테스트 수행(시스템 테스트용 Chrome 설치 포함)

### 배포

- Kamal 젬이 포함되어 있습니다(옵션). 실제 배포 설정은 이 리포지토리에는 포함되어 있지 않습니다. Kamal 사용 시 `config/deploy.yml` 구성 및 레지스트리/서버 설정이 필요합니다.

### 유용한 명령어

```bash
# 서버 실행(웹 + CSS 워처)
bin/dev

# Tailwind 단발성 빌드
bin/rails tailwindcss:build

# DB 작업
bin/rails db:prepare

# 코드 검사/보안
bin/brakeman --no-pager
bin/rubocop -f github

# Regenerate Swarm Configuration
rails generate claude_on_rails:swarm

# To verify your Rails MCP Server setup:
bundle exec rake claude_on_rails:mcp_status
# This shows:
# Installation status
# Downloaded resources
# Missing resources
```

### Git 브랜치 관리

현재 브랜치를 메인 브랜치와 병합하는 방법:

```bash
# 1. 메인 브랜치로 전환
git checkout main

# 2. 메인 브랜치를 최신 상태로 업데이트
git pull origin main

# 3. 현재 작업 브랜치를 메인으로 병합
git merge <branch-name>

# 4. 변경사항을 원격 저장소에 푸시
git push origin main

# 5. 작업 완료된 브랜치 삭제 (선택사항)
git branch -d <branch-name>
git push origin --delete <branch-name>
```

Fast-forward 병합이 아닌 명시적 병합 커밋을 원하는 경우:
```bash
git merge --no-ff <branch-name>
```

### 라이선스

- 본 프로젝트는 MIT 라이센스를 따릅니다.  
  MIT 라이센스는 소프트웨어를 자유롭게 사용, 복사, 수정, 병합, 배포, 출판할 수 있도록 허용하며,  
  저작권 고지와 라이센스 사본을 소스 코드에 포함해야 합니다.  
  이 라이센스는 소프트웨어에 대한 보증을 제공하지 않습니다.
