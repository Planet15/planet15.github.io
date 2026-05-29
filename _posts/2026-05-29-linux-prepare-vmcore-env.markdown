---
layout: post
title: "prepare_vmcore_env.sh를 이용한 Oracle Linux vmcore 분석 환경 자동 준비"
date: 2026-05-29
last_modified_at: 2026-05-29
categories: [linux]
tags: [linux, crash, vmcore]
---

Linux kernel crash dump 를 분석하기 위해서는 `vmcore` 파일만으로는 부족하다.
분석 대상 kernel 과 정확히 일치하는 `vmlinux` debug symbol 파일이 필요하고,
source level 로 확인하려면 해당 kernel source tree 도 같이 준비 되어야 한다.

Oracle Linux 환경에서는 UEK/RHCK 여부와 kernel release 에 따라서 받아야 하는
debuginfo RPM, source SRPM 이름이 달라질 수 있다. 이 과정을 매번 수동으로
진행하면 시간이 많이 걸리고, 분석에 필요한 파일이 서로 맞지 않는 문제가 생길 수
있다.

이를 자동화 하기 위해 [prepare_vmcore_env.sh][prepare_vmcore_env] 스크립트를 사용할 수 있다.

## 스크립트 위치

스크립트는 다음 GitHub repository 에서 확인할 수 있다.

```sh
https://github.com/Planet15/tools/blob/main/prepare_vmcore_env.sh
```

서버에서 바로 내려받아 사용하려면 raw 파일을 다운로드 한다.

```sh
curl -fsSL -o prepare_vmcore_env.sh https://raw.githubusercontent.com/Planet15/tools/main/prepare_vmcore_env.sh
chmod +x prepare_vmcore_env.sh
```

## 동작 개요

`prepare_vmcore_env.sh` 는 기본적으로 `/var/crash` 아래에서 가장 최근의
`vmcore` 파일을 찾고, 분석에 필요한 파일을 `~/vmcore_analysis` 아래에
준비한다.

전체 흐름은 다음과 같다.

```sh
1. /var/crash 아래에서 최신 vmcore 검색
2. vmcore 를 작업 디렉토리로 복사
3. strings 명령으로 OSRELEASE 값 추출
4. UEK/RHCK kernel flavor 판단
5. oss.oracle.com 에서 debuginfo RPM 검색
6. debuginfo RPM 에서 vmlinux 추출
7. source SRPM 다운로드 후 rpmbuild -bp 로 source tree 준비
8. analysis.env 파일 생성
9. crash 명령 실행
```

## 기본 사용법

기본 사용법은 다음과 같다.

```sh
./prepare_vmcore_env.sh
```

별도 옵션 없이 실행하면 다음 기본값을 사용한다.

```sh
vmcore 검색 위치 : /var/crash
작업 디렉토리   : ~/vmcore_analysis
```

분석 준비만 하고 `crash` 를 바로 실행하지 않으려면 다음과 같이 실행한다.

```sh
./prepare_vmcore_env.sh --prepare-only
```

특정 `vmcore` 파일을 지정할 수도 있다.

```sh
./prepare_vmcore_env.sh --vmcore /var/crash/127.0.0.1-2026-05-29-10:00:00/vmcore
```

작업 디렉토리를 별도로 지정하는 경우는 다음과 같다.

```sh
./prepare_vmcore_env.sh \
    --vmcore /var/crash/test/vmcore \
    --workspace /data/vmcore_analysis \
    --prepare-only
```

## 필요한 명령

스크립트는 실행 초기에 필요한 명령이 있는지 확인한다.

```sh
strings
rpm2cpio
cpio
crash
find
sort
grep
sed
awk
rpmbuild
curl 또는 wget
```

`crash`, `rpm-build`, `cpio` 등이 설치되어 있지 않으면 먼저 패키지를 설치해야
한다.

```sh
dnf install -y crash rpm-build cpio
```

## OSRELEASE 기반 자동 판단

스크립트는 `vmcore` 안에서 `OSRELEASE` 값을 찾는다.

