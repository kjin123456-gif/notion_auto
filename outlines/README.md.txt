[README.md 개요]
위치: /README.md
역할: 사람이 읽는 설치·사용 안내서

1. 프로젝트 소개 (한 문단)
2. 사전 준비
   - Claude Code 설치
   - 노션 MCP(또는 API) 연결
   - GWS CLI 설치 및 구글 계정 인증
3. 초기 설정 순서
   - 노션에 트리거 DB와 일정표 DB 생성 (docs/notion-setup.md)
   - config/notion.json에 트리거 DB ID 입력
   - .env.example을 복사해 .env 작성
   - scripts/check-env.ps1로 환경 점검
4. 사용 예시
   - 등록: "개인:친구와 카페약속 토요일 3시"
   - 조회: "오늘 마감작업 확인해줘"
   - 실행: 보고 확인 후 승인 → /run
5. 슬래시 명령 목록 (/sync-triggers, /run)
6. 폴더 구조 설명
7. 자주 발생하는 문제 해결 (인증 만료, 키워드 미인식 등)
