---
title: WSL의 아키텍처 전환, 성능 개선, 현재 위치
description: WSL 1에서 WSL 2로의 전환이 실행 호환성과 파일시스템 성능을 어떻게 바꾸었는지, 현재 개발 상태를 공개 근거로 평가한다.
status: provisional
date: 2026-08-25
updated: 2026-08-25
tags:
  - wsl
  - windows
  - linux
  - virtualization
  - research
sources:
  - ../sources/microsoft-announcing-wsl-2-2019.md
  - ../sources/microsoft-comparing-wsl-versions.md
  - ../sources/microsoft-working-across-windows-linux-filesystems.md
  - ../sources/microsoft-wsl-open-source-2025.md
  - ../sources/github-microsoft-wsl-releases-2026-08-24.md
---
# WSL의 아키텍처 전환, 성능 개선, 현재 위치

## 연구 질문

- WSL 1의 설계는 어떤 장점과 구조적 한계를 가졌는가?
- WSL 2는 실행 호환성과 파일 I/O 성능을 어떻게 바꾸었는가?
- 파일시스템 문제는 어느 범위까지 해결되었는가?
- 공개 자료로 볼 때 WSL의 현재 우선순위를 어떻게 판단할 수 있는가?

## 잠정 결론

WSL의 역사는 “실패한 WSL 1을 WSL 2가 단순히 대체했다”기보다, **Windows NT 위에서 Linux를 번역하던 방식의 구조적 한계를 실제 Linux 커널을 포함하는 관리형 가상화 방식으로 우회한 과정**으로 보는 편이 정확하다. WSL 1은 Linux 시스템 호출을 Windows 커널 안에서 구현했으며, Microsoft도 시간이 지나면서 네이티브 Linux 호환성을 위해 Linux 커널 자체에 의존하는 것이 최선임을 확인했다고 설명한다. 따라서 WSL 1을 무조건 나쁜 접근으로 규정하기보다는, 통합성과 가벼움을 얻는 대신 호환성 확장 비용이 누적된 초기 설계로 평가할 수 있다. [WSL 오픈소스 전환 발표](../sources/microsoft-wsl-open-source-2025.md)

WSL 2는 실제 Linux 커널과 경량 관리형 VM을 채택해 시스템 호출 호환성을 크게 넓혔고, Microsoft의 초기 시험에서 파일 집약 작업은 압축 tar 해제 시 최대 20배, `git clone`·`npm install`·`cmake`에서는 약 2~5배 빨라졌다. 다만 이는 특정 초기 시험의 수치이며 모든 워크로드에 동일하게 적용되는 보편적 배율은 아니다. [WSL 2 발표](../sources/microsoft-announcing-wsl-2-2019.md)

파일시스템 문제는 **Linux 파일시스템 내부 워크로드에서는 크게 개선됐지만 Windows와 Linux 파일시스템 경계를 넘는 I/O까지 완전히 해결된 것은 아니다.** Microsoft는 Linux 도구로 작업할 때 프로젝트를 WSL 파일시스템에 두도록 권고하며, Windows 파일시스템에 파일을 둬야 하는 일부 경우에는 WSL 1이 더 빠를 수 있다고 명시한다. [WSL 버전 비교](../sources/microsoft-comparing-wsl-versions.md) [교차 파일시스템 지침](../sources/microsoft-working-across-windows-linux-filesystems.md)

“Microsoft 내부에서 Windows 11 개선이 더 시급해 WSL의 우선순위가 낮아졌다”는 평가는 공개 자료만으로 확인할 수 없다. 오히려 WSL은 Windows 출시 주기에서 분리된 패키지로 전환됐고 2025년 오픈소스화됐으며, 2026년 8월에도 안정 릴리스와 기능 프리릴리스가 이어졌다. 공개 증거가 지지하는 표현은 **저우선순위 프로젝트**보다 **핵심 아키텍처 전환을 마친 뒤 독립적인 배포·유지보수·기능 확장 단계에 들어간 성숙 프로젝트**에 가깝다. [WSL 오픈소스 전환 발표](../sources/microsoft-wsl-open-source-2025.md) [2026년 WSL 릴리스](../sources/github-microsoft-wsl-releases-2026-08-24.md)

## 요약 주장에 대한 판정

| 요약의 주장 | 판정 | 근거 기반 보정 |
| --- | --- | --- |
| WSL 1의 접근 방식은 좋지 않았다 | 부분 동의 | 번역 계층은 호환성 확장 비용이라는 구조적 한계가 있었지만, Windows와 Linux의 밀접한 통합과 낮은 오버헤드라는 목적에는 맞는 선택이었다. |
| WSL 2에서 실행속도가 비약적으로 발전했다 | 대체로 동의 | 특히 파일 집약 작업과 Linux 호환성에서 큰 도약이 확인된다. 다만 성능 배율은 워크로드와 파일 위치에 따라 달라진다. |
| WSL 2가 파일시스템 문제를 해결했다 | 범위 한정 동의 | Linux 쪽 파일시스템 내부 성능은 크게 개선됐다. 교차 OS 파일 접근은 여전히 예외이며 파일 배치가 중요하다. |
| 현재 WSL은 낮은 우선순위 프로젝트다 | 공개 근거로 확인 불가 | 최신 공개 릴리스와 오픈소스 전환은 지속적인 개발을 보여 준다. 내부 인력·예산 우선순위는 별도 내부 자료가 필요하다. |

