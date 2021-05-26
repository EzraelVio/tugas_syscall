#Kode yang sudah dirubah

##Makefile
```diff
-CS333_PROJECT ?= 1
+CS333_PROJECT ?= 2
PRINT_SYSCALLS ?= 0
CS333_CFLAGS ?= -DPDX_XV6
ifeq ($(CS333_CFLAGS), -DPDX_XV6)
CS333_UPROGS +=	_halt _uptime
endif

ifeq ($(CS333_PROJECT), 2)
CS333_CFLAGS += -DCS333_P1 -DUSE_BUILTINS -DCS333_P2
-CS333_UPROGS += _date _time _ps
+CS333_UPROGS += _date _ps _time
CS333_TPROGS += _testsetuid  _testuidgid _p2-test
endif
```

