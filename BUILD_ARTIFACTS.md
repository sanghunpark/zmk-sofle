# ZMK 빌드 아티팩트 가이드

## 빌드 시 생성되는 파일

GitHub Actions에서 빌드를 실행하면 다음 4개의 `.uf2` 펌웨어 파일이 생성됩니다:

### 📦 생성되는 파일 목록

| 파일명 | 대상 | 용도 | 설명 |
|--------|------|------|------|
| `sofle_left_main.uf2` | **왼쪽 키보드** | 일상 사용 | 기본 펌웨어 (nice_view 디스플레이 포함) |
| `sofle_right_main.uf2` | **오른쪽 키보드** | 일상 사용 | 기본 펌웨어 (커스텀 nice_view 포함) |
| `sofle_left_studio.uf2` | **왼쪽 키보드** | 키맵 편집 | ZMK Studio로 실시간 키맵 수정 가능 |
| `sofle_left_settings_reset.uf2` | **왼쪽 키보드** | 문제 해결 | 모든 설정 초기화 (문제 발생 시) |

---

## 🔍 각 파일의 역할 상세 설명

### 1️⃣ sofle_left_main.uf2
```yaml
board: eyelash_sofle_left
shield: nice_view
```

**역할**: 왼쪽 키보드의 메인 펌웨어
- ✅ 일반적인 키보드 사용을 위한 표준 펌웨어
- ✅ nice_view OLED 디스플레이 지원
- ✅ 키맵 파일(`eyelash_sofle.keymap`)에 정의된 레이아웃 적용
- ⚠️ 펌웨어 업데이트 시 매번 이 파일 사용

**플래싱 방법**:
1. 왼쪽 키보드를 PC에 USB로 연결
2. 리셋 버튼 2회 클릭하여 부트로더 모드 진입
3. `NICENANO` (또는 보드 이름) 드라이브 인식 확인
4. `sofle_left_main.uf2` 파일을 드라이브에 복사
5. 자동으로 재부팅됨

---

### 2️⃣ sofle_right_main.uf2
```yaml
board: eyelash_sofle_right
shield: nice_view_custom
```

**역할**: 오른쪽 키보드의 메인 펌웨어
- ✅ 오른쪽 키보드 전용 펌웨어
- ✅ 커스텀 nice_view 디스플레이 지원
- ✅ 왼쪽 키보드와 Bluetooth로 연결되어 작동
- ⚠️ 왼쪽과 오른쪽 펌웨어는 **반드시 같은 빌드에서 생성된 것**을 사용

**플래싱 방법**:
- 왼쪽과 동일하지만, 오른쪽 키보드에 적용

---

### 3️⃣ sofle_left_studio.uf2
```yaml
board: eyelash_sofle_left
shield: nice_view
snippet: studio-rpc-usb-uart
cmake-args: -DCONFIG_ZMK_STUDIO=y -DCONFIG_ZMK_STUDIO_LOCKING=n
```