## 1. WSL 1: 번역 계층이라는 초기 선택

WSL 1은 pico process provider와 `lxcore.sys`를 이용해 ELF 실행 파일을 Windows에서 직접 실행하고, Linux 시스템 호출을 Windows 커널 안에서 구현했다. 즉 Linux 커널을 구동하는 대신 Linux 프로그램의 기대를 NT 커널의 의미 체계로 번역하는 구조였다. [WSL 오픈소스 전환 발표](../sources/microsoft-wsl-open-source-2025.md)

이 설계의 장점은 별도 VM을 전면에 드러내지 않으면서 Windows 파일과 Linux 도구를 밀접하게 결합할 수 있다는 점이었다. 그러나 Linux 시스템 호출을 계속 재구현해야 하므로 커널 기능이 늘어날수록 호환성의 완전성을 유지하기 어렵다. Microsoft도 WSL 1의 번역 계층에서는 일부 애플리케이션을 실행할 수 없었고, Linux 커널 업데이트의 효용을 얻으려면 해당 변화를 다시 구현해야 했다고 설명한다. [WSL 2 발표](../sources/microsoft-announcing-wsl-2-2019.md)

따라서 “접근 방식이 좋지 않았다”는 평가는 결과를 보고 내린 표현이다. 더 정밀한 평가는 **초기 통합 목표에는 유효했지만 Linux 호환성을 장기적으로 확장하기에는 비용 구조가 불리했다**는 것이다.

## 2. WSL 2: 실제 Linux 커널과 경량 VM으로의 전환

WSL 2는 실제 Linux 커널을 경량 유틸리티 VM 안에서 실행한다. VM은 사용자에게 관리 대상으로 노출되기보다 WSL이 뒤에서 시작·종료·자원 관리를 담당한다. 이 전환은 WSL 1의 번역 계층을 계속 확장하는 대신, Linux의 시스템 호출 의미를 Linux 커널에 맡기는 선택이다. [WSL 2 발표](../sources/microsoft-announcing-wsl-2-2019.md) [WSL 버전 비교](../sources/microsoft-comparing-wsl-versions.md)

변화의 핵심은 두 가지다.

1. **호환성:** 실제 커널을 사용해 전체 시스템 호출 호환성을 제공하고 Docker와 FUSE처럼 WSL 1에서 제약이 컸던 소프트웨어 범위를 넓혔다. [WSL 2 발표](../sources/microsoft-announcing-wsl-2-2019.md)
2. **파일 I/O:** Microsoft의 초기 시험에서는 압축 tar 해제가 최대 20배, `git clone`·`npm install`·`cmake`가 약 2~5배 빨랐다. 이 수치는 WSL 2가 모든 명령을 일률적으로 가속했다는 뜻이 아니라, 파일 집약적 Linux 개발 워크로드의 병목이 크게 줄었다는 증거다. [WSL 2 발표](../sources/microsoft-announcing-wsl-2-2019.md)

즉 WSL 2의 도약은 단순 최적화가 아니라 **호환성 문제의 소유권을 Microsoft의 번역 코드에서 Linux 커널로 옮긴 아키텍처 교체**에서 나왔다.

## 3. 파일시스템 문제: 해결과 잔존 문제의 경계

“파일시스템 문제가 해결됐다”는 문장은 어떤 경로를 쓰는지 명시해야 한다.

| 작업 위치 | 평가 | 권장 방식 |
| --- | --- | --- |
| Linux 도구 + WSL 파일시스템 | WSL 2의 강점 | 프로젝트를 `/home/<user>/Project`처럼 Linux 쪽에 둔다. |
| Linux 도구 + Windows 마운트 | 교차 경계 비용이 남음 | 특별한 이유가 없으면 `/mnt/c/...`를 주 작업 경로로 쓰지 않는다. |
| Windows·Linux 도구가 같은 파일을 빈번히 접근 | WSL 2의 예외 구간 | 도구가 실행되는 OS와 파일 저장 OS를 맞추거나, 필요하면 WSL 1도 비교한다. |

Microsoft의 현재 문서는 WSL 2가 전반적으로 WSL 1보다 우수하다고 권고하면서도 “OS 간 파일시스템 성능”을 예외로 둔다. Linux 명령줄을 쓸 때는 WSL 파일시스템에, PowerShell이나 명령 프롬프트를 쓸 때는 Windows 파일시스템에 파일을 저장하라는 지침도 유지하고 있다. [WSL 버전 비교](../sources/microsoft-comparing-wsl-versions.md) [교차 파일시스템 지침](../sources/microsoft-working-across-windows-linux-filesystems.md)

