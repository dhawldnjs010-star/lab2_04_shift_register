# 실험 후 레포트: LAB2-04 시프트 레지스터

작성자: 상혁 (2025440084) / 작성일: `<기입 — 실제 제출일>` / 소스 커밋: `<기입 — evidence·reports 커밋 후 git log에서 확인한 해시>` / 제출 태그: `lab2-04-submit-v1` (evidence/reports를 커밋한 뒤 그 커밋에 `git tag lab2-04-submit-v1` 후 `git push origin lab2-04-submit-v1`로 생성) / GitHub 저장소: `https://github.com/dhawldnjs010-star/lab2_04_shift_register`

> 실험 후에 채운다. 수행하지 않은 항목은 "미수행"으로 표시하고, 구현 성공을 실물 동작 확인으로 대신하지 않는다. 사전 레포트: [pre](../pre/pre_report.md)

## 진행 경로

- [x] Vivado GUI  - [ ] 오픈소스 CLI (Icarus, Yosys, nextpnr, Project X-Ray, openFPGALoader 버전 기록)

## Vivado 프로젝트와 시뮬레이션

- 프로젝트 이름 `lab2_shift_register`, 부품 `xc7s75fgga484-1`, Design/Simulation/Constraints 소스 등록, Simulation top `tb_shift_register4`, Top module `lab2_shift_register` (실제 화면 기준으로 확인)

| 항목 | VS Code(Icarus) | Vivado(XSim) | 차이·해석 |
|---|---|---|---|
| PASS 로그 | checks=8 | `LAB2_PASS shift_register4 checks=8` (담당 조원 PC 콘솔 로그 기준) | Icarus·XSim 검사 수 일치 |
| 종료 시각 | 106 ns | 106 ns (담당 조원 PC 콘솔 로그 기준) | Icarus·XSim 종료 시각 일치 |
| 주요 파형 | 사전 레포트 표 | `evidence/post/vivado_console.txt`에 근거 | 파형 개형 동일 (PASS 로그·콘솔 로그로 확인) |

## 합성·구현 결과

조원별로 실험을 나눠 맡아, 이 실험은 담당 조원이 자신의 PC(경로 `C:/UOS_ECE_project2/...`)에서 합성·구현·bit 생성까지 진행했다. 담당 조원 PC의 TCL 콘솔 로그(`evidence/post/vivado_console.txt`)에 `launch_runs impl_1 -to_step write_bitstream`까지 확인되며, 이를 근거 자료로 첨부한다. DRC/methodology/timing/utilization 리포트 파일과 .bit 파일 자체는 담당 조원 PC에만 있고 작성자 로컬 PC에는 없어 세부 수치는 담당 조원 PC 기준으로만 확인 가능하다.

| 항목 | 값 | 해석 |
|---|---|---|
| WNS / WHS | 담당 조원 PC에서 확인 (작성자 로컬에는 리포트 없음) | `evidence/post/vivado_console.txt` 참고 |
| DRC | 담당 조원 PC에서 확인 (작성자 로컬에는 리포트 없음) | 위와 같음 |
| TIMING-18 등 남은 경고 | 담당 조원 PC에서 확인 (작성자 로컬에는 리포트 없음) | 위와 같음 |
| 자원 사용량 | 담당 조원 PC에서 확인 (작성자 로컬에는 리포트 없음) | 위와 같음 |

## bit 파일

- 경로: 담당 조원 PC(`C:/UOS_ECE_project2/...`)의 Vivado 프로젝트에 생성됨 — 작성자 로컬 PC에는 파일 자체가 없다.
- SHA-256: 담당 조원 PC 기준(작성자 로컬에서는 해시 확인 불가)
- 참고: `evidence/post/vivado_console.txt`(담당 조원 PC TCL 콘솔 발췌, `launch_runs impl_1 -to_step write_bitstream` 확인 가능)

## 실제 장치 기록과 관찰

- Hardware Manager 콘솔에서 `open_hw_target` → `program_hw_devices`가 실행되었다(담당 조원 PC에서 프로그래밍(조원별 실험 분담)). 콘솔 로그: `evidence/post/vivado_console.txt`.
- Program Device 화면, 보드 전체 사진, 조작 영상은 `evidence/post/`에 추가한다. 아래 표는 실험 중 이미 확인·통과된 결과를 기록한다. 시연 영상: [Google Drive 폴더](https://drive.google.com/drive/folders/1bcvEsSmA-Qr2RFuTtRCmkes4qBkjJrJG?hl=ko).

| 조작 | 예상 (사전 레포트) | 실제 관찰 | 비고 |
|---|---|---|---|
| K4 초기화 | LED=00 | LED=00 | 예상과 일치 |
| SW1=1, N8 한 번 | 08 | 08 | 예상과 일치 |
| SW1=0, N8 한 번 | 04 | 04 | 예상과 일치 |
| SW1=1, N8 한 번 | 0A | 0A | 예상과 일치 |
| SW1=0, N8 한 번 | 05 | 05 | 예상과 일치 |
| SW1=0, N8 네 번 | 02 → 01 → 00 → 00 | 02 → 01 → 00 → 00 | 예상과 일치 |

## 예상과 실제의 차이·문제 해결

차이 없음 — 시뮬레이션(Icarus/XSim)과 실물 보드 동작 모두 사전 레포트의 예상과 일치했다.

## 링크

- 소스 커밋 / 제출 태그: `<기입 — 커밋 해시> / lab2-04-submit-v1`
- 사전 레포트: [pre_report.md](../pre/pre_report.md)
- 실행 로그·파형·사진·영상: `evidence/`
- GitHub 검증 기록(날짜): `<기입>`
