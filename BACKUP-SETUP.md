# 새솔GP 데이터베이스 자동 백업

이 설정은 운영 중인 Supabase PostgreSQL 데이터베이스의 전체 논리 백업을 **매일 한국시간 오전 2시 10분** GitHub Actions에서 수행합니다.

## 최초 1회 설정 (필수)

GitHub 저장소 Settings → Secrets and variables → Actions → New repository secret 에 다음 두 값을 추가하세요.

1. `SUPABASE_BACKUP_DB_URL`: **운영 중인 sesolGP** 프로젝트의 PostgreSQL 연결 문자열. Supabase 대시보드 Connect에서 Session pooler 연결 문자열을 복사하고 데이터베이스 비밀번호를 입력합니다. GitHub 코드나 채팅에 비밀번호를 게시하지 마세요.
2. `BACKUP_ENCRYPTION_PASSWORD`: 충분히 길고 무작위인 백업 암호. **분실하면 백업 복원이 불가능**합니다. 비밀번호 관리 프로그램 등 GitHub와 별도 장소에 보관하세요.

설정 후 GitHub Actions → Encrypted daily Supabase backup → Run workflow 로 첫 실행을 수동 수행하고 성공 및 아티팩트 생성 여부를 확인하세요. 설정 전에는 자동 백업이 성공하지 않습니다.

## 백업 정책

- 매일 02:10 KST, 또는 수동 실행
- pg_dump custom 포맷으로 데이터베이스 논리 백업 (테이블·스키마·데이터)
- AES-256-CBC PBKDF2 암호화, 복호화 및 원본 일치 확인 후 업로드
- GitHub Actions artifact 보존 30일 (보존 기간 이후 자동 삭제)
- 복구 시 아티팩트의 `.dump.enc`를 내려받아 동일 암호로 복호화한 후 `pg_restore` 사용
- 복구는 **새로운 테스트 데이터베이스에 먼저 검증**한 다음 운영 DB에 적용할 것

## 주의사항

- Supabase Auth, Storage 파일 자체, 프로젝트 설정 등은 이 PostgreSQL 논리 백업만으로 모두 복구되지 않을 수 있습니다. 별도 백업 필요.
- 브라우저 localStorage에 저장되는 **경영관리 자금계획 데이터는 이 작업에 포함되지 않습니다**.
- 30일을 넘는 장기보관은 별도의 외부 저장소가 필요합니다.
- GitHub Actions가 비활성화되거나 DB 연결 비밀번호가 변경되면 백업이 실패할 수 있으므로 Actions 실행 결과를 정기적으로 확인하세요.
- 암호화된 아티팩트와 함께 업로드하는 manifest.txt에는 DB 테이블 및 객체 이름이 포함될 수 있습니다.
