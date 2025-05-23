## PCIe Generation

### ✅ PCIe Geration의 개요

PCIe는 CPU와 다양한 장치(GPU, SSD등) 간의 고속 통신을 위한 직렬 인터페이스이며 각 세대(Gen)는 다음과 같은 측면에서 발전되어왔다.

1. `전송 속도(per lane)` : 한 레인(lane)당 보낼 수 있는 양(GT/s)

2. `대역폭 (x1, x4, x8, x16)` : 레인 수에 따라 총 데이터 처리량

3. `인코딩 방식` : 8b/10b, 128b/130b 등의 신호 인코딩

>  8b/10b 인코딩: 실제 데이터의 80%만 사용 가능
>
> 128b/130b 인코딩: 약 98.5% 효율

4. `지연시간(latency), 전력 효율` : 세대가 높을수록 개선됨

### ✅ PCIe 세대별 상세 비교

| Generation | 전송 속도 (1 lane) | 인코딩 방식  | 실제 대역폭 (x1 기준) | 출시 시기   | 주요 특징                                   |
|------------|--------------------|---------------|------------------------|-------------|----------------------------------------------|
| **Gen1**   | 2.5 GT/s           | 8b/10b        | 250 MB/s               | 2003        | 초기 버전, 인코딩 오버헤드 큼                |
| **Gen2**   | 5.0 GT/s           | 8b/10b        | 500 MB/s               | 2007        | 속도 2배 증가                                 |
| **Gen3**   | 8.0 GT/s           | 128b/130b     | 약 985 MB/s            | 2010        | 인코딩 효율 향상, 실효 대역폭 증가          |
| **Gen4**   | 16.0 GT/s          | 128b/130b     | 약 1.97 GB/s           | 2017        | SSD 성능 급상승 시작                         |
| **Gen5**   | 32.0 GT/s          | 128b/130b     | 약 3.94 GB/s           | 2019~2021   | AI, 고성능 컴퓨팅용 서버에 주로 사용         |
| **Gen6**   | 64.0 GT/s (예정)   | PAM-4         | 약 7.88 GB/s           | 2023~2024   | PAM4 도입, ECC 추가, 고속 신호 전송 가능    |

### ✔️GT/s 란? (GigaTransfers per second)

GT/s는 PCIe에서 1초 동안 신호를 몇 번 전송할 수 있는지를 나타낸다(하드웨어 전송속도).

 "Transfer"는 단순히 비트가 아니라, 신호의 전환(transition) 을 의미한다.

 즉 PCIe Gen3는 8.0 GT/s이므로 초당 80억 번 신호 전송이 가능하다는 뜻이다.

 ### ✔️GB/s란? (GigaBytes per second)

1초에 실제로 몇 기가바이트의 데이터를 전송했는가를 의미한다.

### ✔️GT/s ↔ GB/s 변환 방법 ( ex. PCIe Gen3 / 8.0 GT/s / 128b/130b 인코딩 / x1 레인 기준)

1. 전송 속도 : **8.0GT/s** -> **80억 전송/초**

2. 인코딩 효율 : **128/130b** -> 유효 데이터 비율 ≈ **98.56%**

3. 비트 단위로 환산 : **8.0 GT/s × 98.56% ≈ 7.88 Gbps**

4. 바이트 환산(8비트 = 1바이트) : 7.88 Gbps ÷ 8 ≈ **985 MB/s**

∴ Gen3 (1 lane)은 8.0 GT/s지만 실제 **985 MB/s (약 1 GB/s)** 가 전송됨 (다른 Gen들도 8배 정도의 차이로 생각하면 편함)

### ✅ Link Training

PCIe 장치들이 통신을 시작하기 전에 서로 적절한 연결 조건을 결정하는 협상 과정

이 과정을 통해 두 장치는 서로의 상태를 확인하고, 전송 속도(Gen), 레인 수, 그리고 신호 품질 등을 협상한 뒤 실제 통신을 시작한다.

#### ✏️ Link Training이 필요한 이유

1. 각 장치는 지원하는 최대 Gen 속도가 다르다.

> 예를 들어서 PC는 Gen 5를 지원하지만 SSD가 Gen 4까지만 지원할 수도 있음  -> 이런 경우 Gen 5로 통신 불가

2. 장치 간 물리적 연결 상태(레인 수, 신호 품질)가 다양합니다

3. 무조건 최고 속도로 연결하면 신호 손실, 에러가 발생할 수 있으므로,

4. 최적의 속도/레인/신호 환경을 찾아주는 훈련 과정이 필요합니다

#### ✏️ LTSSM (Link Training and Status State Machine)

Link Training은 여러 States를 거치며 진행되고 이들을 LTSSM이라고 한다.