```sh
strings vmcore | grep -m 1 -E "OSRELEASE=[0-9]"
```

이 값으로 Oracle Linux major version 을 판단하여 다음 경로 중 하나를 선택한다.

```sh
ol5
ol6
ol7
ol8
ol9
ol10
```

또한 release 문자열에 `uek` 가 포함되어 있으면 UEK kernel 로 판단하고,
그렇지 않으면 RHCK 로 처리한다.

UEK 인 경우에는 다음과 같은 패키지 이름을 우선 검색한다.

```sh
kernel-uek-debuginfo-${OS_RELEASE}.rpm
kernel-uek-${SOURCE_RELEASE}.src.rpm
```

RHCK 인 경우에는 다음과 같은 debuginfo 패키지를 검색한다.

```sh
kernel-debuginfo-${OS_RELEASE}.rpm
kernel-debuginfo-common-${ARCH}-${OS_RELEASE}.rpm
```

## 준비되는 파일

기본 작업 디렉토리를 사용할 경우 주요 결과물은 다음 위치에 생성된다.

```sh
~/vmcore_analysis/vmcore
~/vmcore_analysis/downloads/${OS_RELEASE}/
~/vmcore_analysis/debug/${OS_RELEASE}/
~/vmcore_analysis/kernel-source/${OS_RELEASE}/
~/vmcore_analysis/analysis.env
```

`analysis.env` 에는 분석에 필요한 주요 경로가 저장된다.

```sh
OS_RELEASE=...
KERNEL_FLAVOR=...
VMCORE_PATH=...
VMLINUX_PATH=...
DEBUGINFO_DIR=...
SOURCE_RPM_PATH=...
KERNEL_SOURCE_DIR=...
```

이 파일을 남겨두면 이후에 같은 vmcore 를 다시 분석할 때 어떤 파일을 기준으로
준비했는지 확인하기 쉽다.

## crash 실행

기본 모드에서는 준비가 끝난 뒤 다음 형식으로 `crash` 를 실행한다.

```sh
crash "$VMLINUX_PATH" "$VMCORE_PATH"
```

`--prepare-only` 로 실행한 경우에는 실제 실행해야 할 명령만 출력하고 종료한다.
따라서 분석 환경을 먼저 준비하고, 이후 별도 세션에서 수동으로 `crash` 를 실행할
수 있다.

```sh
crash "/path/to/vmlinux" "/path/to/vmcore"
```

## 주의 사항

이 스크립트는 Oracle Linux 의 debuginfo/source RPM 을 `oss.oracle.com` 에서
찾는 것을 전제로 한다. 따라서 분석 대상 kernel 과 일치하는 RPM 이 해당 경로에
존재해야 한다.

또한 `vmcore` 에서 `OSRELEASE` 값을 찾을 수 없거나, kernel release 와 일치하는
debuginfo RPM 을 찾지 못하면 중단된다. 이 경우에는 `--vmcore`, `--search-root`,
`--workspace` 옵션을 명시적으로 지정하고, 필요한 RPM 이 실제 repository 에
존재하는지 먼저 확인하는 것이 좋다.

## 정리

`prepare_vmcore_env.sh` 는 vmcore 분석 전에 반복적으로 수행하던 작업을 자동화한다.
특히 다음 작업을 한 번에 처리할 수 있다는 점이 유용하다.

```sh
vmcore 선택
OSRELEASE 확인
UEK/RHCK 판단
debuginfo RPM 다운로드
vmlinux 추출
source SRPM 기반 source tree 준비
crash 실행
```

kernel crash 분석은 symbol 과 source 가 맞지 않으면 시작 단계부터 시간이 많이
소요된다. 이 스크립트를 사용하면 분석에 필요한 기본 환경을 일관된 위치에 준비할
수 있으므로, 실제 원인 분석에 더 빠르게 들어갈 수 있다.

[prepare_vmcore_env]: https://github.com/Planet15/tools/blob/main/prepare_vmcore_env.sh
