## Reset 내용 정리
### ✅ Reset의 종류
1. Controller Reset
2. Subsystem Reset
3. PCIe Hot Reset
4. PCIe Cold Reset

### ▶️ 1. Controller Reset
#### 1) 개념
🔸 NVMe 디바이스의 하나의 컨트롤러만 논리적으로 초기화하는 과정(NVMe 컨트롤러를 껐다가 다시 킨 느낌)

🔸 Admin Queue, I/O Queue, Features 설정 등 컨트롤러 관련 상태 정보 전체를 초기 상태로 되돌리는 역할

#### 2) 구체적으로 수행하는 동작
🔸모든 Submission/Completion Queue가 삭제됨
> Submission Queue (SQ): 호스트가 "이거 읽어줘", "이거 써줘" 하고 요청을 적는 명령 리스트</br>
> Completion Queue (CQ): 컨트롤러가 "이 요청 끝났어!" 라고 결과를 적어주는 공간</br>

> 리셋이 되면 이 큐들이 모두 사라져서, 아무 명령도 보낼 수 없고 받을 수 없음</br>

🔸Features 명령어로 설정된 휘발성 설정(Volatile Features) 초기화
> Features 명령은 NVMe 장치의 작동 방식(성능, 전력 등)을 조정하는 명령</br>
> 이 중 일부는 휘발성(volatile)이라서, 전원이나 리셋 시 초기화(PM, Write Cache 등)</br>

🔸명령 대기 중인 큐의 명령은 실패(Command Aborted) 처리
> 리셋이 발생할 때 이미 큐에 올라간 명령(예: 읽기/쓰기 요청)이 있었다면, 마무리하지 않고 그냥 포기</br>

🔸Identify 정보를 다시 획득해야 함
> Identify 명령은 디바이스가 “나 이런 구조, 용량, 포맷이야” 라고 정보를 알려주는 명령</br>
> 이 명령을 통해 컨트롤러 정보/네임스페이스 정보 등을 확인

🔸Namespace는 비활성화 상태로 되며, 필요 시 다시 Attach 필요
> Namespace란 실제 데이터 저장 공간(SSD 파티션 같은 개념)</br>
> Host가 이 공간에 접근하기 위해서는 Attach(연결)되어 있어야 한다.

#### 3) 수행 방법
🔸NVMe에서 Controller Reset은 명령어 기반이 아니라, PCIe를 통한 레지스터 조작으로 수행

🔸즉 NVMe 레지스터의 CC, CSTS 등은 PCI BAR 영역을 통해 메모리 주소에 매핑 되므로, 이 주소에 read/write하는 코드를 작성함으로써 조작 가능

#### 4) 레지스터 변화: Controller Reset은 다음 두 레지스터를 사용해 수행
🔹CC (Controller Configuration Register)
> EN (Enable) 비트가 0이면 컨트롤러 비활성화 상태 -> 이 값을 통해 컨트롤러 리셋을 트리거

🔹CSTS (Controller Status Register)
> 현재 컨트롤러 상태를 나타냄 -> CSTS.RDY = 1 이면 컨트롤러 준비 완료, 0 이면 비활성화 또는 리셋

#### 5) 리셋 절차
> 1. CC.EN = 0                       ← 컨트롤러 비활성화 요청
> 2. Wait for CSTS.RDY = 0          ← 컨트롤러가 비활성화 되었는지 확인
> 3. (필요 시 Admin Queue 설정 재작성)
> 4. CC.EN = 1                       ← 컨트롤러 재활성화
> 5. Wait for CSTS.RDY = 1          ← 컨트롤러가 준비되었는지 확인
> 6. Identify 명령, Features 재설정, Namespace Attach 등 수행

### ▶️ 2. Subsystem Reset
#### 1) 개념
: NVMe Subsystem이란 하나 이상의 NVMe Controller와 Namespace들이 소속된 논리적 단위
: Subsystem Reset은 이 전체 묶음을 한 번에 리셋 하는 것(컨트롤러를 포함한 전체 NVMe 시스템을 초기화)

#### 2) Subsystem Reset의 트리거 조건
: 자동 발생하거나, 플랫폼/펌웨어/OS에서 제어할 수 있다.
- CC.EN 비트를 0으로 설정했는데 CSTS.RDY가 2초 안에 0이 되지 않음
→ 컨트롤러가 응답 불능이므로, 자동으로 Subsystem Reset 발생
- PCIe Cold Reset (전원 차단) 시 포함
- 펌웨어 명령 또는 시스템 이벤트로 Subsystem 전체 초기화
- 따라서 Admin command로는 조절X > 코드 상으로 CC.EN과 CSTS.RDY 상태를 제어함으로써 조절 가능

#### 3) 수행 동작
: 모든 컨트롤러 리셋, 모든 큐(Admin 및 I/O 큐) 제거, 모든 휘발성 Features 초기화, 모든 명령 상태 중간, 모든 네임스페이스 연결 비활성화(다시 Attach 필요) 등

