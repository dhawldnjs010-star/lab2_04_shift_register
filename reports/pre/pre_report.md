# 실험 전 레포트: LAB2-04 시프트 레지스터

작성자: 상혁 (2025440084) / 작성일: 2026-09-20 / 소스 커밋: `e622e98` / workspace: `LAB1.code-workspace` (템플릿 v2.0.1) / OS: `Windows 11 Home 10.0.26200` / Python: `Python 3.14.7` / 시뮬레이터: Icarus Verilog `Icarus Verilog version 12.0 (devel) (s20150603-1539-g2693dd32b)`

> 이 레포트는 VS Code(Icarus) 시뮬레이션까지의 사전 검증이다. Vivado GUI와 실물 보드 결과는 실험 후 레포트([post](../post/post_report.md))에서 다룬다. 시각은 clk 상승 에지(5, 15, 25, … ns)의 1 ns 뒤, 즉 TB가 비교하는 시각으로 적었다.

## 목적과 예상 동작

직렬 입력을 bit 3으로 받아 bit 0 방향으로 이동시키는 4비트 시프트 레지스터를 설계한다. 입력 이력이 어떻게 값에 쌓이는지 계산한다.

### 포트 (`shift_register4.v`의 `shift_register4`)

| 포트 | 방향 | 비트 폭 | 설명 |
|---|---|---|---|
| clk | in | 1 | 상승 에지 기준 클록 |
| rst | in | 1 | 동기 active-high 리셋. enable보다 우선 |
| enable | in | 1 | 1이면 시프트, 0이면 유지 |
| serial_in | in | 1 | 새로 들어오는 비트(bit 3로 입력) |
| value | out (reg) | 4 | 현재 4비트 이력 |

최상위(`lab2_shift_register.v`, `lab2_shift_register`): `enable = press`, `serial_in = switches[7]`(SW1), `led = {4'b0000, value}`.

### 동작 규칙과 경계 입력

- 규칙: value(k+1) = rst ? 0000 : (enable ? {serial_in, value[3:1]} : value).
- 정상·경계: 입력 1, 0, 1, 0을 넣으면 1000, 0100, 1010, 0101. enable=0에서는 입력이 바뀌어도 유지. 0 입력 4번이면 전부 비워진다. rst는 최우선.
- 시간 기준: TB 클록은 10 ns 주기(상승 에지 5, 15, 25, … ns)이고 입력은 에지 1 ns 뒤에 바꾼다. 레지스터는 다음 상승 에지에서 갱신된다. 실제 보드 클록(1 kHz)은 시뮬레이션의 10 ns와 별개다.

## 소스와 테스트벤치

- 설계 top: `lab2_shift_register` / 시뮬레이션 top: `tb_shift_register4`
- 소스: [`src/shift_register4.v`](../../src/shift_register4.v), [`src/input_frontend.v`](../../src/input_frontend.v), [`src/lab2_shift_register.v`](../../src/lab2_shift_register.v)
- 테스트벤치: [`sim/tb_shift_register4.sv`](../../sim/tb_shift_register4.sv)
- 제약: [`constraints/lab2_shift_register.xdc`](../../constraints/lab2_shift_register.xdc)
- 설정: [`simulation.json`](../../simulation.json) (sources 3개, testbench `sim/tb_shift_register4.sv`, simulation_top `tb_shift_register4`)

| 파일 | 역할 |
|---|---|
| `src/shift_register4.v` | 핵심 동작을 담은 코어 `shift_register4`. TB가 이 모듈을 직접 검사한다. |
| `src/input_frontend.v` | 버튼·스위치를 클록에 맞추는 입력 회로. 리셋 2단 해제 동기화, 버튼·스위치 2단 동기화 플립플롭, STABLE_CYCLES=20(1 kHz에서 20 ms) 안정 확인 뒤 한 클록짜리 `press` 펄스를 만든다. |
| `src/lab2_shift_register.v` | 보드 top. 프런트엔드와 코어를 연결하고 LED로 출력한다. |
| `sim/tb_shift_register4.sv` | 입력 자극, 기대값 계산, 자동 비교(`check`), PASS/FAIL 출력, VCD 생성, watchdog. |
| `constraints/lab2_shift_register.xdc` | 핀 번호·전압과 1 kHz 클록 정의. Icarus는 XDC를 읽지 않으므로 이 사전 시뮬레이션에는 사용되지 않는다. |
| `simulation.json` | VS Code 시뮬레이션 작업이 읽는 소스 목록·테스트벤치·시뮬레이션 top. |

### 테스트벤치 동작

