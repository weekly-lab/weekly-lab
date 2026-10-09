# 위클리랩 배포

위클리스쿨과 같은 인스턴스에서 GitHub Actions의 SCP 전송과 호스트 Nginx를 사용합니다.
HTML/CSS 정적 사이트이므로 Node.js 빌드, Docker 컨테이너, API 프록시는 필요하지 않습니다.

| 항목 | 위클리스쿨 | 위클리랩 |
|---|---|---|
| 도메인 | weekly-school.site | weekly-lab.com |
| 정적 파일 경로 | /usr/share/nginx/html | /usr/share/nginx/weekly-lab |
| Nginx 설정 | weekly-school.site.conf | weekly-lab.conf |
| API | /trpc, localhost:4000 | 없음 |

`www.weekly-lab.com`은 경로와 쿼리를 유지하며 `https://weekly-lab.com`으로 301 이동합니다.
두 도메인을 모두 포함한 별도 TLS 인증서를 사용합니다.
위클리스쿨 설정, 인증서와 정적 파일 경로는 변경하지 않습니다.

## 1. DNS와 서버 확인

DNS 관리 화면에 아래 레코드를 등록합니다.

| 타입 | 이름 | 값 |
|---|---|---|
| A | @ | 위클리스쿨 인스턴스의 고정 공인 IPv4 |
| CNAME | www | weekly-lab.com |

`172.26.9.132`는 사설 주소이므로 A 레코드에 넣지 않습니다.
아래 설정은 IPv4 기준입니다. IPv6를 별도로 구성하지 않았다면 두 이름에 잘못된 AAAA 레코드가 없어야 합니다.
보안 그룹과 서버 방화벽에서 외부 TCP 80/443 접근, 배포 실행기의 SSH 접근이 가능해야 합니다.

서버에서 전체 Nginx 설정의 중복 도메인, include 경로, 기존 Certbot 상태를 확인합니다.
출력에 내부 정보가 포함될 수 있으므로 공개 저장소에 저장하지 않습니다.

```sh
sudo nginx -T
sudo certbot certificates
sudo nginx -t
```

`weekly-lab.com` 인증서나 같은 이름의 server block이 이미 있으면 먼저 기존 설정을 대조합니다.
처음 설정하기 전 `/etc/nginx`를 별도 위치에 백업합니다.

## 2. 배포 폴더와 GitHub Secrets

서버의 배포 계정이 `ec2-user`인 경우입니다. 다른 계정이면 아래 계정명과 그룹명을 바꿉니다.

```sh
sudo install -d -m 755 -o ec2-user -g ec2-user /usr/share/nginx/weekly-lab
```

이 저장소의 Settings > Secrets and variables > Actions에 다음 Repository secrets를 등록합니다.
다른 저장소에 등록된 secrets는 자동으로 공유되지 않습니다.

| Secret | 값 |
|---|---|
| SSH_HOST | 인스턴스 공인 IP 또는 SSH 접속 도메인 |
| SSH_USER | ec2-user 또는 실제 배포 계정 |
| SSH_KEY | 해당 계정에 접속 가능한 SSH 개인키 |
| SSH_FINGERPRINT | 서버 SSH 호스트 공개키의 SHA256 지문 |

신뢰할 수 있는 기존 서버 접속이나 콘솔에서 호스트 키 지문을 확인합니다.
아래는 서버가 기본 ECDSA 호스트 키를 제공하는 경우입니다.

```sh
sudo ssh-keygen -lf /etc/ssh/ssh_host_ecdsa_key.pub -E sha256
```

출력의 `SHA256:...` 값을 등록합니다. 이 Action은 ECDSA를 RSA/Ed25519보다 먼저 협상할 수 있습니다.
다른 키 구성이면 `sudo sshd -T`의 `hostkey`와 `hostkeyalgorithms`를 확인하고,
실제로 협상되는 키의 공개키 파일을 `ssh-keygen -lf <공개키 경로> -E sha256`으로 검증합니다.
지문 불일치가 나면 서버 키와 협상 결과를 대조하며, 지문 검증을 끄거나 검증되지 않은 값을 등록하지 않습니다.
`WEB_DEPLOY_PATH`는 사용하지 않습니다. 잘못된 secret으로 위클리스쿨 파일을 덮어쓰지 않도록 경로를 워크플로에 고정했습니다.

## 3. HTTP 설정과 첫 파일 배포

이 저장소의 `deploy/nginx/` 파일 두 개를 서버의 작업 폴더로 복사합니다.
아래 `cp` 명령은 그 폴더에서 실행합니다.
처음에는 인증서가 필요 없는 HTTP 설정만 설치합니다.

```sh
sudo cp weekly-lab.http.conf /etc/nginx/conf.d/weekly-lab.conf
sudo nginx -t && sudo systemctl reload nginx
```

두 설정을 동시에 `conf.d/*.conf`에 넣지 않습니다. 이후에도 동일한 파일 하나만 교체합니다.
검사 실패 시 reload하지 말고 새 설정을 수정하거나 백업을 복구합니다.