LTSSM은 PCIe PHY 레벨에서 동작하며, 링크가 형성되고 안정적인 상태(L0)에 도달하기까지의 전체 상태 전이(State Transitions)를 제어한다.

PCIe 링크가 초기화되면, 다음 단계들이 자동으로 진행된다

**1. Detect State**

▪️Root Complex(또는 Switch)는 PCIe 슬롯에 전기적으로 장치가 연결되었는지(Presence Detect) 확인한다.

▪️Receiver가 Beacon 신호 또는 전압 변화를 감지한다.

▪️TX와 RX가 둘 다 Active 상태로 진입할 준비가 되었는지 확인한다.

**2.Polling State**

▪️시작은 항상 Gen1 (2.5GT/s) 속도에서 시작

▪️각 장치는 반복적으로 **Training Sequences (TS1, TS2)** 라는 특수 패턴을 교환하며 연결 시도

> TS1, TS2에 담긴 정보 : 장치의 Device Type, link number, 지원 속도 및 레인 수
>
> `TS1` : 초기 신호 정렬 및 Receiver 감지
>
> `TS2` : 속도/레인 수 협상 및 준비 상태 통지

▪️신호가 올바르게 수신되는지 확인

> 양쪽에서 TS2 수신이 완료되면 → Configuration 상태로 전이

**3.Configuration State**

▪️서로가 지원하는 최대 Gen과 레인 수 협상

> 예: Host는 Gen5 지원, 장치는 Gen3만 지원 → Gen3로 동작

▪️Lane별 상태 확인 (예: x4, x8, x16)

▪️이 시점부터 인코딩 동기화, Symbol Alignment, Elastic Buffer 초기화 등의 설정이 진행된다.

**4.Recovery State**

▪️실제 협상된 속도 (예: Gen3, Gen4 등)로 속도 전환

▪️신호 품질 불량 감지 시 → Gen 속도 낮추거나 레인 수 줄이고 다시 Polling 상태로 돌아감

**5.L0(Active)**

▪️최종 링크 설정 완료 → 정상 데이터 전송 시작

▪️TLP(Transaction Layer Packet)와 DLLP(Data Link Layer Packet) 전송 가능

▪️이 상태에서 NVMe Admin Queue 구성, I/O 요청 처리 가능해진다.

### ✅Gen 전환과 하드웨어 레지스터

PCIe 장치의 **Gen (속도)** 를 소프트웨어 또는 펌웨어/드라이버가 강제로 바꾸거나 확인하는데에 관련있는 핵심 레지스터는 아래와 같다.

| 레지스터 이름 | 역 |
|------------|--------------------|
| **PCI_EXP_LNKCAP**   | 장치가 최대 지원하는 속도/레인 수 정보 제공           |
| **PCI_EXP_LNKCTL2**   | 목표 속도 (Target Link Speed) 설정 가능           |
| **PCI_EXP_LNKSTA**   | 현재 속도 및 레인 수 (Current Link Status) 조회           |
| **PCI_EXP_LNKSTA2**   | 링크 오류 상태 등 추가 정보 제공 (선택 사항)          |

이들은 모두 **PCIe의 Configuration Space** 안의 **Capability 구조체** 중 **PCI Express Capability 영역**에 위치해 있다.

▶ **Configuration Space**

```
[00h ~ FFh]   : Standard PCI Header
[100h ~ xxx]  : Extended Capabilities (ID 0x10 for PCIe Capability)
```

▶ **Capability 구조체**

```
Offset (from PCIe Cap Base):
- 0x0: PCI Express Capabilities Header
- 0x0C: Link Capabilities (LNKCAP)
- 0x10: Link Control / Status (LNKCTL / LNKSTA)
- 0x30: Link Control 2 / Status 2 (LNKCTL2 / LNKSTA2)
```

⭐ **Target Link Speed 설정 : LNKCTL2 레지스터**

▪️ 레지스터 위치: PCIe Capability + Offset 0x30 (Link Control 2 Register)

▪️비트 설명 : [3:0] = Target Link Speed

> 1h: Gen1 (2.5 GT/s)
>
> 2h: Gen2 (5.0 GT/s)
>
> 3h: Gen3 (8.0 GT/s)
>
> 4h: Gen4 (16.0 GT/s)
>
> 5h: Gen5 (32.0 GT/s)

⭐ **Link Training 재개 요청**

▪️ 일반적으로 LNKCTL 레지스터 (0x10)의 "Retrain Link" 비트를 설정함으로써 Link Training 재시

> Bit 5 : Retrain Link → 1로 설정하면 Link Training을 다시 시작
>
> 이 시점부터 LTSSM은 다시 Polling → Configuration → Recovery → L0 단계를 거치며 새로 설정된 Target Link Speed로 연결을 재시도

### ✅Gen 전환 수행 방법(Linux 명령어)

**0. 준비 사항**

```bash
sudo apt install pciutils
```

