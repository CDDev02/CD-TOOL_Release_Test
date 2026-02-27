# CD-TOOL Release Channel 배포 가이드

이 문서는 `CD-TOOL_Release`(운영) / `CD-TOOL_Release_Test`(테스트) 배포 절차를 정리한 가이드입니다.
구버전 CMD 명칭과 현재 통합 CMD(`cdtool.cmd`) 명칭을 함께 제공합니다.

## 1) 채널 구조

- 운영 채널: `CDDev02/CD-TOOL_Release`
- 테스트 채널: `CDDev02/CD-TOOL_Release_Test`
- 앱 릴리즈 태그 예시: `v1.6.2`
- 인스톨러 릴리즈 태그: `installer`
- 인덱스/히스토리 파일은 각 레포 `main` 루트에서 읽음

## 2) 빌드 산출물 위치

- 앱/설치 파일 번들: `Release/obf/Bundle`
- 앱 릴리즈 업로드 파일: `Release/obf/Bundle/dll/*`
- 설치 릴리즈 업로드 파일: `Release/obf/Bundle/installer/*`
- 레포 `main` 업로드 파일: `Release/obf/Bundle/repo-main/*`

## 3) 릴리즈 업로드 규칙

앱 릴리즈 태그(`vX.Y.Z`) Assets

- `CDTOOL.dll`
- `CDTOOL.Core.dll`
- `integrity.json`
- `integrity.sig`

`installer` 태그 Assets

- `CDTOOL_Setup.exe`
- `CDTOOL_Updater.exe`
- `integrity.json`
- `integrity.sig`

레포 `main` 루트 파일

- `update-index.json`
- `update-index.sig`
- `installer-index.json`
- `installer-index.sig`
- `update-history.json`
- `update-history.sig`

## 4) 명령어(구버전/현재)

빌드/난독화

- 구버전: `obfuscate.cmd`
- 현재: `cdtool.cmd build`

테스트 채널 히스토리 생성

- 구버전: `update-history-test.cmd -PrivateJwkBase64File .\private_jwk_base64.txt`
- 현재: `cdtool.cmd history test -PrivateJwkBase64File .\private_jwk_base64.txt`

운영 채널 히스토리 생성

- 구버전: `update-history-prod.cmd -PrivateJwkBase64File .\private_jwk_base64.txt`
- 현재: `cdtool.cmd history prod -PrivateJwkBase64File .\private_jwk_base64.txt`

테스트 채널 토글

- 구버전 ON: `on_test_channel.cmd`
- 구버전 OFF: `off_test_channel.cmd`
- 현재 ON: `cdtool.cmd channel on`
- 현재 OFF: `cdtool.cmd channel off`
- 현재 상태 확인: `cdtool.cmd channel status`

참고

- `-AppReleaseTag`는 선택값이며, 미입력 시 `Release/update-index.json`의 tag를 자동 사용
- 히스토리 생성 후 업로드 대상:
  - `Release/obf/Bundle/repo-main/update-history.json`
  - `Release/obf/Bundle/repo-main/update-history.sig`

## 5) 표준 배포 순서

1. 코드 수정 후 빌드/난독화/서명 생성
   - 구버전: `obfuscate.cmd`
   - 현재: `cdtool.cmd build`
2. 앱 릴리즈 태그(`vX.Y.Z`) 생성 후 앱 Assets 업로드
3. `installer` 태그 릴리즈에 installer Assets 업로드
4. 채널에 맞는 히스토리 생성 실행
   - 테스트: `update-history-test.cmd` 또는 `cdtool.cmd history test`
   - 운영: `update-history-prod.cmd` 또는 `cdtool.cmd history prod`
5. `Release/obf/Bundle/repo-main`의 인덱스/서명/히스토리 파일을 해당 레포 `main` 루트에 업로드
6. 테스트 설치본으로 업데이트 확인 후 운영 반영

## 6) 채널별 점검 포인트

- 테스트 채널 검증 시 `test-release-remote.txt` 플래그가 활성화되어야 함
- 테스트 히스토리 생성은 반드시 테스트 채널 명령으로 실행
- 운영 배포 전 운영 채널 명령으로 히스토리 재생성 후 운영 레포에 업로드