그러므로 WSL 2가 해결한 것은 **Linux 개발 환경 내부의 파일 I/O 성능과 Linux 의미 체계에 맞는 실행 기반**이다. Windows와 Linux의 저장소 경계를 투명하고 무비용으로 만든 것은 아니다.

## 4. 현재의 WSL: 저우선순위인가, 성숙 단계인가

2021년 Microsoft는 WSL을 Windows 코드베이스와 분리해 별도 패키지로 옮겼고, 2022년에는 Windows 10도 지원하는 WSL 1.0.0을 정식 출시했다. Windows 11 24H2부터는 기본 내장 WSL에서 새 패키지로 사용자를 전환하기 시작했다. 이 변화는 WSL이 Windows 대형 업데이트를 기다리지 않고 자체 주기로 서비스될 수 있게 한 조직·배포 구조의 변화다. [WSL 오픈소스 전환 발표](../sources/microsoft-wsl-open-source-2025.md)

2025년에는 WSL 본체가 오픈소스화됐으며 Microsoft는 이를 커뮤니티가 수정과 기능 개발에 참여할 수 있는 활성 개발 프로젝트로 설명했다. [WSL 오픈소스 전환 발표](../sources/microsoft-wsl-open-source-2025.md)

2026년 8월 18일에는 안정 릴리스 2.7.12가, 8월 24일에는 프리릴리스 2.9.8이 공개됐다. 2.9.8 변경 목록에는 API 문서와 샘플, 네트워크 명령, 컨테이너 상태 검사, 메모리 회수 개선, 파일시스템·네트워크·CLI 수정 등이 포함됐다. 이는 최소한 공개 개발 활동이 중단되거나 보안 패치만 남은 상태가 아님을 보여 준다. [2026년 WSL 릴리스](../sources/github-microsoft-wsl-releases-2026-08-24.md)

다만 릴리스 활동만으로 Microsoft 내부의 인력, 예산, 경영 우선순위를 측정할 수는 없다. Windows 11과 비교한 상대적 우선순위를 주장하려면 팀 규모 변화, 로드맵, 예산 또는 담당자 발언 같은 내부·1차 자료가 추가로 필요하다. 현재 근거에서 가능한 결론은 다음과 같다.

> **WSL은 초기의 대규모 아키텍처 전환기를 지나 성숙 단계에 들어갔지만, Windows 본체에서 분리된 주기로 기능 개발과 유지보수가 계속되는 프로젝트다.**

## 실무적 함의

- 새 Linux 개발 환경은 특별한 예외가 없다면 WSL 2를 기본으로 선택한다. Microsoft도 WSL 2를 현재 기본 버전으로 둔다. [WSL 버전 비교](../sources/microsoft-comparing-wsl-versions.md)
- Node.js, Git, 컴파일, 패키지 설치처럼 작은 파일을 많이 다루는 작업은 소스 트리를 WSL 파일시스템에 둔다. [교차 파일시스템 지침](../sources/microsoft-working-across-windows-linux-filesystems.md)
- Windows 도구와 Linux 도구가 같은 트리를 빈번히 읽고 쓰는 워크플로는 실제 경로 배치로 벤치마크한다. Windows 파일시스템 고정이 필수라면 WSL 1이 더 나을 수 있다는 공식 예외도 남아 있다. [WSL 버전 비교](../sources/microsoft-comparing-wsl-versions.md)
- WSL의 제품 상태를 평가할 때 Windows 기능 업데이트 빈도만 보지 말고 Microsoft Store 패키지와 공식 GitHub 릴리스 흐름을 함께 본다. [WSL 오픈소스 전환 발표](../sources/microsoft-wsl-open-source-2025.md) [2026년 WSL 릴리스](../sources/github-microsoft-wsl-releases-2026-08-24.md)

## 한계와 추가 확인 사항

- 성능 수치는 Microsoft의 2019년 초기 시험이므로 최신 하드웨어와 최신 WSL 버전에서 재현 벤치마크가 필요하다. [WSL 2 발표](../sources/microsoft-announcing-wsl-2-2019.md)
- “Windows 11이 더 시급해 WSL 우선순위가 낮아졌다”는 내부 판단은 공개 자료로 검증하지 못했다.
- 현재 문서는 공식 Microsoft 자료와 공식 GitHub 릴리스에 집중했다. 독립 벤치마크를 추가하면 파일 위치·워크로드별 성능 차이를 더 정밀하게 평가할 수 있다.

## 출처

- [Microsoft, Announcing WSL 2 (2019)](../sources/microsoft-announcing-wsl-2-2019.md)
- [Microsoft Learn, Comparing WSL Versions](../sources/microsoft-comparing-wsl-versions.md)
- [Microsoft Learn, Working across Windows and Linux file systems](../sources/microsoft-working-across-windows-linux-filesystems.md)
- [Microsoft, The Windows Subsystem for Linux is now open source (2025)](../sources/microsoft-wsl-open-source-2025.md)
- [microsoft/WSL Releases snapshot (2026-08-24)](../sources/github-microsoft-wsl-releases-2026-08-24.md)