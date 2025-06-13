## 🌈SSD Software Feature

1. Wear Leveling

2. Bad Block Management

3. Error Correction Code (ECC)

4. Over-Provisioning (OP)

5. Thermal Throttling

6. SLC Caching

7. Power Loss Protection (PLP)

## 1️⃣Wear Leveling

### ✅ Wear Leveling이란 

- 플래시 메모리에 Write/Erase등의 작업은 물리적 한계에 의해 일정 횟수 이상 반복할 수 없다.

- 왜냐하면 플래시 메모리의 특정 블록에 반복적으로 데이터가 기록되면 그 부분이 빠르게 마모되는 등의 이유로 저장장치 수명이 단축되고 오류가 발생할 수 있기 때문.

- 이에 따라 "P/E 주기"라고 불리는 Program/Erase 주기에 제한이 생긴다

> SSD 특성상 Write에는 반드시 Erase를 수반하기 때

- 따라서 모든 셀이 비슷한 횟수만큼 사용되도록 블록 사용을 분산시키는 알고리즘이 필요하다 -> **Wear Leveling**

### ✅ Wear Leveling의 종류

**1. Dynamic Wear Leveling**

- 쓰기 작업이 발생할 때마다 항상 새로운 블록 중에서 가장 적게 사용된 블록에 데이터를 저장하는 방식.

- 실제로 자주 사용하는 user data들을 대상으로 한다.

> OS, 응용소프트웨어처럼 잘 변하지 않는 데이터들은 대상 X

**2. Static Wear Leveling**

- 위 dynamic에 추가로, 거의 쓰이지 않지만 오랫동안 살아있는 유효 데이터까지도 이동시킴

- 예를 들어, 1000번 쓰인 블록과, 10번만 쓰인 블록이 있을 때, 10번만 쓰인 블록에 있는 데이터를 다른 곳으로 옮기고 해당 블록을 재활용

- 더 많은 데이터 이동과 copy 작업이 필요하지만, 전체 셀 소모를 더욱 균등하게 만듦

### ✅ Wear Leveling의 효과

- SSD 수명 증가: 평균적인 블록 소모를 균등화하여 특정 셀의 조기 사망을 막음

- GC 효율 향상: 유효 데이터가 덜 집중되므로 GC 시에도 복사할 데이터가 적어짐

- 데이터 신뢰성 확보: 특정 셀에 무리가 가지 않기 때문에 ECC 복구 확률도 줄어듦

## 2️⃣Bad Block Management

### ✅ Bad Block Management란 

- 플래시 메모리는 반도체 공정상 **태생적으로 결함이 있는 셀(=불량 블록)** 이 존재

- 또한 시간이 지남에 따라 SSD를 사용하면서도 불량 블록이 동적으로 발생

- 이런 불량 블럭이 사용되면 데이터 신뢰성에 문제가 발생

- 이들을 탐지하고 피하는 메커니즘 -> **Bad Block Management (BBM)**

### ✅ Bad Block Management의 종류

**1. Factory/Initial Bad Block (출하시 불량 블록)**

- NAND 제조 공정에서 처음부터 결함이 있는 셀들

- 제조사가 출하 전 **Bad Block Table (BBT)** 을 만들어서, 해당 블록에 접근하지 않도록 플래그를 지정해둠

> BBT는 Bad Block Table이라는 데이터 구조를 의미하며 모든 불량 블록의 위치와 상태가 저장되어 있다.
>
> 이 테이블은 DRAM 또는 NAND의 메타 영역에 보관되고, SSD 부팅 시 항상 로딩되어 FTL이 참고하게 된다.

- SSD 초기화 시 FTL이 이 정보를 참고하여 사용하지 않도록 차단

**1. Runtime Bad Block (운영 중 발생하는 블록)**

- 사용 중 반복적인 쓰기, 고온, 전력 불안정 등으로 인해 점점 블록이 망가짐

- 프로그램이나 읽기 동작에서 ECC 오류가 특정 수준 이상 발생하거나 복구 실패 시, 해당 블록을 불량으로 판단

### ✅ Bad Block 인지 어떻게 알지? 

- 읽기 실패 횟수 누적 : 특정 페이지에서 ECC가 반복적으로 발동되면 경고 신호로 인식

- Program/Erase 실패 : 블록에 쓰기 또는 삭제 명령 수행 시 실패하는 경우

- Verification Error : 쓰기 후 읽기 검증(Read-Back)에서 데이터 불일치가 발생한 경우

### ✅ Bad Block 회피 및 처리 방법

- Bad Block이 감지되면 해당 블록은 사용 금지 처리(BBT에 등록)하고, 내부적으로 ‘Bad’ 상태로 마킹

- Bad Block을 Spare Block으로 swap -> **Block Remapping**

> 운영 중에 Bad Block이 발생했을 때, 이 Bad Block에 있던 데이터를 다른 정상 블록에 옮기고 매핑 테이블을 수정해 해당 Spare Block을 대신 쓰게 만든다.
>
> 즉 FTL 매핑 테이블에서 기존 LBA → Bad Block 매핑을 LBA → Spare Block으로 수정

## 3️⃣ ECC (Error Correction Code)

### ✅ECC란

- 데이터를 저장할 때, 함께 오류 검출 및 정정을 위한 코드를 추가적으로 저장하고, 데이터를 읽을 때 오류가 발생했는지 감지하고 수정하는 기술

- 플래시 메모리는 물리적으로 전기 신호에 취약하고 P/E Cycle이 반복될수록 셀의 신뢰성이 떨어지기 때문에, ECC가 필수적으로 필요

### ✅ECC가 필요한 이유

- NAND 플래시는 에러가 쉽게 발생함

> 전기적 간섭, 누설 전류, 셀 간 간섭, 고온, 방전 등으로 인해 비트 반전 발생 가능
>
> 특히 TLC/QLC는 1셀에 더 많은 비트를 저장하기 때문에 더 많은 오류 가능성 존재

- ECC는 이런 오류를 복구 가능하게 만들어주는 기능

> 읽은 데이터에 오류가 있어도 ECC가 이를 감지하고 자동으로 복원해주면 사용자 입장에선 "정상 동작"처럼 느껴짐

### ✅ECC의 동작 원리

**1. 쓰기 시 (Encode)**

- 호스트가 SSD에 데이터(예: 512B 또는 4KB)를 쓰면

- ECC 인코더는 그 데이터에 대해 패리티 비트 또는 정정용 코드를 계산하여 함께 저장

- 이 때 **전체 데이터+ECC를 Page 단위(예: 16KB Page)** 에 저장함

**2. 읽기 시 (Decode)**

- 데이터를 읽으면 원래 데이터와 ECC 코드를 같이 읽음

- ECC 디코더가 오류 검출 알고리즘을 통해 에러 여부 판단

- 만약 복구 가능한 오류면 자동으로 수정하여 유저에게 반환

- 복구 불가능(Uncorrectable Error)이면 ECC 실패로 처리되고, 펌웨어는 다른 조치를 취함 (예: Spare Block 사용, 오류 로그 기록 등)