## 7) 변경 유형별 배포 절차

### 7-1. Installer/Updater만 변경 (DLL 무변경)

적용 대상

- `CDTOOL_Setup.exe`
- `CDTOOL_Updater.exe`

필수 반영 파일

- `installer` 태그 Assets 4종
  - `CDTOOL_Setup.exe`, `CDTOOL_Updater.exe`, `integrity.json`, `integrity.sig`
- 레포 `main` 루트 2종
  - `installer-index.json`, `installer-index.sig`

반영하지 않는 파일

- `update-index.json/.sig`
- `update-history.json/.sig`
- 앱 태그(`vX.Y.Z`) DLL Assets

절차

1. 빌드 실행 (`obfuscate.cmd` 또는 `cdtool.cmd build`)
2. `Release/obf/Bundle/installer/*`를 `installer` 태그에 업로드
3. `Release/obf/Bundle/repo-main/installer-index.json(.sig)`를 대상 채널 레포 `main` 루트에 업로드
4. 설치/업데이터 동작 검증 후 운영 채널 반영

### 7-2. DLL만 변경 (Installer/Updater 무변경)

적용 대상

- `CDTOOL.dll`
- `CDTOOL.Core.dll` (필요 시 앱 번들 DLL)

필수 반영 파일

- 앱 태그(`vX.Y.Z`) Assets
  - `CDTOOL.dll`, `CDTOOL.Core.dll`, `integrity.json`, `integrity.sig`
- 레포 `main` 루트
  - `update-index.json`, `update-index.sig`
  - `update-history.json`, `update-history.sig` (히스토리 반영 시)

반영하지 않는 파일

- `installer` 태그 Assets
- `installer-index.json/.sig`

절차

1. 앱 태그 `vX.Y.Z` 결정
2. 빌드 실행 (`obfuscate.cmd` 또는 `cdtool.cmd build`)
3. `Release/obf/Bundle/dll/*`를 앱 태그(`vX.Y.Z`)에 업로드
4. 채널에 맞는 히스토리 생성 실행
5. `Release/obf/Bundle/repo-main/update-index.*`, `update-history.*`를 해당 채널 레포 `main` 루트에 업로드
6. 테스트 검증 후 운영 반영

## 8) 자주 발생하는 문제

문제: 테스트 히스토리에 운영 버전(`v1.6.1`, `v1.6`, `v1.5`)이 섞임

- 원인: 운영 repo 기준으로 히스토리를 생성/업로드
- 해결: 테스트 채널 히스토리 명령 재실행 후 test repo `main`의 `update-history.json/.sig` 교체

문제: `Signature verification failed`

- 원인: `json`만 바꾸고 `sig`를 같이 교체하지 않음
- 해결: `json`과 `sig`를 항상 세트로 업로드

문제: `Missing -Channel argument`

- 원인: 채널 인자 없이 히스토리 명령 실행
- 해결: 테스트/운영 채널 명시 실행
  - `cdtool.cmd history test`
  - `cdtool.cmd history prod`

문제: 테스트 채널 ON 실패 (`Failed to create ... test-release-remote.txt`)

- 원인: 관리자 권한 부족, 보안 정책, 또는 `ProgramData` 경로 충돌
- 해결:
  - 관리자 권한 CMD로 실행
  - `C:\ProgramData\CDTOOL` / `C:\ProgramData\CDTOOL\Installer`가 폴더인지 확인
  - 아래 파일 생성 가능 여부 직접 확인
    - `C:\ProgramData\CDTOOL\test-release-remote.txt`
    - `C:\ProgramData\CDTOOL\Installer\test-release-remote.txt`

## 9) 운영 원칙

- 릴리즈 Assets와 `main` 인덱스/서명은 항상 같은 빌드 기준으로 맞춘다
- 수동 교체 시에도 `*.json`과 `*.sig`는 반드시 함께 교체한다
- 채널 혼동 방지를 위해 히스토리 생성 시 채널 명령을 명확히 구분한다
