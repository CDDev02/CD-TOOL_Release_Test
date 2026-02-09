# CD-TOOL Release Channel 배포 가이드

이 문서는 `CD-TOOL_Release`(운영) / `CD-TOOL_Release_Test`(테스트) 레포 배포 절차를 정리한 가이드입니다.

## 1) 채널 구조

- 운영 채널: `CDDev02/CD-TOOL_Release`
- 테스트 채널: `CDDev02/CD-TOOL_Release_Test`
- 앱 릴리즈 태그 예시: `v1.6.2`
- 인스톨러 릴리즈 태그: `installer`
- 인덱스/히스토리 파일은 레포 `main` 루트에서 읽음

## 2) 빌드 산출물 위치

- 앱/설치 파일 번들: `Release/obf/Bundle`
- 앱 릴리즈 업로드 파일: `Release/obf/Bundle/dll/*`
- 설치 릴리즈 업로드 파일: `Release/obf/Bundle/installer/*`
- 레포 main 업로드 파일: `Release/obf/Bundle/repo-main/*`

## 3) 릴리즈 업로드 규칙

### 앱 릴리즈 태그(`vX.Y.Z`) Assets

- `CDTOOL.dll`
- `CDTOOL.Core.dll`
- `integrity.json`
- `integrity.sig`

### installer 태그 Assets

- `CDTOOL_Setup.exe`
- `CDTOOL_Updater.exe`
- `integrity.json`
- `integrity.sig`

### 레포 main 루트 파일

- `update-index.json`
- `update-index.sig`
- `installer-index.json`
- `installer-index.sig`
- `update-history.json`
- `update-history.sig`

## 4) 히스토리 생성 명령

### 테스트 채널 히스토리 생성

```cmd
update-history-test.cmd -PrivateJwkBase64File .\private_jwk_base64.txt
```

### 운영 채널 히스토리 생성

```cmd
update-history-prod.cmd -PrivateJwkBase64File .\private_jwk_base64.txt
```

- `-AppReleaseTag`는 선택값이며, 미입력 시 `Release/update-index.json`의 `tag`를 자동 사용함
- 생성 후 업로드 대상은 `Release/obf/Bundle/repo-main/update-history.json`, `Release/obf/Bundle/repo-main/update-history.sig`

## 5) 표준 배포 순서

1. 코드 수정 후 `obfuscate.cmd`로 빌드/난독화/서명 생성
2. 앱 릴리즈 태그(`vX.Y.Z`) 생성 후 앱 Assets 업로드
3. `installer` 태그 릴리즈에 installer Assets 업로드
4. 채널에 맞는 `update-history-*.cmd` 실행
5. `Release/obf/Bundle/repo-main`의 인덱스/서명/히스토리 파일을 해당 레포 `main` 루트에 업로드
6. 테스트 설치본으로 업데이트 확인

## 6) 채널별 점검 포인트

- 테스트 채널 검증 시 `test-release-remote.txt` 플래그가 활성화되어야 함
- 테스트 히스토리 생성은 반드시 `update-history-test.cmd` 사용
- 운영 배포 전 `update-history-prod.cmd`로 재생성 후 운영 레포에 업로드

## 7) 자주 발생하는 문제

- 문제: 테스트 히스토리에 운영 버전(`v1.6.1`, `v1.6`, `v1.5`)이 섞임  
원인: 운영 repo 기준으로 히스토리를 생성해서 업로드함  
해결: `update-history-test.cmd` 재실행 후 test repo `main`의 `update-history.json/.sig` 교체

- 문제: `Signature verification failed`  
원인: `json`만 바꾸고 `sig`를 같이 교체하지 않음  
해결: `json`과 `sig`를 항상 세트로 업로드

- 문제: `Missing -Channel argument`  
원인: `update-history.cmd` 직접 실행 시 채널 인자 누락  
해결: `update-history-test.cmd` 또는 `update-history-prod.cmd` 사용

## 8) 운영 원칙

- 릴리즈 Assets와 `main` 인덱스/서명은 항상 같은 빌드 기준으로 맞춘다
- 수동 교체 시에도 `*.json`과 `*.sig`는 반드시 함께 교체한다
- 채널 혼동 방지를 위해 히스토리 생성은 채널 전용 cmd만 사용한다
