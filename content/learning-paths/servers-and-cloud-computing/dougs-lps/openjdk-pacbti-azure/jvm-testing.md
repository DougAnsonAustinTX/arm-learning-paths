---
title: Testing the installed/optimized JVM for PAC/BTI enablement
weight: 5

### FIXED, DO NOT MODIFY
layout: learningpathall
---


### Create the testing script

Create this script and save it as $HOME/test-pacbti.sh:

```bash
#!/usr/bin/env bash
set -euo pipefail

# check-jvm-pac-bti.sh
#
# Usage:
#   ./check-jvm-pac-bti.sh --pid <java-pid>
#   ./check-jvm-pac-bti.sh --java /path/to/java
#
# Notes:
# - No readelf, no ELF note parsing.
# - Uses auxv hwcaps, JVM flags/cmdline, and disassembly.
# - Best on Linux/AArch64.
# - For a running JVM, PAC "active in JIT code" cannot be proven perfectly
#   from outside the process without deeper VM tooling; this script reports
#   "likely enabled" based on host support + requested JVM mode + binary code.

usage() {
  cat <<'EOF'
Usage:
  check-jvm-pac-bti.sh --pid <java-pid>
  check-jvm-pac-bti.sh --java <path-to-java>

Checks:
  1) Host PAC/BTI support via auxv hwcaps
  2) JVM support for -XX:UseBranchProtection
  3) Requested mode for a running JVM (from /proc/<pid>/cmdline)
  4) PAC/BTI instructions present in java/libjvm disassembly

Exit code:
  0 on success
  1 on usage or fatal detection error
EOF
}

die() {
  echo "ERROR: $*" >&2
  exit 1
}

need_cmd() {
  command -v "$1" >/dev/null 2>&1 || die "required command not found: $1"
}

PID=""
JAVA_BIN=""

while [ $# -gt 0 ]; do
  case "$1" in
    --pid)
      [ $# -ge 2 ] || die "--pid requires an argument"
      PID="$2"
      shift 2
      ;;
    --java)
      [ $# -ge 2 ] || die "--java requires an argument"
      JAVA_BIN="$2"
      shift 2
      ;;
    -h|--help)
      usage
      exit 0
      ;;
    *)
      die "unknown argument: $1"
      ;;
  esac
done

if [ -n "$PID" ] && [ -n "$JAVA_BIN" ]; then
  die "use either --pid or --java, not both"
fi

if [ -z "$PID" ] && [ -z "$JAVA_BIN" ]; then
  usage
  exit 1
fi

if [ -n "$PID" ]; then
  [ -d "/proc/$PID" ] || die "PID not found: $PID"
  JAVA_BIN="$(readlink -f "/proc/$PID/exe")"
  [ -x "$JAVA_BIN" ] || die "cannot resolve java executable from /proc/$PID/exe"
else
  JAVA_BIN="$(readlink -f "$JAVA_BIN")"
  [ -x "$JAVA_BIN" ] || die "not executable: $JAVA_BIN"
fi

need_cmd uname
need_cmd grep
need_cmd awk
need_cmd sed
need_cmd tr
need_cmd python3

OBJDUMP=""
if command -v objdump >/dev/null 2>&1; then
  OBJDUMP="$(command -v objdump)"
elif command -v llvm-objdump >/dev/null 2>&1; then
  OBJDUMP="$(command -v llvm-objdump)"
else
  die "need objdump or llvm-objdump"
fi

ARCH="$(uname -m)"
if [ "$ARCH" != "aarch64" ] && [ "$ARCH" != "arm64" ]; then
  die "this script is intended for Linux/AArch64; found architecture: $ARCH"
fi

if [ "$(uname -s)" != "Linux" ]; then
  die "this script is intended for Linux"
fi

# Resolve libjvm.so.
find_libjvm_from_pid() {
  local pid="$1"
  awk '/libjvm\.so/ {print $6; exit}' "/proc/$pid/maps" 2>/dev/null || true
}

find_libjvm_from_java() {
  local java_bin="$1"
  local java_real
  java_real="$(readlink -f "$java_bin")"
  local java_dir jhome c1 c2 c3
  java_dir="$(cd "$(dirname "$java_real")" && pwd)"
  jhome="$(cd "$java_dir/.." && pwd)"
  c1="$jhome/lib/server/libjvm.so"
  c2="$jhome/jre/lib/server/libjvm.so"
  c3="$jhome/lib/amd64/server/libjvm.so"
  for f in "$c1" "$c2" "$c3"; do
    if [ -f "$f" ]; then
      printf '%s\n' "$f"
      return 0
    fi
  done
  return 1
}

LIBJVM=""
if [ -n "$PID" ]; then
  LIBJVM="$(find_libjvm_from_pid "$PID")"
  if [ -z "$LIBJVM" ]; then
    LIBJVM="$(find_libjvm_from_java "$JAVA_BIN" || true)"
  fi
else
  LIBJVM="$(find_libjvm_from_java "$JAVA_BIN" || true)"
fi
[ -n "$LIBJVM" ] || die "could not locate libjvm.so"
[ -f "$LIBJVM" ] || die "libjvm.so not found: $LIBJVM"

# Host support via auxv. Linux kernel docs recommend auxv hwcaps for this.
AUXV_JSON="$(
python3 - <<'PY'
import ctypes, json

AT_HWCAP = 16
AT_HWCAP2 = 26

# arm64 hwcap bits in Linux UAPI
HWCAP_PACA = 1 << 30
HWCAP_PACG = 1 << 31
HWCAP2_BTI = 1 << 17

libc = ctypes.CDLL(None)
libc.getauxval.argtypes = [ctypes.c_ulong]
libc.getauxval.restype = ctypes.c_ulong

hwcap = libc.getauxval(AT_HWCAP)
hwcap2 = libc.getauxval(AT_HWCAP2)

out = {
    "hwcap": int(hwcap),
    "hwcap2": int(hwcap2),
    "paca": bool(hwcap & HWCAP_PACA),
    "pacg": bool(hwcap & HWCAP_PACG),
    "bti": bool(hwcap2 & HWCAP2_BTI),
}
print(json.dumps(out))
PY
)"

HOST_PACA="$(printf '%s\n' "$AUXV_JSON" | python3 -c 'import sys,json; print("yes" if json.load(sys.stdin)["paca"] else "no")')"
HOST_PACG="$(printf '%s\n' "$AUXV_JSON" | python3 -c 'import sys,json; print("yes" if json.load(sys.stdin)["pacg"] else "no")')"
HOST_BTI="$(printf '%s\n' "$AUXV_JSON" | python3 -c 'import sys,json; print("yes" if json.load(sys.stdin)["bti"] else "no")')"

if [ "$HOST_PACA" = "yes" ] || [ "$HOST_PACG" = "yes" ]; then
  HOST_PAC="yes"
else
  HOST_PAC="no"
fi

# Check JVM support for UseBranchProtection.
FLAG_LINE="$("$JAVA_BIN" -XX:+PrintFlagsFinal -version 2>&1 | grep -F 'UseBranchProtection' || true)"
if [ -n "$FLAG_LINE" ]; then
  JVM_HAS_FLAG="yes"
  JVM_DEFAULT_MODE="$(printf '%s\n' "$FLAG_LINE" | awk '{print $NF}')"
else
  JVM_HAS_FLAG="no"
  JVM_DEFAULT_MODE="unknown"
fi

# Requested mode from running process cmdline, if PID was supplied.
REQUESTED_MODE="unknown"
if [ -n "$PID" ]; then
  CMDLINE="$(tr '\000' ' ' < "/proc/$PID/cmdline" || true)"
  if printf '%s\n' "$CMDLINE" | grep -q -- '-XX:UseBranchProtection='; then
    REQUESTED_MODE="$(printf '%s\n' "$CMDLINE" | sed -n 's/.*-XX:UseBranchProtection=\([^ ]*\).*/\1/p' | head -n1)"
  else
    REQUESTED_MODE="not-set-on-cmdline"
  fi
fi

# Instruction scan helpers.
scan_binary() {
  # Echo "PAC=yes/no BTI=yes/no"
  local bin="$1"
  local dis
  if [ "$OBJDUMP" = "$(command -v llvm-objdump 2>/dev/null || true)" ] && [ -n "$OBJDUMP" ]; then
    dis="$("$OBJDUMP" -d "$bin" 2>/dev/null || true)"
  else
    dis="$("$OBJDUMP" -d "$bin" 2>/dev/null || true)"
  fi

  local pac="no"
  local bti="no"

  if printf '%s\n' "$dis" | grep -Eiq '\b(paciasp|autiasp|pacibsp|autibsp|retaa|retab)\b'; then
    pac="yes"
  fi

  if printf '%s\n' "$dis" | grep -Eiq '(^|[[:space:]])bti([[:space:]]|$)'; then
    bti="yes"
  fi

  printf 'PAC=%s BTI=%s\n' "$pac" "$bti"
}

JAVA_SCAN="$(scan_binary "$JAVA_BIN")"
LIBJVM_SCAN="$(scan_binary "$LIBJVM")"

JAVA_HAS_PAC="$(printf '%s\n' "$JAVA_SCAN" | sed -n 's/.*PAC=\([^ ]*\).*/\1/p')"
JAVA_HAS_BTI="$(printf '%s\n' "$JAVA_SCAN" | sed -n 's/.*BTI=\([^ ]*\).*/\1/p')"
LIBJVM_HAS_PAC="$(printf '%s\n' "$LIBJVM_SCAN" | sed -n 's/.*PAC=\([^ ]*\).*/\1/p')"
LIBJVM_HAS_BTI="$(printf '%s\n' "$LIBJVM_SCAN" | sed -n 's/.*BTI=\([^ ]*\).*/\1/p')"

# Heuristic overall verdicts.
PAC_REQUESTED="unknown"
if [ -n "$PID" ]; then
  case "$REQUESTED_MODE" in
    pac-ret|standard) PAC_REQUESTED="yes" ;;
    none|""|not-set-on-cmdline) PAC_REQUESTED="no" ;;
    *) PAC_REQUESTED="unknown" ;;
  esac
fi

PAC_LIKELY="no"
BTI_LIKELY="no"

# PAC likely enabled in a running JVM:
# - host PAC support present
# - JVM supports UseBranchProtection
# - process requested pac-ret or standard
# - libjvm/java contain PAC instructions
if [ -n "$PID" ]; then
  if [ "$HOST_PAC" = "yes" ] && \
     [ "$JVM_HAS_FLAG" = "yes" ] && \
     [ "$PAC_REQUESTED" = "yes" ] && \
     { [ "$LIBJVM_HAS_PAC" = "yes" ] || [ "$JAVA_HAS_PAC" = "yes" ]; }; then
    PAC_LIKELY="yes"
  fi
else
  # Without a PID, this is only a build/binary capability check.
  if [ "$HOST_PAC" = "yes" ] && \
     [ "$JVM_HAS_FLAG" = "yes" ] && \
     { [ "$LIBJVM_HAS_PAC" = "yes" ] || [ "$JAVA_HAS_PAC" = "yes" ]; }; then
    PAC_LIKELY="possible-but-not-proven"
  fi
fi

# BTI likely enabled for runtime binaries:
# - host BTI support present
# - binary contains BTI instructions
if [ "$HOST_BTI" = "yes" ] && \
   { [ "$LIBJVM_HAS_BTI" = "yes" ] || [ "$JAVA_HAS_BTI" = "yes" ]; }; then
  BTI_LIKELY="yes"
else
  if [ -z "$PID" ] && { [ "$LIBJVM_HAS_BTI" = "yes" ] || [ "$JAVA_HAS_BTI" = "yes" ]; }; then
    BTI_LIKELY="possible-but-not-proven"
  fi
fi

echo "== JVM PAC/BTI check =="
echo "java executable : $JAVA_BIN"
echo "libjvm          : $LIBJVM"
if [ -n "$PID" ]; then
  echo "pid             : $PID"
fi
echo
echo "-- Host support (auxv/hwcaps) --"
echo "PAC APIA/generic support : $HOST_PAC (PACA=$HOST_PACA PACG=$HOST_PACG)"
echo "BTI support              : $HOST_BTI"
echo
echo "-- JVM support/config --"
echo "UseBranchProtection flag : $JVM_HAS_FLAG"
echo "JVM default flag value   : $JVM_DEFAULT_MODE"
if [ -n "$PID" ]; then
  echo "Requested on cmdline     : $REQUESTED_MODE"
fi
echo
echo "-- Binary instruction scan --"
echo "java contains PAC instr  : $JAVA_HAS_PAC"
echo "java contains BTI instr  : $JAVA_HAS_BTI"
echo "libjvm contains PAC instr: $LIBJVM_HAS_PAC"
echo "libjvm contains BTI instr: $LIBJVM_HAS_BTI"
echo
echo "-- Verdict --"
echo "PAC status               : $PAC_LIKELY"
echo "BTI status               : $BTI_LIKELY"
echo
echo "Interpretation:"
if [ "$PAC_LIKELY" = "yes" ]; then
  echo "  PAC is likely enabled for this running JVM."
elif [ "$PAC_LIKELY" = "possible-but-not-proven" ]; then
  echo "  PAC-capable build detected, but no running JVM cmdline was checked."
else
  echo "  PAC not proven enabled."
fi

if [ "$BTI_LIKELY" = "yes" ]; then
  echo "  BTI is likely enabled in the runtime binaries."
elif [ "$BTI_LIKELY" = "possible-but-not-proven" ]; then
  echo "  BTI instructions are present in the binaries, but no running JVM was checked."
else
  echo "  BTI  not proven enabled."
fi
```

