---
layout: post
title: "Repowatch를 이용한 Oracle Linux RPM Changelog 검색"
date: 2026-05-29
last_modified_at: 2026-05-29
categories: [linux]
tags: [linux, oraclelinux, rpm, changelog]
---

Oracle Linux 에서 특정 RPM package 의 changelog 를 확인해야 하는 경우가 있다.
특히 kernel package 는 버전이 많고, UEK/RHCK repo 가 나뉘어 있기 때문에 원하는
변경점을 찾기 위해 여러 버전의 changelog 를 직접 비교해야 하는 경우가 많다.

이 작업을 웹 화면에서 빠르게 확인하기 위해 다음 Repowatch 서비스를 사용할 수 있다.

```sh
http://adcore.iptime.org:8000
```

## 서비스 개요

해당 페이지는 `Repowatch - Changelog Search` 라는 이름의 검색 UI 를 제공한다.
화면 설명에는 `Oracle Linux RPM Changelog Viewer` 로 표시되어 있다.

기본적으로 다음 조건을 선택하거나 입력해서 RPM changelog 를 검색한다.

```sh
릴리스
Repo
패키지명
검색어
시작 버전
끝 버전
```

현재 화면에서 선택 가능한 Oracle Linux release 는 다음과 같다.

```sh
oraclelinux7.x86_64
oraclelinux8.x86_64
oraclelinux9.x86_64
oraclelinux10.x86_64
```

## 기본 화면

접속하면 기본 검색 조건은 Oracle Linux 9 와 `kernel-uek` 기준으로 표시된다.
검색어에는 `mm/` 가 입력되어 있어 kernel memory management 관련 changelog 를
확인하는 예제로 바로 사용할 수 있다.

화면은 크게 세 영역으로 구성되어 있다.

```sh
검색 조건
결과 내 검색
검색 결과 출력
```

`검색 조건` 영역에서 package, repo, version 범위를 정하고 `검색` 버튼을 누르면
결과가 출력된다. 출력된 결과 안에서 다시 찾고 싶은 문자열이 있으면 `결과 내 검색`
영역을 사용한다.

## 검색 조건

검색 조건의 의미는 다음과 같다.

```sh
릴리스      : Oracle Linux major version
Repo        : 대상 yum repository
패키지명    : changelog 를 확인할 RPM package 이름
검색어      : changelog line 안에서 찾을 문자열
시작 버전   : 검색 범위의 시작 package version
끝 버전     : 검색 범위의 끝 package version
```

예를 들어 Oracle Linux 9 UEK R8 kernel package 에서 memory management 관련
changelog 를 찾으려면 다음과 같이 선택한다.

```sh
릴리스      : oraclelinux9.x86_64
Repo        : ol9_UEKR8
패키지명    : kernel-uek
검색어      : mm/
```

버전 목록은 선택한 release, repo, package 기준으로 갱신된다. 따라서 package 를
변경한 뒤에는 `목록 새로고침` 버튼을 이용해 repo 와 version 목록을 다시 가져오는
것이 좋다.

## 검색 결과

검색 결과는 version 단위로 묶여 출력된다.

```sh
[6.12.0-202.76.1.el9uek]
- mm: ...
- mm/slab: ...
- mm/vmalloc: ...

[6.12.0-201.74.1.el9uek]
- mm/page_alloc: ...
```

각 항목에는 changelog line 이 그대로 표시되며, `Orabug`, `CVE` 번호가 포함된
경우 같이 확인할 수 있다. kernel update 분석이나 특정 bug fix 포함 여부를 확인할
때 유용하다.

검색 결과 상단에는 실제 검색에 사용된 조건과 Oracle yum repowatch 검색 URL 도
같이 표시된다.

```sh
릴리스
Repo
패키지명
시작 버전
끝 버전
검색어
검색 URL
```

## 결과 내 검색

검색 결과가 많은 경우에는 `결과 내 검색` 영역을 사용한다.

```sh
찾기
이전
초기화
```

검색어를 입력하고 Enter 를 누르면 결과 line 중 일치하는 항목으로 이동한다.
Shift + Enter 를 사용하면 이전 항목으로 이동할 수 있다.

이 기능은 changelog 검색 결과 안에서 다시 `CVE`, `Orabug`, subsystem 이름 등을
확인할 때 편리하다.

## 활용 예

다음과 같은 상황에서 사용할 수 있다.

```sh
특정 kernel version 사이에 포함된 bug fix 확인
CVE 번호가 어떤 package version 에 포함되었는지 확인
Orabug 번호 기준으로 수정 내역 확인
mm, xfs, nfs, ocfs2 같은 subsystem 변경점 검색
운영 서버 update 전 영향 범위 확인
```

예를 들어 kernel memory 관련 수정 내역을 확인할 때는 다음과 같이 검색할 수 있다.

```sh
package : kernel-uek
term    : mm/
```

특정 CVE 반영 여부를 확인하려면 검색어에 CVE 번호를 입력한다.

```sh
term : CVE-2025-38084
```

Oracle bug 번호를 알고 있다면 `Orabug` 번호로도 확인할 수 있다.

```sh
term : 38132168
```

## 주의 사항

이 서비스는 `http://adcore.iptime.org:8000` 주소로 제공되므로, 네트워크 상태나
서비스 실행 상태에 따라 접속이 되지 않을 수 있다.

또한 changelog 검색은 release, repo, package, version 범위가 정확해야 의미 있는
결과를 얻을 수 있다. package 이름을 변경한 뒤 repo/version 목록을 갱신하지 않으면
검색 조건이 맞지 않아 결과가 없거나 오류가 발생할 수 있다.

운영 서버에 적용할 update 를 판단할 때는 Repowatch 검색 결과를 1차 확인 자료로
사용하고, 실제 적용 전에는 Oracle Linux yum repository 와 package version 을
다시 확인하는 것이 좋다.

## 정리

Repowatch UI 를 사용하면 Oracle Linux RPM changelog 를 version 범위 기준으로
검색할 수 있다. 특히 kernel package 처럼 changelog 가 많은 RPM 에서 특정 subsystem,
CVE, Orabug 항목을 빠르게 찾을 때 유용하다.

```sh
서비스 주소 : http://adcore.iptime.org:8000
주요 용도   : Oracle Linux RPM changelog 검색
대표 예시   : kernel-uek package 의 mm/, CVE, Orabug 검색
```

수동으로 여러 package changelog 를 비교하는 것보다 검색 조건을 명확하게 지정할 수
있으므로, update 영향 분석이나 kernel bug fix 확인 작업을 더 빠르게 진행할 수 있다.
