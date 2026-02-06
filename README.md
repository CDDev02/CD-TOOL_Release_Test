## 배포 세팅 가이드 (현재 운영 방식)

### 1) 채널 구조
- **App 채널**: 버전 태그 릴리즈 (예: `v1.6.0`)
- **Installer 채널**: 고정 태그 릴리즈 `installer`
- **Index 채널**: 저장소 `main` 루트 파일
  - `update-index.json`
  - `update-index.sig`
  - `installer-index.json`
  - `installer-index.sig`

Updater/Setup은 `main`의 index + sig를 먼저 읽고 서명 검증 후 릴리즈를 조회합니다.

---

### 2) App 배포 (일반 기능 배포)
앱 코드(DLL/VLX)만 바뀐 경우:

1. `obfuscate.cmd` 실행
2. App 릴리즈 태그 입력 (예: `v1.6.0`)
3. `Bundle\dll` 산출물 업로드:
   - `CDTOOL.dll`
   - `CDTOOL.Core.dll`
   - `CDTOOL_LSPLOAD.VLX`
   - `integrity.json`
   - `integrity.sig`
4. `Bundle\repo-main`의 index 4종을 `main` 루트에 반영
   - `update-index.json/.sig`
   - `installer-index.json/.sig` (항상 같이 갱신됨)

주의:
- App 릴리즈(`v1.x.x`)에는 `CDTOOL_Setup.exe`, `CDTOOL_Updater.exe`를 올리지 않습니다.

---

### 3) Installer 배포 (Setup/Updater 변경 배포)
Setup/Updater가 바뀐 경우:

1. `obfuscate.cmd` 실행 (App 태그는 현재 운영 버전 기준 입력)
2. `installer` 태그 릴리즈의 Assets 교체:
   - `CDTOOL_Setup.exe`
   - `CDTOOL_Updater.exe`
   - `integrity.json`
   - `integrity.sig`
3. `Bundle\repo-main`의 index 4종을 `main` 루트에 반영
4. `installer-index.json`의 `revision`은 반드시 증가 상태여야 함
   - 스크립트 기본값: UTC ticks (자동 증가)

---

### 4) 테스트 채널
- ON: `on_test_channel.cmd`
- OFF: `off_test_channel.cmd`

테스트 채널 플래그 파일:
- `C:\ProgramData\CDTOOL\test-release-remote.txt`
- `C:\ProgramData\CDTOOL\Installer\test-release-remote.txt`

테스트 시에는 `CD-TOOL_Release_Test`를 조회합니다.

---

### 5) 운영 체크리스트
- App 배포: `vX.Y.Z` 릴리즈 + `Bundle\dll` 업로드 완료
- Installer 배포(필요 시): `installer` 릴리즈 assets 교체 완료
- `main` 루트 index 4종(`json/sig`) 반영 완료
- index 서명 파일(`*.sig`) 누락 없음
- Setup/Updater를 앱 릴리즈에 섞어 올리지 않았는지 확인

---

### 6) 자주 발생하는 이슈
- **“설치 프로그램 확인 필요” 토스트**
  - 원인: 로컬 Setup 파일과 `installer` 채널 기준(무결성)이 불일치
  - 보통 App 릴리즈의 Setup으로 설치했을 때 발생
  - 해결: 사용자 배포용 Setup은 항상 `installer` 릴리즈에서 제공
