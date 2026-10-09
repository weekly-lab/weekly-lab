# 위클리랩 GEO 관리

GEO(Generative Engine Optimization)는 AI 검색에서 위클리랩이 발견되고 출처로 인용되는지 살피고, 고객이 판단할 수 있는 정보를 보강하는 작업이다.
기존 [SEO 관리](seo.md)를 기반으로 한다. 파일 추가, 구조화 데이터나 FAQ만으로 검색 노출과 추천을 보장하지 않는다.

## 콘텐츠와 llms.txt

- `index.html#faq`에서 준비 자료, 견적과 일정, 운영 비용, 기존 시스템 개선과 유지보수를 안내한다. JavaScript 없이 읽고 펼칠 수 있다.
- 사례는 기존 공개 자료에서 확인한 문제와 구현 기능을 설명한다. 홈페이지와 콘텐츠 관리 사례는 구축 진행 상태를 유지한다.
- `llms.txt`는 공개 서비스 요약과 본문 링크를 제공한다. 홈페이지의 `rel="describedby"` 링크로 찾을 수 있다.
- 파일은 [llms.txt 제안](https://llmstxt.org/)의 제목, 요약, 설명, 링크 목록 형식을 따른다. 현재 사이트는 정적 HTML이므로 별도 Markdown 본문 사본은 만들지 않는다.
- `llms.txt`를 사용하는 도구의 읽기를 돕는 용도다. Google은 이 파일을 검색 노출과 순위에 사용하지 않는다고 [안내한다](https://developers.google.com/search/docs/fundamentals/ai-optimization-guide).
- 제공 서비스, 연락처, 상담 조건이나 사례 상태를 바꾸면 HTML과 `llms.txt`를 함께 고친다. 비공개 자료, 미확인 성과 수치와 고정 가격을 추가하지 않는다.
- 사이트맵에는 대표 HTML 페이지를 유지한다. `#faq` 같은 섹션과 `llms.txt`를 별도 서비스 페이지로 등록하지 않는다.

`llms.txt` 변경도 자동 배포를 실행하며 SCP 전송 대상에 포함된다. `docs/`는 공개 웹 폴더로 전송하지 않는다.

## 검색 접근과 배포 점검

1. 대표 URL, `robots.txt`, `sitemap.xml`, `llms.txt`의 HTTPS 응답과 실제 본문을 확인한다. 없는 파일은 404여야 한다.
2. HTML의 `robots` 메타 태그와 응답의 `X-Robots-Tag`에서 `noindex`, `nosnippet` 또는 의도하지 않은 제한을 확인한다.
3. 검색 봇의 User-Agent로 요청해 차단 여부를 살핀다. 이 검사는 실제 봇의 IP, CDN/WAF 정책과 방문 로그를 검증하지 않는다.
4. Search Console에서 대표 URL의 색인 상태, 최근 크롤링, Google이 선택한 대표 주소와 사이트맵 읽기 결과를 각각 확인한다.
5. 생성 AI 기능의 포함 설정과 보고서가 계정에 표시되는지 확인한다. 보고서 미표시를 노출 0으로 기록하지 않는다.
6. 배포 후 HTML과 `llms.txt`가 같은 변경분인지 비교한다. 로컬 수정만으로 운영 반영을 완료 처리하지 않는다.

`robots.txt`의 `User-agent: *`와 `Allow: /`는 유지한다. 이미 허용된 봇을 위한 중복 그룹은 추가하지 않는다.
[OpenAI 공식 문서](https://developers.openai.com/api/docs/bots)에 따르면 `OAI-SearchBot`은 검색용이고 `GPTBot`은 학습용이다. 검색 노출과 학습 허용 정책은 별개이며 이번 작업은 기존 정책을 바꾸지 않는다.

### 2026-10-09 운영 확인

아래는 GEO 변경분 배포 전의 확인 결과다. 새 FAQ와 `llms.txt`의 운영 반영 결과가 아니다.

| 확인 대상 | 관측 결과 |
|---|---|
| 대표 홈페이지 | HTTPS 200, 공개 본문 정상 응답, `noindex`와 `X-Robots-Tag` 없음 |
| robots.txt / sitemap.xml | 모두 200, 저장소 내용과 일치 |
| 봇 User-Agent 요청 | Googlebot, OAI-SearchBot, PerplexityBot, bingbot 모두 홈페이지 200 |
| llms.txt | 운영 404. 이번 변경분 배포 후 200과 본문 일치 확인 필요 |
| HTTP 기본 주소 | HTTPS 기본 주소로 이동 후 200 |
| www 주소 | www에서 직접 200. 운영은 두 호스트를 같은 HTTPS server block에서 제공하며 HTTP만 HTTPS로 이동 |
| Google URL 검사 | `https://weekly-lab.com/` 색인 생성됨. 최근 크롤링 2026-10-09 16:49:14, Googlebot 스마트폰, 가져오기 성공, 크롤링/색인 허용 |
| Google 대표 주소 | 사용자 선언과 Google 선택 모두 `https://weekly-lab.com/` |
| Search Console 개요 | 실적과 색인 요약은 데이터 처리 중. URL 검사 결과와 구분 |
| Search Console 사이트맵 | 제출일 2026-10-09, 가져올 수 없음, 발견된 페이지 0. URL 검사에도 사이트맵 일시적인 처리 오류 표시 |
| 생성 AI 설정 / 보고서 | 설정 화면에 Google 검색 생성형 AI 포함으로 표시. 보고서는 현재 속성 메뉴에서 확인되지 않아 실제 AI 노출은 미확인 |
| Search Console robots 보고서 | robots.txt 파일 없음, 크롤링 통계 데이터 없음으로 표시. 공개 URL의 200 응답과 구분 |

사이트맵의 공개 HTTP 응답 성공과 Search Console의 읽기 성공은 다르다. 발견된 페이지 0을 홈페이지 미색인으로 해석하지 않는다.
후속 진단에서 Googlebot의 16:51:14와 16:58:58 사이트맵 요청이 모두 200인 것을 서버 로그로 확인했고, 요청 IP의 역방향/정방향 DNS도 대조했다.
17:16:46 Search Console 실시간 검사에서도 사이트맵 크롤링 허용과 가져오기 성공을 확인했다. 현재 접근 오류는 재현되지 않으며 Google 측 처리나 보고서 반영 지연은 가설이다. 최초 실패 원인은 확정하지 못했다.

운영 파일은 `/etc/nginx/conf.d/weekly-lab.com.conf`이며 `https://$host$request_uri`로 이동하므로 www를 유지한다.
저장소 Nginx 예시의 www 제거는 주소 통일을 위한 선택 사항이며 사이트맵 오류 해결의 필수 조건이 아니다.
변경을 선택하면 [서버 설정 절차](../deploy/README.md)에 따라 실제 파일을 대조한다. 정적 파일 배포로 서버 설정은 바뀌지 않는다.

## 성과를 나눠 기록하기

한 주 단위로 같은 기간을 비교한다. 배포 일자와 변경 내용을 함께 적고, 보고서별 시간대와 집계 단위를 남긴다.
검색 노출 증가와 상담 증가를 같은 성과로 취급하지 않는다.

| 구분 | 확인 방법 | 해석 범위 |
|---|---|---|
| Google AI 노출 | Search Console의 생성 AI 실적 보고서가 표시되면 기간별 노출 확인 | AI Overviews와 AI Mode의 링크 노출. 개별 답변의 추천 순위나 상담 수가 아님 |
| Bing AI 인용 | Bing Webmaster Tools의 AI Performance 사용 가능 여부를 확인한 뒤 URL별 인용 기록 | 지원하는 AI 서비스 범위의 인용. 전체 AI 검색 점유율이 아님 |
| AI 서비스 유입 | GA4 트래픽 획득에서 세션 소스/매체를 확인하고 실제 관측된 AI 서비스 출처를 기록 | 전달된 출처로 식별 가능한 방문만 포함 |
| 실제 상담 | 수신한 상담 메일의 유입 경로 선택 응답을 집계 | 자기신고 표본. 이메일 버튼 클릭과 메일 작성 창 열기는 상담 완료가 아님 |

GA4에서는 `chatgpt.com`, `perplexity.ai` 같은 소스를 먼저 찾아보고 실제 나타난 소스/매체 조합을 남긴다. 확인하지 않은 도메인 전체를 AI 유입으로 단정하지 않는다.
출처 정보가 없으면 `(direct) / (none)` 등으로 섞일 수 있으며, Google AI 유입을 이 목록만으로 분리할 수 없다.
신규 추적 스크립트는 추가하지 않는다. 기존 GA4와 메일 수신 기록을 사용한다.
상담 메일 템플릿에 `위클리랩을 알게 된 경로 (선택)`를 추가했다. 응답자의 이름, 연락처와 메일 본문은 이 문서에 기록하지 않는다.

### 주간 기록 양식

아래 행을 복사해 집계 수치만 기록한다. `미확인`, `데이터 처리 중`, `보고서 미표시`를 숫자 0으로 바꾸지 않는다.

| 기간 / 시간대 | 배포 또는 변경 | Google AI 노출 | Bing AI 인용 | 식별 가능한 AI 유입 | AI 경로 응답 상담 | 다음 확인 |
|---|---|---|---|---|---|---|
| 시작일~종료일 / 각 보고서 시간대 | 변경 없음 또는 배포 일자와 요약 | 값 또는 상태 | 값 또는 상태 | 소스/매체와 세션 수 | 수신 메일 기준 건수 | 확인할 항목 |

Google 생성 AI 보고서의 날짜는 PT 기준이다. GA4와 Bing은 해당 보고서 설정을 확인하고, 시간대가 다르면 일별 수치를 직접 비교하지 않는다.
초기 기준선은 위 운영 확인 결과로 남겼으며, GA4/Bing 수치와 실제 상담 건수는 이번 작업에서 조회하지 않았다.

### 답변 표본 확인

정식 노출 통계와 별도로, 같은 질문을 같은 서비스의 새 대화에서 주기적으로 확인할 수 있다.
예시 질문은 `위클리랩은 어떤 개발을 하나요?`, `홈페이지 제작을 의뢰할 때 어떤 자료를 준비해야 하나요?`, `엑셀로 관리하는 재고와 발주를 자동화하려면 무엇을 정리해야 하나요?`다.
서비스, 확인 시각과 시간대, 검색 사용 여부, 질문 원문, 브랜드 언급 여부, 인용 URL과 사실 오류를 기록한다.
질문 표본의 결과는 전체 노출 점유율이나 고정 순위가 아니다. 이번 작업에서는 AI 답변 표본을 수집하지 않았다.

## 공식 참고

- [Google 생성 AI 검색 최적화 안내](https://developers.google.com/search/docs/fundamentals/ai-optimization-guide)
- [Google 생성 AI 실적 보고서](https://support.google.com/webmasters/answer/16984139)
- [Bing AI Performance 안내](https://blogs.bing.com/webmaster/2026/2/Introducing-AI-Performance-in-Bing-Webmaster-Tools-Public-Preview/)
- [GA4 direct / none 해석](https://support.google.com/analytics/answer/15258820)
- [OpenAI 검색 및 학습 크롤러](https://developers.openai.com/api/docs/bots)
- [llms.txt 형식 제안](https://llmstxt.org/)