### Enable execution of the script

Add execution to the script $HOME/test-pacbti.sh:

```bash
chmod 755 $HOME/test-pacbti.sh
```

### Run the test script

Run the test script to confirm PAC/BTI enablement:

```bash
$HOME/test-pacbti.sh --java /usr/bin/java
```

Output should resemble:

```output
== JVM PAC/BTI check ==
java executable : /home/ubuntu/jdk/build/linux-aarch64-server-release/jdk/bin/java
libjvm          : /home/ubuntu/jdk/build/linux-aarch64-server-release/jdk/lib/server/libjvm.so

-- Host support (auxv/hwcaps) --
PAC APIA/generic support : yes (PACA=yes PACG=yes)
BTI support              : yes

-- JVM support/config --
UseBranchProtection flag : yes
JVM default flag value   : {default}

-- Binary instruction scan --
java contains PAC instr  : yes
java contains BTI instr  : yes
libjvm contains PAC instr: no
libjvm contains BTI instr: no

-- Verdict --
PAC status               : possible-but-not-proven
BTI status               : yes

Interpretation:
  PAC-capable build detected, but no running JVM cmdline was checked.
  BTI is likely enabled in the runtime binaries.
```

### What we have learned

By default, some OpenJDK JVMs are not published with PAC/BTI enabled.  This is primarily due to the number of older arm architectures running in the wild.  Over time, this will change though. If needed though, it's simple to create a private instance of a JVM that is built with PAC/BTI and registered with the underlying operating system.

Using JVMs that have the PAC/BTI features enabled, when run on platforms that support PAC/BTI, provide added safety and protection for Java-based workloads running in the JVM.