- 자극 순서: 리셋 → serial_in=1, 0 → enable=0(입력 1) 유지 → enable=1(입력 1) → 입력 0 → 0을 4클록 넣어 flush → serial_in=1, rst=1로 리셋 우선. 총 8회 검사.
- 검사 횟수: 8 (reset, input enters MSB, move toward LSB, hold, retain old stage values, four bit history, flush with four zeros, reset priority)
- 종료·watchdog: 마지막 검사 뒤 `finish` task가 `LAB2_PASS shift_register4 checks=N`을 출력하고 `$finish`한다. 별도로 100000 ns(100 µs) 뒤에 `watchdog timeout`으로 `$fatal` 처리한다. 예상 종료 시각은 106 ns이다.
- 이 TB는 코어 `shift_register4`만 시험한다. 입력 동기화·디바운스와 실제 핀·타이밍이 통과했다는 뜻은 아니다.

### XDC 설명

`lab2_shift_register.xdc`은 포트 이름을 `lab2_shift_register.v`과 맞춰 핀을 지정한다. 모든 I/O는 `LVCMOS33`이고 `create_clock -name trainer_1khz -period 1000000.000 [get_ports clk]`로 주 클록을 1 kHz(주기 1,000,000 ns)로 정의하며 `set_false_path -from [get_ports {rst button sw[*]}]`로 비동기 입력을 타이밍 경로에서 제외한다.

| 포트 | 핀 | 보드 대응 |
|---|---|---|
| clk | B6 | 1 kHz 주 클록 |
| rst | K4 | 리셋 (active-high) |
| button | N8 | 스텝 버튼 |
| sw[7:0] | U4(sw[0]), V4, W1, W4, T1, U2, W3, Y1(sw[7]) | DIPSW8..DIPSW1 (DIPSW1..8 = sw[7]..sw[0]) |
| led[7:0] | N5(led[0]), M1, M3, M7, N7, M2, M4, L4(led[7]) | LED0..LED7 |

## VS Code 실행 과정

1. File → New Window → File → Open Workspace from File...로 `LAB1.code-workspace`를 연다. 확장(slang, VaporView, vscode-pdf)을 설치한다.
2. RTL·TB·XDC·`simulation.json`을 직접 입력하고 File → Save All.
3. Terminal → Run Task... → `01 Check tools`로 Git·Python·iverilog·vvp 버전을 확인한다.
4. `02 Simulate`를 실행해 `LAB2_PASS`와 종료 시각을 확인한다.
5. `03 Open waveform`으로 `build/sim/wave.vcd`를 VaporView로 연다. 신호: clk, rst, enable, serial_in, value[3:0] (value는 2진수).

정상 실행 로그(본인 로그로 교체하고 `evidence/pre/`에 복사):

```text
LAB2_PASS shift_register4 checks=8
sim/tb_shift_register4.sv:19: $finish called at 106000 (1ps)
```

- 본인 실행 로그: [`../../evidence/pre/lab2_04_normal.log`](../../evidence/pre/lab2_04_normal.log)
- VCD: [`../../evidence/pre/lab2_04_wave_normal.vcd`](../../evidence/pre/lab2_04_wave_normal.vcd)
- 파형 캡처(VaporView): `evidence/pre/lab2_04_wave_zoom.png`(상승 에지 확대)
- 오류: 첫 실행에서 발생한 오류가 있으면 첫 오류 → 수정 → 재실행 로그 순서로 기록한다. (없으면 "없음", Python 실행 경로를 고쳤다면 그 내용 기입)

### 사전 파형 해석

| 시간 구간 | 입력 | 예상 | 실제 파형 | 해석 |
|---|---|---|---|---|
| 6 ns | rst=1 | value=0000 | value=0000 | 동기 리셋. |
| 16 ns | rst=0, enable=1, serial_in=1 · 에지 15 ns | value=1000 | value=1000 | 새 입력은 bit 3으로 들어온다. |
| 26 ns | serial_in=0 · 에지 25 ns | value=0100 | value=0100 | 기존 값이 bit 0 방향으로 한 칸 이동. |
| 36 ns | enable=0, serial_in=1 · 에지 35 ns | value=0100 | value=0100 | enable=0이므로 입력이 1이어도 유지. |
| 46 ns | enable=1, serial_in=1 · 에지 45 ns | value=1010 | value=1010 | 이전 단(0100)이 이동하면서 새 1이 bit 3으로. |
| 56 ns | serial_in=0 · 에지 55 ns | value=0101 | value=0101 | 입력 이력 1, 0, 1, 0이 1010→0101로 정리. |
| 96 ns | 0을 4클록 더 입력 · 에지 95 ns | value=0000 | value=0000 | 0을 4번 넣으면 이전 이력이 모두 밀려나 flush. |
| 106 ns | serial_in=1, rst=1 · 에지 105 ns | value=0000 | value=0000 | 리셋이 입력 1보다 우선. |

