---
layout: post
title:  "audit에서 발생한 로그 중 HEX 해석 하기"
date:   2022-06-27
last_modified_at: 2022-06-27
categories: [linux]
tags: [linux]
---

audit 로그중 proctitle에서 확인된 값에 대해서 해석이 필요 하였다.

```sh
type=PROCTITLE msg=audit(1449583261.740:1899): proctitle=2F7573722F62696E2F7065726C002F7573722F73686172652F617773746174732F777777726F6F742F6367692D62696E2F617773746174732E706C002D757064617465002D636F6E6669673D68756C6B2E6C6F63616C002D636F6E6669676469723D2F6574632F61777374617473
```

간단히 python3을 통해 해석 방법은 다음과 같다.

```python

test.py

import binascii;

cvstring = binascii.a2b_hex('2F7573722F62696E2F7065726C002F7573722F73686172652F617773746174732F777777726F6F742F6367692D62696E2F617773746174732E7
06C002D757064617465002D636F6E6669673D68756C6B2E6C6F63616C002D636F6E6669676469723D2F6574632F61777374617473')

print(cvstring);

```

이후 cvstring에 protitle에기입된 ascii code를 string으로 변환 하여 확인한 값이다.

```sh

# python3 test.py
b'/usr/bin/perl\x00/usr/share/awstats/wwwroot/cgi-bin/awstats.pl\x00-update\x00-config=hulk.local\x00-configdir=/etc/awstats'

```

**[ref1] 사이트 ** 참고 하여 hex 값에 대한 해석을 진행

[ref1]:https://plautrba.fedorapeople.org/how-to-decode-hex-strings-in-audit-logs.html

