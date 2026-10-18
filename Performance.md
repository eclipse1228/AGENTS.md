찾아보니, **AI에 적용할 종합 성능 최적화 스킬은 `addyosmani/web-quality-skills`를 가장 추천**합니다. **직접 읽으면서 개선 항목을 점검하려면 `Front-End-Performance-Checklist`의 한국어판**이 좋습니다.

GitHub의 README와 실제 `SKILL.md` 내용을 확인해, 질문하신 **“웹사이트 속도 개선 방법 총정리”에 잘 맞는 자료 6개**와 함께 사용할 측정 도구를 정리했습니다. 여기서 스킬은 AI 코딩 에이전트가 참고하는 작업 지침을 뜻합니다.

## 1. addyosmani/web-quality-skills — 종합 성능 최적화 스킬로 가장 추천

**GitHub:** [addyosmani/web-quality-skills](https://github.com/addyosmani/web-quality-skills)  
**성능 스킬 바로 보기:** [performance / SKILL.md](https://github.com/addyosmani/web-quality-skills/blob/main/skills/performance/SKILL.md)

특정 프레임워크에 제한되지 않는 웹 품질 개선 스킬 모음입니다. 속도 개선에 관련된 핵심 스킬은 다음과 같습니다. [GitHub](https://github.com/addyosmani/web-quality-skills?utm_source=chatgpt.com)

| 스킬 | 다루는 내용 |
|---|---|
| `performance` | 로딩 속도, JavaScript 실행, 이미지·폰트, 캐싱, 서버 응답 등 전반적인 성능 개선 |
| `core-web-vitals` | 주요 콘텐츠 표시 속도인 **LCP**, 입력 반응성인 **INP**, 화면 배치 안정성인 **CLS** 진단과 개선 |
| `web-quality-audit` | 성능을 포함해 접근성·SEO·웹 품질 전반을 점검 |

실제 성능 스킬에는 다음과 같은 개선 방법이 들어 있습니다.

- **서버·네트워크:** 서버 응답 지연 개선, CDN, Gzip/Brotli 압축, 캐시 설정.
- **JavaScript:** 코드 분할, 필요한 기능만 불러오기, 비필수 스크립트 로딩 지연.
- **이미지:** 이미지 형식과 크기 최적화, 반응형 이미지, 주요 이미지의 로딩 우선순위.
- **CSS·폰트:** 첫 화면을 그리는 데 필요한 리소스 우선 처리, 웹폰트 로딩 개선.
- **화면 동작:** 불필요한 레이아웃 계산 감소, 긴 목록 처리, 무거운 외부 위젯 로딩 개선. [GitHub](https://github.com/addyosmani/web-quality-skills/blob/main/skills/performance/SKILL.md?utm_source=chatgpt.com)

**추천하는 이유:** 개선 항목의 범위가 넓고, **측정 → 병목 진단 → 관련 코드 수정 → 같은 조건에서 재측정**하는 절차까지 안내합니다. Chrome DevTools MCP 연동은 선택 사항이며, 사용할 수 없으면 Lighthouse나 PageSpeed Insights 등의 대안을 활용하도록 구성되어 있습니다. [GitHub](https://github.com/addyosmani/web-quality-skills?utm_source=chatgpt.com)

README에 안내된 설치 명령은 다음과 같습니다.

```bash
npx skills add addyosmani/web-quality-skills
```

**질문하신 용도로 하나만 선택한다면 이 저장소부터 보시는 것을 권합니다.**

## 2. cloudflare/skills의 web-perf — 실제 사이트가 느린 원인을 찾는 데 추천

**GitHub:** [cloudflare/skills](https://github.com/cloudflare/skills)  
**해당 스킬 바로 보기:** [web-perf / SKILL.md](https://github.com/cloudflare/skills/blob/main/skills/web-perf/SKILL.md)

이 스킬은 **실제 페이지를 조사하고 병목을 찾아내는 절차**가 구체적입니다.

주요 점검 항목은 다음과 같습니다.

- 페이지 로딩 과정의 성능 기록 분석.
- 주요 콘텐츠가 늦게 나타나는 원인과 화면 배치가 움직이는 원인.
- 화면 표시를 막는 CSS·JavaScript.
- 앞선 요청이 끝나야 다음 요청이 시작되는 네트워크 지연.
- 캐시 헤더, 큰 파일, 미사용 코드, 압축 상태.

특히 어떤 파일이나 요청이 문제인지 확인하고, **예상 개선 효과를 근거로 우선순위를 정하도록 한다는 점**이 좋습니다. Chrome DevTools의 성능·네트워크 분석 도구가 연결된 환경에서 활용도가 높으며, 측정 도구가 없으면 가능한 소스 분석을 진행하고 측정하지 못한 내용을 구분하도록 합니다. [GitHub](https://github.com/cloudflare/skills/blob/main/skills/web-perf/SKILL.md?utm_source=chatgpt.com)

**추천 상황:** 이미 운영 중인 사이트 URL이 있고, “우리 사이트는 정확히 무엇 때문에 느린가?”를 AI에게 조사시키고 싶을 때.

## 3. vercel-labs/agent-skills의 react-best-practices — React·Next.js라면 추가 추천

**GitHub:** [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills)  
**해당 스킬 바로 보기:** [react-best-practices / SKILL.md](https://github.com/vercel-labs/agent-skills/blob/main/skills/react-best-practices/SKILL.md)

React와 Next.js의 성능 개선 규칙을 모아 놓은 Vercel의 스킬입니다. 다음과 같은 **코드 구조와 데이터 로딩 문제**를 점검하기 좋습니다.

- 독립적인 데이터 요청을 불필요하게 순서대로 실행하는 문제.
- 초기 JavaScript 번들이 커지는 문제.
- 서버에서 데이터를 가져오고 처리하는 방식.
- 클라이언트의 데이터 요청과 중복 작업.
- 불필요한 재렌더링과 비싼 렌더링 작업.

영향도에 따라 규칙의 우선순위가 정리되어 있고, 각 규칙에서 개선 전후 코드 예시를 확인할 수 있습니다. [GitHub](https://github.com/vercel-labs/agent-skills?utm_source=chatgpt.com)

**추천 상황:** React·Next.js 프로젝트의 소스코드를 직접 개선할 때. 이미지·폰트·네트워크까지 넓게 점검하는 **1번 스킬과 함께 사용하는 조합**을 권합니다.

## 4. Front-End-Performance-Checklist — 한국어로 읽는 종합 체크리스트

**원본:** [thedaviddias/Front-End-Performance-Checklist](https://github.com/thedaviddias/Front-End-Performance-Checklist)  
**한국어판:** [parksb/Front-End-Performance-Checklist](https://github.com/parksb/Front-End-Performance-Checklist)

**“사이트 속도를 높이기 위해 무엇을 확인해야 하는지 한 문서에서 읽고 싶다”**는 목적에 잘 맞습니다.

HTML, CSS, 폰트, 이미지, JavaScript 등으로 항목을 나누고, 각 항목에 **왜 필요한지와 어떻게 개선하는지**, 참고 도구와 문서를 함께 제공합니다. 한국어판에서도 이미지 크기와 반응형 이미지, 스크립트 로딩, 라이브러리 용량, CDN, 캐시 헤더, 압축 등을 살펴볼 수 있습니다. [GitHub](https://github.com/thedaviddias/Front-End-Performance-Checklist?utm_source=chatgpt.com)

다만 **일부 항목은 현재 프로젝트 조건에 맞춰 판단해야 합니다.** 예를 들어 CSS 파일을 하나로 합치는 방법은 HTTP 버전에 따라 효과가 달라지며, 원문도 HTTP/2에서는 검증이 필요하다고 설명합니다. 기본 원리를 익히고 점검 목록을 만드는 데 활용하면 좋습니다. [GitHub](https://github.com/thedaviddias/Front-End-Performance-Checklist?utm_source=chatgpt.com)

**추천 상황:** 한국어 자료가 필요하거나, 개발자가 직접 체크하면서 성능 개선 작업을 정리할 때.

## 5. Front-End-Checklist — 성능과 프론트엔드 품질을 함께 점검

**GitHub:** [thedaviddias/Front-End-Checklist](https://github.com/thedaviddias/Front-End-Checklist)  
**성능 항목 바로 보기:** [Performance 섹션](https://github.com/thedaviddias/Front-End-Checklist#performance)

성능 외에도 HTML·CSS·JavaScript·이미지·접근성·SEO 등을 함께 점검하는 저장소입니다. 현재 README는 사람이 직접 확인하는 체크리스트와 AI용 스킬·MCP 활용 방법을 모두 안내합니다. [GitHub](https://github.com/thedaviddias/Front-End-Checklist?utm_source=chatgpt.com)

성능 항목에서는 다음 내용을 확인할 수 있습니다.

- LCP·INP·CLS 개선.
- 초기 화면 이미지와 화면 밖 이미지의 로딩 전략.
- 렌더링을 막는 리소스와 긴 요청 연결 관계 개선.
- 캐싱·압축·CDN.
- 뒤로 가기·앞으로 가기 캐시 활용.
- 비필수 코드의 조건부 로딩과 HTML 스트리밍. [GitHub](https://github.com/thedaviddias/Front-End-Checklist?utm_source=chatgpt.com)

**추천 상황:** 출시 전 점검표를 만들거나, 성능 개선을 전체 프론트엔드 품질 관리 과정에 포함하고 싶을 때.

## 6. awesome-wpo — 관련 GitHub·도구·학습 자료를 넓게 찾는 자료집

**GitHub:** [davidsonfellipe/awesome-wpo](https://github.com/davidsonfellipe/awesome-wpo)

웹 성능 최적화에 관련된 도구·문서·책·강연 등을 분야별로 모아 놓은 자료집입니다.

성능 분석기, 번들 분석, CDN, 이미지 최적화, 지연 로딩, 코드 압축, 실제 사용자 성능 모니터링, 웹 서버 관련 자료까지 폭넓게 찾을 수 있습니다. Agent Skills를 모아 놓은 섹션도 있습니다. [GitHub](https://github.com/davidsonfellipe/awesome-wpo?utm_source=chatgpt.com)

**추천 상황:** “이미지 최적화 도구는 어떤 것이 있나?”, “번들 분석이나 성능 모니터링 관련 프로젝트를 더 보고 싶다”처럼 분야별로 탐색할 때.

오래된 자료도 함께 수록되어 있으므로, 여기서 도구를 찾은 뒤 해당 프로젝트의 현재 README를 확인하는 방식으로 활용하는 것이 좋습니다. 실제 목록에는 폐기된 문서라고 명시된 항목도 있습니다. [GitHub](https://github.com/davidsonfellipe/awesome-wpo?utm_source=chatgpt.com)

## 함께 사용하면 좋은 측정용 GitHub 3개

위 자료를 적용할 때는 **어디가 느린지, 수정 후 얼마나 달라졌는지**를 확인하는 도구를 함께 쓰는 것이 좋습니다.

| 저장소 | 역할 | 활용 시점 |
|---|---|---|
| [GoogleChrome/lighthouse](https://github.com/GoogleChrome/lighthouse) | 브라우저 테스트 환경에서 페이지를 검사하고 성능 지표와 개선 항목 제공. DevTools와 CLI로 사용 가능. | **개선 전 문제 파악, 개선 후 재측정.** [GitHub](https://github.com/GoogleChrome/lighthouse?utm_source=chatgpt.com) |
| [GoogleChrome/lighthouse-ci](https://github.com/GoogleChrome/lighthouse-ci) | 코드 변경마다 Lighthouse 검사를 반복하고 결과 비교·성능 예산 검사를 자동화. | **새 기능을 추가한 뒤 다시 느려지는 문제 감지.** [GitHub](https://github.com/GoogleChrome/lighthouse-ci?utm_source=chatgpt.com) |
| [GoogleChrome/web-vitals](https://github.com/GoogleChrome/web-vitals) | 실제 방문자의 LCP·INP·CLS 등을 수집하는 라이브러리. 원인 분석을 위한 추가 정보도 제공. | **운영 환경에서 실제 사용자 경험 확인.** 결과를 분석할 수집 시스템과 연결해야 함. [GitHub](https://github.com/GoogleChrome/web-vitals?utm_source=chatgpt.com) |

### 1번 스킬을 적용한 뒤 사용할 요청 예시

다음처럼 **대상·점검 범위·검증 방식**을 함께 지정하면 작업을 구체적으로 맡길 수 있습니다.

```text
performance와 core-web-vitals 스킬을 사용해서
[사이트 URL]과 현재 프로젝트 코드를 점검해 주세요.

먼저 모바일 환경의 성능을 측정하고,
이미지, 폰트, JavaScript 번들, CSS, 캐싱, 서버 응답,
외부 스크립트에서 병목을 찾아주세요.

실제 측정으로 확인한 문제와 코드만 보고 추정한 문제를 구분하고,
사용자에게 미치는 영향이 큰 항목부터 수정해 주세요.
수정 후에는 같은 조건에서 다시 측정해 전후 결과를 비교해 주세요.
```
