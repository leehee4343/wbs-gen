# WBS 일정 분석 프로그램

정적 웹 애플리케이션입니다. 별도 서버 없이 GitHub Pages에서 실행할 수 있으며, WBS 엑셀을 브라우저에서 분석합니다.

## 구성

- 화면: `index.html`, `js/app.js`, `css/style.css` (빌드 없는 Vanilla JS)
- 설정: `config/supabase.config.js`, `config/emailjs.config.js`
- 데이터베이스: Supabase 스키마 `wbs_gen` (테이블 `wbs_gen_projects`, `wbs_gen_members`, `wbs_gen_analyses`). 새 환경은 `sql/supabase_setup_wbs_gen.sql`로 생성

## 메뉴

- 일정/공정율 분석: 대시보드, 금주 한 일, 차주 할 일, 지연 업무, WBS 전체 일정 (비밀번호 없이 조회)
- 관리자 전용 서비스: 프로젝트 관리, WBS 업데이트, 분석 결과 조회, 구성원 관리, 일정 관련 메일보내기 (관리자 비밀번호 필요)

## GitHub Pages에서 메일 발송 활성화

메일 직접 발송은 EmailJS 브라우저 SDK를 사용합니다. GitHub Pages는 서버 비밀값을 안전하게 보관할 수 없으므로, **EmailJS Public Key만** 사용해야 합니다. Private Key, 메일 비밀번호, API Secret은 저장소에 절대 올리지 마세요.

1. EmailJS에서 메일 서비스와 템플릿을 만듭니다.
2. 템플릿에 아래 변수를 연결합니다.

   - 받는 사람: `{{to_email}}`
   - 참조: `{{cc_email}}`
   - 제목: `{{subject}}`
   - 본문(평문 호환): `{{message}}`
   - 본문(에디터 서식 포함): `{{{message_html}}}`
   - 회신 주소: `{{reply_to}}`
   - 발신 표시명: `{{from_name}}`

3. [config/emailjs.config.js](config/emailjs.config.js)의 `publicKey`, `serviceId`, `templateId`, `fromName`, `replyTo`를 입력합니다. `replyTo`에는 회신을 받을 업무용 이메일을 입력합니다.
4. EmailJS 대시보드에서 허용 도메인/Origin 제한을 사용 중이라면 다음을 등록합니다.

   - `https://<GitHub-사용자명>.github.io`
   - 프로젝트 사이트: `https://<GitHub-사용자명>.github.io/<저장소명>/`

5. 변경사항을 GitHub 기본 브랜치에 올린 뒤 **Settings → Pages → Deploy from a branch**에서 해당 브랜치와 `/(root)`를 선택합니다.
6. 배포 사이트에서 `구성원 관리`에 수신자를 등록하고, `일정 관련 메일보내기`에서 테스트 메일을 발송합니다.

`config/emailjs.config.js`가 비어 있으면 메일 메뉴에서 브라우저별 임시 설정을 입력할 수 있으며, 이 경우 설정은 해당 브라우저에만 저장됩니다. 배포 환경에서는 설정 파일 방식이 권장됩니다.

### 로컬 테스트

로컬에서도 같은 방식으로 실제 발송을 시험할 수 있습니다. `config/emailjs.config.js` 또는 메일 메뉴의 브라우저 설정에 세 식별자를 입력하고, EmailJS의 허용 도메인에 `http://localhost`를 추가한 뒤 로컬 서버 주소로 접속하세요. `file://` 주소는 EmailJS의 도메인 제한에 걸릴 수 있어 권장하지 않습니다.

## 주의

- GitHub Pages는 정적 호스팅입니다. 프로젝트, 구성원, 분석 결과, 업로드한 엑셀은 Supabase에 저장되어 어느 브라우저에서나 같은 내용을 봅니다. 브라우저의 IndexedDB(WbsGenDB)에는 불러온 결과의 임시 복사본과 Supabase 저장 실패 시의 예비 저장본만 남습니다.
- EmailJS 공개 식별자는 클라이언트에 노출됩니다. EmailJS 쪽에서 도메인 제한, 발송 한도 및 스팸 방지 옵션을 설정하세요.
- 관리자 메뉴 비밀번호는 현재 클라이언트 코드 기반의 간단한 화면 보호입니다. 실제 권한 관리는 서버 인증이 필요합니다.
