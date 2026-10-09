# 위클리랩 홈페이지

[위클리랩 홈페이지](https://weekly-lab.com)의 소스 저장소입니다.
HTML과 CSS로 서비스 소개, 구축/개발 사례, 의뢰 절차와 상담 연락처를 제공합니다.

회사 소개는 [GitHub 조직 대문](https://github.com/weekly-lab)에서 확인할 수 있습니다.
조직 대문 문서는 [weekly-lab/.github](https://github.com/weekly-lab/.github/blob/main/profile/README.md)에서 관리합니다.

## 로컬 실행

패키지 설치나 빌드 없이 실행할 수 있습니다. 저장소 루트에서 다음 명령을 실행합니다.

```sh
python3 -m http.server 8000 --bind 127.0.0.1
```

브라우저에서 <http://127.0.0.1:8000>에 접속합니다. 종료는 `Ctrl+C`입니다.
`index.html`을 브라우저에서 직접 열어도 됩니다.

## 파일 구성

| 경로 | 내용 |
|---|---|
| `index.html` | 홈페이지 콘텐츠와 상담 링크 |
| `styles.css` | 스타일과 반응형 레이아웃 |
| `assets/` | 로고와 서비스 소개 이미지 |
| `.github/workflows/deploy.yml` | 정적 파일 자동 배포 |
| `deploy/nginx/` | Nginx 설정 예시 |

## 배포

`main`에 HTML, CSS, 이미지 또는 배포 워크플로 변경을 푸시하면 GitHub Actions가 실행됩니다.
SCP로 `/usr/share/nginx/weekly-lab`에 파일을 전송하고, Nginx가 서비스합니다.
README와 문서만 변경하면 홈페이지 배포는 실행되지 않습니다.
DNS, 인증서와 Nginx 설정은 서버에서 별도로 관리합니다.

- [서버 설정과 배포 안내](deploy/README.md)
- [디자인 기준과 콘텐츠 작성 범위](docs/website.md)
