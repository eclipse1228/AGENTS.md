# AI 역공학 도구 20선

01. 분석 코어

1. REA
AI 에이전트로 앱과 바이너리를 분석하는 통합 MCP
https://github.com/morluto/rea

2. Ghidra
무료 디스어셈블러와 디컴파일러
https://github.com/NationalSecurityAgency/ghidra

3. GhidraMCP
Claude Code, Codex 등에서 Ghidra 분석 기능 사용
https://github.com/LaurieWired/GhidraMCP

4. IDA Pro MCP
IDA Pro를 AI 에이전트와 연결
https://github.com/mrexodia/ida-pro-mcp

02. 모바일 앱 분석

5. jadx
Android APK와 DEX를 Java 코드 형태로 분석
https://github.com/skylot/jadx

6. Apktool
Android 앱 리소스와 코드를 분석하고 재구성
https://github.com/iBotPeaches/Apktool

7. Frida
실행 중인 앱의 함수와 동작을 추적
https://github.com/frida/frida

8. ILSpy
.NET 프로그램의 코드를 분석하는 디컴파일러
https://github.com/icsharpcode/ILSpy

03. 바이너리 분석

9. radare2
터미널 기반 바이너리 분석 프레임워크
https://github.com/radareorg/radare2

10. x64dbg
Windows 프로그램용 디버거
https://github.com/x64dbg/x64dbg

11. ImHex
바이너리 파일 구조를 분석하는 헥스 에디터
https://github.com/WerWolv/ImHex

12. angr
프로그램의 가능한 실행 경로를 분석하는 프레임워크
https://github.com/angr/angr

04. 네트워크 및 실행 분석

13. mitmproxy
HTTP와 HTTPS 트래픽 분석 및 디버깅
https://github.com/mitmproxy/mitmproxy

14. dnSpyEx
.NET 프로그램 디버깅 및 코드 분석
https://github.com/dnSpyEx/dnSpy

15. Rizin
오픈소스 역공학 프레임워크
https://github.com/rizinorg/rizin

16. RAD Debugger
Windows와 Linux용 네이티브 디버거
https://github.com/EpicGames/raddebugger

05. 자료 모음과 재구현

17. Awesome AI Reverse Engineering
AI 역공학 도구 100개 이상을 분류한 자료 모음
https://github.com/zhuyansen/awesome-ai-reverse-engineering

18. Awesome Reverse Engineering
REA 제작자 측에서 정리한 역공학 도구와 학습 자료
https://github.com/morluto/awesome-reverse-engineering

19. REA 분석 워크플로
프로그램의 동작을 관찰하고 증거를 수집해 기능과 구조를 문서화하는 방식
https://github.com/morluto/rea

20. 클린룸 재구현
원본 동작 관찰 → 기능 명세서 작성 → 독립적인 코드 구현 → 동작 비교 → 수정

핵심 흐름은 다음과 같습니다.

분석 대상 → REA → 기능 명세서 → 재구현 → 비교 검증