#### 4) 레지스터 및 동작 흐름
: Subsystem Reset은 어떤 단일 레지스터를 write해서 직접 수행하지는 않지만, 보통 Controller Disable 실패 (CC.EN=0 후 CSTS.RDY=0 미 도달) 상황이 발생하면 트리거 됨.
🔹CC (Controller Configuration) > CC.EN = 0: 비활성화 요청
🔹CSTS (Controller Status) > RDY 비트가 2초 안에 0이 안 되면 → Subsystem Reset 발생

#### 5) 수행 흐름
> 1. OS가 CC.EN = 0으로 설정 (Controller Disable 시도)
> 2. CSTS.RDY != 0 상태가 2초 이상 유지됨
> 3. 컨트롤러가 응답 없음으로 판단됨
> 4. 펌웨어 또는 플랫폼이 Subsystem Reset을 강제 수행
> 5. 이후 모든 컨트롤러 및 자원이 재초기화됨
 
3. PCIe Hot Reset
1) 개념
: PCIe Hot Reset은 PCI Express 링크 상의 디바이스 연결을 끊고 다시 재 연결하는 동작
: 흔히 "PCIe 장치가 뺐다 다시 꽂힌 것처럼" 논리적으로 재협상(link re-training)을 수행
: 전원은 계속 연결되어 있고 신호만 끊음으로써 “다시 연결되었구나”라고 느끼도록
2) 목적
- 디바이스 오류 복구
	> 컨트롤러가 먹통일 때 재연결을 시도하여 recovery를 유도
- 링크 초기화
	> 전송 속도나 설정이 꼬였을 때 가시 초기 상태로 맞추기
- 펌웨어 업데이트 후 연결 회복 
	> 전원을 끄지 않고도 디바이스 전체 동작 상태 초기화 가능 
- PCIe 에러 복구 
	> Transaction Layer Packet(TLP) 오류가 계속 발생할 경우 recovery
3) 수행 동작
- PCIe링크 재훈련 발생: L0 → Detect → L0 
- Configuration 공간 초기화(BAR, ID등) -> OS가 재설정 해야함
- NVMe 컨트롤러의 CC, CSTS, 큐, 설정 모두 초기화됨(Controller Reset과 비슷한 상태)
- 메모리 매핑(MMIO) BAR주소 재할당 필요
- 진행 중인 I/O명령 모두 중단

 
4. PCIe Cold Reset (= PCIe Conventional Reset or Fundamental Reset)
1) 개념
: PCIe Cold Reset은 장치의 전원을 완전히 껐다가 다시 켜는 리셋
: 물리적으로든 논리적으로든, 이 Reset은 시스템이 해당 장치를 처음 부팅했을 때와 같은 상태로 되돌림
: 전지가 완전히 차단되므로 장치의 모든 휘발성 정보(MMIO 상태, Queue, 컨트롤러 레지스터, DRAM) 등이 완전히 초기화됨 -> 단순히 명령 큐나 설정만 지우는 게 아니라, 하드웨어 자체를 다시 부팅시키는 수준
2) 수행되는 방법
- 물리적 전원 차단: 서버 또는 시스템이 PCIe 슬롯에 공급되는 전원을 끊었다가 다시 넣음
- 시스템 또는 펌웨어 트리거: BIOS/UEFI, BMC, ACPI 등이 제어하여 장치 전원을 리사이클
- 전원 자체를 끊는 방법이므로 코드 상으로 구현 불가능

5) 추가 내용
* Reset 이후의 초기화 순서 (모든 Reset 이후에는 다음과 같은 순서로 장치를 재초기화해야 정상 동작)
> 1. Admin Queue 생성
> 2. Identify Controller 수행
> 3. 필요한 Features 설정
> 4. Namespace Attach
> 5. I/O Queue 생성
> 6. Identify Namespace 수행
 
* Admin Queue (Submission Queue, Completion Queue)에 대해서
1) 기본 개념 정리
- Submission Queue: Host가 NVMe 컨트롤러에게 명령을 “적어 넣는” 공간
- Completion Queue: NVMe 컨트롤러가 “이 명령 끝났어”라는 결과를 Host에 알려주는 공간
- Admin Queue: 이 SQ/CQ의 특수한 인스턴스 → Admin 명령 전용 (Identify, Features 등)
2) Admin Queue의 생성 흐름
> 1. 리셋 후 CC.EN = 0 유지
> 2. ASQ, ACQ, AQA 레지스터 설정 (주소 + 크기)
> 3. CC 레지스터 설정 → EN 비트 = 1 (컨트롤러 활성화)
> 4. CSTS.RDY == 1 되는지 확인
> 5. 이제 Admin Queue를 통해 Identify, Set Features 등 가능
3) Admin Queue로 명령 전송하는 과정
> Host는 SQ에 명령 (예: Identify)을 작성
> Doorbell (DB) 레지스터를 이용해 컨트롤러에게 알림
> 컨트롤러는 명령 수행 후 CQ에 Completion Entry 작성
> Host는 CQ에서 Completion 읽고 처리