**1. 장치의 최대 지원 속도 확인(LNKCAP)**

```bash
sudo lspci -vvv -s <DEVICE_BDF>
```

▫️ 입력 예시

```bash
sudo lspci -vvv -s 0000:01:00.0
```

▫️ 출력 예시

```yaml
LnkCap: Port #0, Speed 8GT/s, Width x4
LnkSta: Speed 5GT/s, Width x4
```

> LnkCap은 장치의 최대 지원 속도, LnkSta는 현재 동작 속도

**2. Target Link Speed 설정(LNKCTL2)**

```bash
# 예: Gen2로 강제 설정 (value 2 = 0x2)
sudo setpci -s 0000:01:00.0 ECAP_PCI_EXP+30.w=0x2
```

▫️ECAP_PCI_EXP는 일반적으로 0x100 번지에 있으므로 아래와 같이 지정할 수도 있다.

```bash
sudo setpci -s 0000:01:00.0 100.w
sudo setpci -s 0000:01:00.0 130.w=0x0002   # LNKCTL2
```

**3. Retrain Link 비트 설정(LNKCTL)**

```bash
# Bit 5 (0x20)를 set → Link Retrain 요청
sudo setpci -s 0000:01:00.0 110.w=0x0020
```

> 해당 명령은 커널에서 관리 중인 PCIe 장치일 경우 바로 효과가 없을 수도 있으며, 이 경우 PERST# 리셋이나 unbind/rebind가 필요할 수 있다.

**4. 변경 여부 확인**

```bash
sudo lspci -vvv -s 0000:01:00.0 | grep LnkSta
```

▫️ 출력 예시
```bash
LnkSta: Speed 5GT/s (downgraded), Width x4
```

**5. 참고 : 만약 Reset이 필요하면?**

```bash
# Unbind 후 rebind 방법
echo 0000:01:00.0 | sudo tee /sys/bus/pci/devices/0000:01:00.0/driver/unbind
echo 0000:01:00.0 | sudo tee /sys/bus/pci/drivers/<driver_name>/bind
```

### ✅Gen 전환 수행 방법(C++ 코드)

리눅스에서 PCI configuration space를 C++로 접근하는 방법은 아래와 같으며 3번째 방법이 안정적이고 범용적인 방법으로 알려져있다.

▪️sysfs (/sys/bus/pci/devices/)

▪️/dev/mem 또는 iopl() + inline asm

▪️libpci (pciutils 제공) – 아래에 소개할 방법

**1. 준비 과정**

```bash
sudo apt install libpci-dev
```

**2. 예제 코드 (LNKCAP, LNKSTA, LNKCTL2 접근)**

```cpp
#include <pci/pci.h>
#include <cstdio>

int main() {
    struct pci_access *pacc;
    struct pci_dev *dev;
    char namebuf[1024];

    pacc = pci_alloc();
    pci_init(pacc);
    pci_scan_bus(pacc);

    for (dev = pacc->devices; dev; dev = dev->next) {
        pci_fill_info(dev, PCI_FILL_IDENT | PCI_FILL_BASES | PCI_FILL_EXT_CAPS);

        // 예: NVMe 장치만 필터링
        if (dev->device_class == 0x010802) {  // NVMe class code
            printf("Device: %02x:%02x.%d\n", dev->bus, dev->dev, dev->func);

            // PCIe Capability offset 추출
            uint8_t pcie_cap = pci_find_cap(dev, PCI_CAP_ID_EXP);
            if (!pcie_cap) continue;

            // LNKCAP (max supported speed)
            uint32_t lnkcap = pci_read_long(dev, pcie_cap + 0x0C);
            uint32_t max_speed = lnkcap & 0xF;
            printf("LNKCAP max speed: Gen%d\n", max_speed);

            // LNKSTA (current speed)
            uint16_t lnksta = pci_read_word(dev, pcie_cap + 0x12);
            uint16_t cur_speed = lnksta & 0xF;
            printf("LNKSTA current speed: Gen%d\n", cur_speed);

            // LNKCTL2 (set target speed to Gen2)
            pci_write_word(dev, pcie_cap + 0x30, 0x2);
            printf("Target speed set to Gen2.\n");

            // LNKCTL: Retrain bit set
            uint16_t lnkctl = pci_read_word(dev, pcie_cap + 0x10);
            lnkctl |= (1 << 5);  // Set bit 5
            pci_write_word(dev, pcie_cap + 0x10, lnkctl);
            printf("Link retrain requested.\n");
        }
    }

    pci_cleanup(pacc);
    return 0;
}
```

**3. 빌드**

```bash
g++ -o pci_speed_setter your_code.cpp -lpci
```

> 이 코드는 루트 권한에서 실행되어야 하며, 일부 디바이스는 커널이 점유하고 있으면 효과가 없을 수 있다.

