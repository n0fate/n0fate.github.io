---
layout: post
title: Solaris(with Sun’s SPARC HW) wtmpx Analyzer
date: 2015-05-22 14:47:22.000000000 +09:00
type: post
published: true
status: publish
categories:
- OS Artifacts
tags:
- Solaris
- SPARC
- SUN
- wtmpx
---

얼마 전 지인의 요청으로 Sun Sparc 장비(솔라리스)의 사용자 접속 로그를 분석할 일이 있었다. 리눅스/유닉스 시스템은 보통 현재 접속 중인 세션 정보는 utmp/utmpx에 저장하고 접속 누적 정보는 wtmp/wtmpx 에 저장하고 있어서 이 파일을 분석하면 된다. 솔라리스도 동일한 파일에 데이터를 기록하고 있다.

솔라리스는 ‘/var/adm/utmpx or wtmpx’에 기록하며, 헤더 정보는 <a href="https://java.net/projects/solaris/sources/on-src/content/usr/src/head/utmp.h?rev=13149" target="_blank">Solaris Development Portal</a>에서 확인할 수 있다.

```bash
    struct futmpx {
        char ut_user[32]; /* user login name */
        char ut_id[4]; /* inittab id */
        char ut_line[32]; /* device name (console, lnxx) */
        pid32_t ut_pid; /* process id */
        int16_t ut_type; /* type of entry */
        struct {
            int16_t e_termination; /* process termination status */
            int16_t e_exit; /* process exit status */
        } ut_exit; /* exit status of a process */
        struct timeval32 ut_tv; /* time entry was made */
        int32_t ut_session; /* session ID, user for windowing */
        int32_t pad[5]; /* reserved for future use */
        int16_t ut_syslen; /* significant length of ut_host */
        char ut_host[257]; /* remote host name */
    };
```
솔라리스의 wtmpx 로그 분석 스크립트는 용기백배(@yk100)님의 utmp parser를 기반으로 개발되었으며, 다음 경로에서 받을 수 있다.

<a href="https://github.com/n0fate/utmpxparser" target="_blank">Unix/Linux/OS X Login Session Parser by n0fate on Github</a>

도구 수행 결과는 다음과 같다.

```bash
    user    id session type   terminal  pid  start time(utc+0)   status  ip
    ...
    xxxxxx ftp 0 USER_PROCESS ftp2xxx9 2xxx9 20xx 03 xx 02:51:17.000 x.xx.x.xx
    xxxxxx ftp 0 DEAD_PROCESS ftp2xxx8 2xxx8 20xx 03 xx 03:00:00.000 x.xx.x.xx
    xxxxxx ftp 0 DEAD_PROCESS ftp2xxx8 2xxx8 20xx 03 xx 03:00:00.000 x.xx.x.xx
    xxxxxx ftp 0 DEAD_PROCESS ftp2xxx9 2xxx9 20xx 03 xx 03:00:00.000 x.xx.x.xx
               0 DOWN_TIME    system down 0  20xx 03 xx 03:12:59.000
    reboot ~   0 BOOT_TIME    system boot 0  20xx 03 xx 03:31:14.000
               0 RUN_LVL      run-level S 0  20xx 03 xx 03:31:30.000
```
분석에 도움이 되길 바란다 ;-)