표의 값은 상승 에지 직후 안정된 값이다. `LAB2_PASS`만 적지 않고 각 행에서 입력, 이전 상태, 다음 상태를 비교한다. 파형 캡처에서 위 시각을 확대해 본인 화면으로 확인한다.

## 코드 수정·실패·복구 실험

- 변경: 시프트 방향을 뒤집는다(`{serial_in, value[3:1]}` → `{value[2:0], serial_in}`).
- 변경한 파일과 위치: `src/shift_register4.v` 9행
- 테스트벤치 기대값은 바꾸지 않는다.

```diff
-    {serial_in, value[3:1]}
+    {value[2:0], serial_in}
```

실행 전 계산: 첫 입력 1은 원래 bit 3으로 들어가 1000이 되어야 한다. 변경 회로는 bit 0으로 들어가 0001이 되고, TB의 `value===4'b1000` 검사가 16 ns에 실패한다.

| 단계 | 소스 커밋 또는 해시 | 실행 폴더·로그 링크 | 입력·기대값·실제값 | 해석 |
|---|---|---|---|---|
| 정상 코드 | `e622e98` | [normal.log](../../evidence/pre/lab2_04_normal.log) | 16 ns 기대 value=1000, 실제 value=1000. `LAB2_PASS shift_register4 checks=8` | 모든 검사 통과, 106 ns 종료. |
| 지정한 RTL 변경 | 미커밋 수정본(`e622e98` 기준, 로컬 실행) | [mod.log](../../evidence/pre/lab2_04_mod.log) | 16 ns 기대 value=1000, 실제 value=0001. `LAB2_FAIL input enters MSB time=16000`, `FATAL: sim/tb_shift_register4.sv:12: check failed` | `input enters MSB` 검사가 변경을 발견했다(로그의 time은 ps 단위, 16000 ps = 16 ns). |
| 원래 코드로 복구 | `e622e98` | [recover.log](../../evidence/pre/lab2_04_recover.log) | 복구 후 전체 검사 재실행. `LAB2_PASS shift_register4 checks=8`, `$finish called at 106000 (1ps)` | PASS와 종료 시각이 정상 실행과 같고 새 VCD를 확인한다. |

- 첫 실패 이후에는 `$fatal`로 시뮬레이션이 끝나므로 뒤의 검사는 실행되지 않는다. 변경 전후 파형은 각각 별도 폴더에 보관한다.
- 문법 오류를 경험했다면 오류 위치로 이동한 화면, 원인, 수정 내용과 재실행 로그도 이 절에 연결한다.

## 보드 실험 계획

- 부품·프로젝트: Vivado 2026.1, RTL Project `lab2_shift_register`, 부품 `xc7s75fgga484-1`(정확히 -1). Design Sources: `shift_register4.v`, `input_frontend.v`, `lab2_shift_register.v`(Copy sources 해제). Simulation Sources: `tb_shift_register4.sv`(Set as Top: `tb_shift_register4`). Constraints: `lab2_shift_register.xdc`. Project Summary의 Top module name은 `lab2_shift_register`이다.
- 장비 클록·제약: Combo II-DLD S75 주 클록 B6을 1 kHz로 맞추고 XDC의 `trainer_1khz`(1,000,000 ns)와 일치시킨다.
- 입력: DIPSW1=`sw[7]`=serial_in, N8=press(시프트 한 번), K4=리셋. SW1을 먼저 정한 뒤 N8을 누른다. 주 클록은 1 kHz.
- 출력: LED[3:0]=value, LED[7:4]=0.

| 조작 | 예상 LED / 동작 |
|---|---|
| K4 초기화 | LED=00 |
| SW1=1, N8 한 번 | 08 |
| SW1=0, N8 한 번 | 04 |
| SW1=1, N8 한 번 | 0A |
| SW1=0, N8 한 번 | 05 |
| SW1=0, N8 네 번 | 02 → 01 → 00 → 00 |

N8을 누르지 않고 SW1만 바꾸면 LED 값은 유지되어야 한다.

촬영할 장면: 보드 전체(배선·입력·출력이 함께 보이는 사진)와 위 표의 조작별 LED 상태 사진·영상. 예상되는 차이: 버튼·스위치는 동기화·안정 확인 지연이 있고, 빠른 신호는 눈으로 구분되지 않을 수 있다.

이 단계에서는 Vivado GUI와 실물 보드의 결과를 수행한 것처럼 기록하지 않는다. 합성·구현·bit 생성과 실제 장치 기록은 실험 후 레포트에서 다룬다.