**역할**: ZMK Studio를 통한 실시간 키맵 편집
- 🎯 **임시 사용 전용** - 키맵 수정 후 원래 펌웨어로 복원 필요
- ✅ USB 연결 상태에서 웹 브라우저로 키맵 실시간 변경 가능
- ✅ ZMK Studio ([studio.zmk.dev](https://studio.zmk.dev)) 접속하여 사용
- ✅ 설정 잠금 비활성화로 자유로운 편집 가능
- ❌ 일상적인 사용에는 적합하지 않음 (배터리 소모 증가)

**사용 시나리오**:
1. 키맵을 빠르게 테스트하고 싶을 때
2. GitHub에 push하지 않고 임시로 키 배치 변경
3. 여러 레이아웃을 시험해보고 싶을 때

**사용 후 주의사항**:
- ⚠️ 편집 완료 후 반드시 `sofle_left_main.uf2`로 복원
- ⚠️ Studio 모드에서는 전력 소모가 크므로 장시간 사용 비추천

---

### 4️⃣ sofle_left_settings_reset.uf2
```yaml
board: eyelash_sofle_left
shield: settings_reset
```

**역할**: 설정 초기화 (Factory Reset)
- 🆘 **문제 해결 전용** - 키보드가 오작동할 때만 사용
- ✅ 모든 저장된 설정 삭제 (Bluetooth 페어링 포함)
- ✅ EEPROM/Flash 저장소 초기화
- ❌ 일반적으로는 사용할 일 없음

**언제 사용하나요?**:
- ✋ Bluetooth 연결이 안 될 때
- ✋ 키보드가 이상하게 동작할 때
- ✋ 펌웨어 업데이트 후 이상 증상
- ✋ 여러 디바이스와 페어링 꼬였을 때

**사용 방법**:
1. `sofle_left_settings_reset.uf2` 플래싱
2. 자동으로 재부팅 (LED 깜빡임 등으로 확인)
3. 즉시 `sofle_left_main.uf2` 재플래싱 필수
4. Bluetooth 디바이스 재페어링 필요

---

## 🚀 일반적인 사용 워크플로우

### 최초 설정 시:
```
1. sofle_left_main.uf2 → 왼쪽 키보드
2. sofle_right_main.uf2 → 오른쪽 키보드
3. Bluetooth 페어링
```

### 펌웨어 업데이트 시:
```
1. GitHub에서 keymap 수정 후 push
2. Actions에서 빌드 완료 확인
3. Artifacts 다운로드
4. sofle_left_main.uf2 → 왼쪽 플래싱
5. sofle_right_main.uf2 → 오른쪽 플래싱
```

### 키맵 빠르게 테스트할 때:
```
1. sofle_left_studio.uf2 → 왼쪽 플래싱
2. studio.zmk.dev 접속
3. 실시간으로 키 변경하며 테스트
4. 완료 후 sofle_left_main.uf2로 복원
```

### 문제 발생 시:
```
1. sofle_left_settings_reset.uf2 → 왼쪽 플래싱
2. 재부팅 대기
3. sofle_left_main.uf2 → 왼쪽 플래싱
4. sofle_right_main.uf2 → 오른쪽 플래싱
5. 디바이스 재페어링
```

---

## 📁 파일 구조 개선 사항

기존 파일명이 명확하지 않아 다음과 같이 개선되었습니다:

| 기존 | 개선 후 | 개선 이유 |
|------|---------|----------|
| `eyelash_sofle_left-nice_view.uf2` | `sofle_left_main.uf2` | 용도('main')가 명확하게 표시됨 |
| `eyelash_sofle_right-nice_view_custom.uf2` | `sofle_right_main.uf2` | 좌우 구분이 명확해짐 |
| `eyelash_sofle_studio_left.uf2` | `sofle_left_studio.uf2` | 'studio'가 앞으로 나와 임시용임을 강조 |
| `eyelash_sofle_left-settings_reset.uf2` | `sofle_left_settings_reset.uf2` | 초기화 용도가 더 명확하게 표시됨 |

---

## ⚙️ build.yaml 설정

파일명을 명시적으로 지정하기 위해 `build.yaml`에서 `artifact-name`을 사용합니다:

```yaml
include:
  # 왼쪽 키보드 - 일반 사용 펌웨어
  - board: eyelash_sofle_left
    shield: nice_view
    artifact-name: sofle_left_main
  
  # 오른쪽 키보드 - 일반 사용 펌웨어
  - board: eyelash_sofle_right
    shield: nice_view_custom
    artifact-name: sofle_right_main
  
  # 왼쪽 키보드 - ZMK Studio 설정용
  - board: eyelash_sofle_left
    shield: nice_view
    snippet: studio-rpc-usb-uart
    cmake-args: -DCONFIG_ZMK_STUDIO=y -DCONFIG_ZMK_STUDIO_LOCKING=n
    artifact-name: sofle_left_studio
  
  # 왼쪽 키보드 - 설정 초기화용
  - board: eyelash_sofle_left
    shield: settings_reset
    artifact-name: sofle_left_settings_reset
```

---

## 📌 추가 팁

### 파일 식별 방법:
- **main**: 일상 사용용 → 가장 자주 사용
- **studio**: 임시 편집용 → 사용 후 main으로 복원
- **settings_reset**: 긴급 복구용 → 문제 발생 시만 사용

### 안전한 펌웨어 관리:
1. 항상 좌우 펌웨어를 같은 빌드에서 생성된 것으로 사용
2. Studio 펌웨어는 임시 사용만
3. Settings reset 후에는 반드시 main 펌웨어 재설치
4. 중요한 변경사항은 반드시 keymap 파일에 커밋

---

## 🔗 참고 자료

- [ZMK 공식 문서](https://zmk.dev/)
- [ZMK Studio](https://studio.zmk.dev)
- [ZMK Discord](https://zmk.dev/community/discord/invite)

