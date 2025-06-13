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

- 플래시 메모리에 Write/Read/Erase등의 작업은 물리적 한계에 의해 일정 횟수 이상 반복할 수 없다.

> 플래시 메모리의 특정 블록에 반복적으로 데이터가 기록되면 그 부분이 빠르게 마모되는 등의 이유로 저장장치 수명이 단축되고 오류가 발생할 수 있다

- 따라서 모든 셀이 비슷한 횟수만큼 사용되도록 블록 사용을 분산시키는 알고리즘이 필요하다 -> **Wear Leveling**

### ✅ Wear Leveling의 종류

**1. Dynamic Wear Leveling**

- 쓰기 작업이 발생할 때마다 항상 새로운 블록 중에서 가장 적게 사용된 블록에 데이터를 저장하는 방식.

- 즉, 현재 쓰기 작업이 발생하는 "유효 데이터"만 대상으로 함

**2. Static Wear Leveling**

- 위 dynamic에 추가로, 거의 쓰이지 않지만 오랫동안 살아있는 유효 데이터까지도 이동시킴

- 예를 들어, 1000번 쓰인 블록과, 10번만 쓰인 블록이 있을 때, 10번만 쓰인 블록에 있는 데이터를 다른 곳으로 옮기고 해당 블록을 재활용

- 더 많은 데이터 이동과 copy 작업이 필요하지만, 전체 셀 소모를 더욱 균등하게 만듦

### ✅ Wear Leveling의 효과

- SSD 수명 증가: 평균적인 블록 소모를 균등화하여 특정 셀의 조기 사망을 막음

- GC 효율 향상: 유효 데이터가 덜 집중되므로 GC 시에도 복사할 데이터가 적어짐

- 데이터 신뢰성 확보: 특정 셀에 무리가 가지 않기 때문에 ECC 복구 확률도 줄어듦
