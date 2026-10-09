# 위클리랩 SEO 관리

## 대상과 대표 주소

브랜드명, 홈페이지 제작 외주, 업무 관리 시스템 개발과 업무 자동화 검색을 함께 다룬다.
대표 주소는 `https://weekly-lab.com/`이다. HTML의 canonical, 공유용 og:url, 구조화 데이터와 사이트맵을 이 주소로 맞춘다.
2026년 10월 9일 확인한 운영 서버는 `www`에서도 같은 HTML을 200으로 제공한다. canonical은 대표 주소 선택을 돕는 신호이며 리디렉션은 아니다.
저장소의 Nginx 예시는 www를 기본 주소로 301 이동시키지만 자동 배포는 Nginx 설정을 적용하지 않는다.

## 구현과 관리 도구

- 홈페이지 제목과 설명에 제공 서비스를 명시한다. 본문에 없는 실적, 가격, 지역이나 보장 문구는 추가하지 않는다.
- `Organization`과 `WebSite` 구조화 데이터에 공개된 상호, 로고, 주소와 이메일을 제공한다.
- `robots.txt`는 공개 콘텐츠 크롤링을 허용하고 사이트맵 위치를 안내한다.
- `sitemap.xml`에는 실제 대표 페이지 하나만 넣는다. 섹션 이동용 `#services` 같은 주소는 별도 페이지가 아니다.
- 새 페이지를 만들면 고유한 제목, 설명, canonical, 내부 링크와 사이트맵 항목을 함께 추가한다.
- Google Search Console은 사용자가 인증한 `weekly-lab.com` 도메인 속성을 사용한다. www와 HTTP/HTTPS를 함께 포함한다.
- 사이트맵 제출 주소는 `https://weekly-lab.com/sitemap.xml`이다. 제출, 읽기 성공, 페이지 색인 생성은 각각 별도로 확인한다.
- 네이버 서치어드바이저는 소유 확인과 사이트맵 제출 상태를 별도로 확인한다. Google 인증으로 네이버 인증을 대신하지 않는다.
- GA4 측정 ID는 `G-N7XY0LQX3V`다. 방문 분석과 검색엔진의 검색어/노출 분석은 구분한다.

## 매주 확인할 것

1. Search Console에서 색인 생성 여부, 크롤링 오류, Google이 선택한 대표 주소를 확인한다.
2. 검색어를 브랜드명, 홈페이지 제작, 업무 시스템/자동화로 나눠 노출, 클릭, CTR, 평균 게재순위를 본다.
3. 같은 기간의 GA4 자연 검색 유입과 실제 상담 문의를 함께 확인한다. 이메일 버튼 클릭은 메일 발송 완료가 아니다.
4. 노출은 있지만 클릭이 적은 검색어는 제목과 설명을 점검하고, 방문 후 문의가 적으면 사례와 상담 안내를 점검한다.
5. 실제 검색어와 상담 질문이 쌓이면 홈페이지 제작, 업무 시스템 개발, 자동화의 상세 안내를 각각 보강한다. 비슷한 문구를 복제한 페이지를 늘리지 않는다.

검색엔진의 색인과 순위는 설정 저장만으로 확정되지 않는다. 초기 보고서의 데이터 없음과 검색 결과 미노출도 구분한다.
현재 사이트는 이메일 상담 방식이므로 문의 완료를 자동 측정하지 않는다. 실제 상담은 수신 기록과 대조한다.

## 공식 참고

- [Google SEO 시작 가이드](https://developers.google.com/search/docs/fundamentals/seo-starter-guide)
- [Google 대표 주소 지정](https://developers.google.com/search/docs/crawling-indexing/consolidate-duplicate-urls)
- [Google 사이트맵 만들기와 제출](https://developers.google.com/search/docs/crawling-indexing/sitemaps/build-sitemap)
- [Google 회사 구조화 데이터](https://developers.google.com/search/docs/appearance/structured-data/organization)
- [네이버 robots.txt 안내](https://searchadvisor.naver.com/guide/seo-basic-robots)
