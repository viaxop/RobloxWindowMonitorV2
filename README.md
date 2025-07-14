# Roblox Window Monitor v2

로블록스(Roblox) 게임 창의 화면 변화를 감지하여 사용자에게 알림을 보내는 모니터링 프로그램입니다. 매크로 실행 중 게임이 중단되거나, 예기치 않은 상황이 발생하는지 원격으로 확인할 수 있습니다.

이전 버전과 달리, 윈도우를 활성화하지 않고 백그라운드 상태에서 화면을 캡처하며, 변화 감지 시 최근 동작을 GIF 파일로 생성하여 더욱 상세한 상황 파악을 돕습니다.

## 주요 특징

- **비밀번호 인증**: 프로그램 실행 시 `config.ini`에 설정된 비밀번호로 서버 인증을 거쳐야만 동작합니다.
- **백그라운드 모니터링**: 로블록스 창이 비활성 상태이거나 다른 창에 가려져 있어도 화면을 감지하고 캡처합니다.
- **메모리 기반 스크린샷 저장**: 디스크에 불필요한 파일을 남기지 않고, 최근 스크린샷을 메모리에만 저장하여 효율적으로 관리합니다.
- **GIF 자동 생성 및 전송**: 화면 변화가 감지되면, 직전까지의 상황이 녹화된 GIF 애니메이션을 생성하여 Discord 웹훅으로 전송합니다.
- **안정적인 윈도우 처리**: 로블록스 창을 닫았다가 다시 실행해도 자동으로 새로운 창을 찾아 모니터링을 재개합니다.
- **화면 깨짐 방지**: 캡처된 화면이 비정상적인 흰색 화면일 경우, 이를 감지하고 재시도하여 오탐을 줄입니다.

## 동작 방식

1.  프로그램을 시작하면 `config.ini` 파일을 읽어 설정을 불러옵니다.
2.  `password` 값으로 인증 서버에 인증을 요청합니다. 인증에 실패하거나 비밀번호가 없으면 프로그램이 종료됩니다.
3.  `WINDOWSCLIENT` 클래스와 `Roblox` 창 제목을 가진 윈도우를 찾습니다.
4.  윈도우를 찾으면, 백그라운드 스레드를 실행하여 `auto_screenshot_interval` 간격으로 전체 화면을 캡처해 메모리에 순차적으로 저장합니다. (`gif_generate_image_count` 개수만큼 유지)
5.  메인 루프에서는 `interval_seconds` 간격으로 `screen_position`에 지정된 특정 영역만 캡처합니다.
6.  이전 캡처본과 현재 캡처본의 구조적 유사도(SSIM)를 비교합니다.
7.  유사도가 `similarity_percent` 임계값 미만으로 떨어지면(화면 변화 감지), 다음과 같은 동작을 수행합니다:
    -   변화가 감지된 시점의 전체 화면을 캡처합니다.
    -   "Change detected" 메시지와 함께 전체 화면 스크린샷을 Discord 웹훅으로 전송합니다.
    -   메모리에 저장된 최근 스크린샷들을 사용해 GIF 파일을 생성합니다.
    -   생성된 GIF 파일을 Discord 웹훅으로 추가 전송합니다.
8.  설정된 `interval_seconds` 간격으로 5~7번 과정을 반복합니다.

## `config.ini` 설정

프로그램의 모든 동작은 `config.ini` 파일을 통해 제어됩니다.

```ini
[DEFAULT]
password = YOUR_SECRET_PASSWORD
webhook_url = YOUR_DISCORD_WEBHOOK_URL
screen_position = 40, 250, 170, 410
interval_seconds = 10
similarity_percent = 91
auto_screenshot_interval = 0.5
gif_generate_image_count = 20
brightness_threshold = 200
white_screen_threshold = 0.8
```

| 설정 항목 | 설명 | 기본값 |
| :--- | :--- | :--- |
| `password` | **(필수)** 프로그램 실행을 위한 인증 비밀번호입니다. | 없음 |
| `webhook_url` | **(필수)** Discord 알림을 받을 웹훅 URL입니다. | 없음 |
| `screen_position` | 변화를 감지할 화면 영역 좌표입니다. (left, top, right, bottom) | `40, 250, 170, 410` |
| `interval_seconds` | 화면 변화를 비교할 시간 간격(초)입니다. | `10` |
| `similarity_percent` | 이미지 유사도 임계값(%)입니다. 이 값보다 낮으면 변화로 간주합니다. | `91` |
| `auto_screenshot_interval` | GIF 생성을 위해 백그라운드에서 스크린샷을 저장하는 간격(초)입니다. | `0.5` |
| `gif_generate_image_count` | GIF 생성에 사용할 최근 스크린샷의 최대 개수입니다. | `20` |
| `brightness_threshold` | 흰색 화면을 판단하는 밝기 임계값입니다. (0-255) | `200` |
| `white_screen_threshold` | 이미지가 흰색 화면으로 간주될 밝은 픽셀의 비율(0.0-1.0)입니다. | `0.8` |

## Discord 알림 예시

화면 변화가 감지되면 아래와 같이 두 개의 메시지가 순차적으로 전송됩니다.

**1. 변화 감지 알림 (전체 스크린샷 포함)**
> Change detected: 2024.09.11 - 00:14:07 (Similarity : 80.06%)

**2. GIF 파일**
> GIF created from 20 recent screenshots (memory-based)
> (GIF 애니메이션 파일 첨부)

## 실행 방법

```bash
윈도우 탐색기 에서 RobloxWindowMonitorV2.exe 더블 클릭
또는 cmd 창을 열어 RobloxWindowMonitorV2.exe 실행
```
*참고: 같은 디렉토리에 config.ini 파일이 존재해야 합니다.*

## License

MIT
