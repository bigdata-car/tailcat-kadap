# tailcat-kadap

KADAP 개발 서버의 포트를 안전하게 연결하고, 필요할 때 외부 미리보기 URL을 만드는 Bash 기반 도구입니다.

## 목적

- `dev01-vm205`의 내부 서비스를 클라이언트 로컬 포트로 전달
- 연결을 `start`, `stop`, `status`로 관리
- 포트 매핑을 `add`, `del`, `list`로 관리
- `tunnel` 실행 시에만 서버에 Cloudflare Quick Tunnel을 설치하고 외부 HTTPS URL 생성
- `reset`으로 연결별 설정과 원격 서비스를 확인 절차 후 정리

## 활용 오픈소스

- [Tailscale Tailcat](https://github.com/tailscale/tailcat): Tailscale 데이터 플레인을 이용한 암호화 TCP 연결
- [cloudflared](https://github.com/cloudflare/cloudflared): Cloudflare Quick Tunnel 클라이언트
- [Cloudflare Quick Tunnels 문서](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/do-more-with-tunnels/trycloudflare/)

`cloudflared`는 `setup` 때 설치하지 않습니다. `tunnel`을 실행할 때 서버에 없으면 해당 시점에 설치합니다. Quick Tunnel은 개발·테스트용이며 프로세스가 종료되면 공개 URL도 종료되고, URL은 재시작 시 바뀔 수 있습니다.

## 스크린샷

![tailcat status 예시](docs/tailcat-status.svg)

## 설치

```bash
install -d -m 755 ~/.local/bin
install -m 755 tailcat ~/.local/bin/tailcat
export PATH="$HOME/.local/bin:$PATH"
```

연결 설정은 다음으로 생성합니다.

```bash
tailcat setup
```

## 사용법

```text
tailcat                         도움말 표시
tailcat start                   저장된 포워더 시작
tailcat stop                    포워더 중지
tailcat status                  포워더와 Tunnel session 상태 표시
tailcat list                    포트 매핑 목록
tailcat add <remote> <local>   원격 포트 추가 및 로컬 매핑
tailcat del <local>             로컬 매핑 제거
tailcat tunnel [remote-port]   외부 Quick Tunnel 생성 (기본 3000)
tailcat reset                   확인 후 연결 설정 초기화
```

예시:

```bash
tailcat start
tailcat add 22 2022
tailcat tunnel 3000
tailcat status
```

`tailcat add`는 로컬 포트가 비어 있는지, 원격 대상 포트가 실제로 리스닝 중인지 확인합니다. `tailcat reset`은 `RESET <연결명>` 확인 문자열을 요구하며, 클라이언트 바이너리와 `tailcat` 명령은 유지합니다.

## 주의사항

Quick Tunnel URL은 공개 인터넷에서 접근 가능하므로 민감한 관리 화면을 그대로 노출하지 마세요. 상시 운영에는 계정 기반 Cloudflare Tunnel과 인증 정책을 사용해야 합니다.