서버 폴더와 secrets 준비 후 저장소의 `main`에 홈페이지와 워크플로를 올립니다.
GitHub Actions의 `CD - Deploy` 실행 결과를 확인합니다. 수동 실행도 `main`에서 가능합니다.
`index.html`, `styles.css`, `robots.txt`, `sitemap.xml`, `llms.txt`, `assets/`를 전송하며 README, Nginx 설정, `.git`은 공개 폴더에 전송하지 않습니다.

```sh
curl -I http://weekly-lab.com/
curl -I http://www.weekly-lab.com/
```

DNS 전파가 끝나고 두 주소에서 홈페이지가 응답하는지 확인합니다.
403이면 파일 읽기 권한, 상위 폴더 접근 권한, SELinux 적용 여부와 Nginx 오류 로그를 확인합니다.

## 4. 인증서와 HTTPS

이미 설치된 Certbot을 사용합니다. webroot 방식은 위클리스쿨의 Nginx 설정을 자동 편집하지 않습니다.
서버에서 두 도메인을 묶어 인증서를 발급합니다.

```sh
sudo certbot certonly --webroot \
  -w /usr/share/nginx/weekly-lab \
  --cert-name weekly-lab.com \
  -d weekly-lab.com -d www.weekly-lab.com \
  --deploy-hook 'nginx -t && systemctl reload nginx'
```

발급 성공 후 서버 작업 폴더의 HTTPS 설정으로 교체합니다.
설정은 위클리스쿨이 사용하는 기존 `options-ssl-nginx.conf`, `ssl-dhparams.pem` 파일을 참조합니다.

```sh
sudo cp weekly-lab.conf /etc/nginx/conf.d/weekly-lab.conf
sudo nginx -t && sudo systemctl reload nginx
sudo certbot renew --cert-name weekly-lab.com --dry-run
```

기존 Certbot timer 또는 cron이 활성화되어 있는지 확인합니다.
`systemctl list-timers --all`과 실제 Certbot 설치 방식의 cron 설정을 대조합니다.
예약 실행이 없다면 해당 설치 방식에 맞춰 등록해야 합니다. `--dry-run` 성공만으로 자동 실행을 보장하지 않습니다.
발급 때 지정한 deploy hook은 갱신된 인증서를 Nginx가 다시 읽게 합니다.
일반 `--dry-run`은 deploy hook을 실행하지 않으므로 위의 명시적 Nginx 검사와 reload 결과도 확인합니다.

## 5. 공개 확인과 이후 배포

```sh
curl -I https://weekly-lab.com/
curl -I https://weekly-lab.com/styles.css
curl -I https://weekly-lab.com/llms.txt
curl -I 'https://www.weekly-lab.com/?from=www'
curl -I http://weekly-lab.com/
curl -I http://www.weekly-lab.com/
curl -I https://weekly-lab.com/not-found
curl -I https://weekly-school.site/
```

기본 주소는 200, www와 HTTP는 기본 HTTPS 주소로 301, 없는 파일은 404여야 합니다.
`llms.txt`는 200과 `Content-Type: text/plain; charset=utf-8`로 제공하고, 본문이 저장소의 UTF-8 파일과 같은지 확인합니다.
파일이 UTF-8이어도 문자셋 헤더를 생략하면 브라우저가 다른 인코딩으로 읽을 수 있습니다. Nginx 예시의 `location = /llms.txt`에 `charset utf-8`을 지정합니다.
`llms.txt`는 UTF-8 BOM을 포함해 파일 자체에서도 인코딩을 식별합니다. 문자셋 헤더만 바꾸면 이전 ETag의 304 응답이 캐시된 문자셋을 유지할 수 있으므로, 수정 전 ETag로 요청해도 새 파일이 200으로 오는지와 브라우저의 `document.characterSet`이 `UTF-8`인지 함께 확인합니다.
www 요청의 `Location`에는 `?from=www`가 유지되어야 합니다.
HTML/CSS/이미지는 `Cache-Control: no-cache`로 변경 여부를 재검증합니다.
고정 파일명에 위클리스쿨의 `1y, immutable` 정책을 복사하지 않습니다.
위클리스쿨의 화면과 기존 `/trpc` 요청도 정상인지 확인합니다.

이후 `main`의 홈페이지 파일 변경은 SCP로 반영됩니다. 정적 파일 변경에는 Nginx reload가 필요 없습니다.
Nginx 설정 변경은 자동 적용하지 않으며, 서버에서 검사 후 별도 적용합니다.
파일을 같은 경로에 덮어쓰므로 배포 중 잠깐 이전 파일과 새 파일이 섞일 수 있습니다.
삭제한 파일은 서버에 남습니다. 제거가 필요하면 위클리랩 폴더의 해당 파일만 확인 후 삭제합니다.
콘텐츠 복구는 정상 버전으로 되돌리는 커밋을 `main`에 배포합니다. 실패한 전송도 다시 실행해 전체 파일을 복구합니다.

## 참고

- [Nginx 도메인별 요청 처리](https://nginx.org/en/docs/http/request_processing.html)
- [Nginx 캐시 헤더](https://nginx.org/en/docs/http/ngx_http_headers_module.html)
- [Certbot webroot와 갱신](https://eff-certbot.readthedocs.io/en/stable/using.html#webroot)
- [사용 중인 SCP Action](https://github.com/appleboy/scp-action/tree/v0.1.7)
