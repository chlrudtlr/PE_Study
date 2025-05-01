## SSD Power Management

#### 연결 : Host(PC) - Link(NVMe) - Device(SSD)
> PM 주체가 누구인지에 따라서 서로 다른 전력 상태 단계를 정의하고 있다.

---

#### ✅ Host단의 PM
> D0 : 장치가 완전히 작동 중 </br>
> D1/D2 : 장치가 부분적으로 꺼진 상태. 구현은 장치 제조사에 따라 다름 </br>
> D3 Hot : 드라이버가 장치 사용을 중단했지만 전원은 일부 공급 </br>
> D3 Cold	 : 장치에 전원이 완전히 차단됨. 재활성화하려면 재초기화 필요 </br>

#### ✅ NVMe/PCIe단의 PM
> L0 : 데이터가 정상적으로 전송되는 상태 </br>
> L0s : 아주 짧은 유휴 시간 동안 진입하는 경량 절전 모드 </br>
> L1 : SSD가 일정 시간 유휴일 때 진입. PHY(물리 계층)는 여전히 활성화됨 </br>
> L1.1 : PHY 일부 전원 차단 </br>
> L1.2 : PHY 전체 차단. 소비 전력 수 µW 수준까지 낮아짐 </br>

✏️ ASPM(Active State Power Management)</br>
> PCIe 인터페이스에서 동작하는 링크 절전 기술

#### ✅ Device단의 PM
> PS0 : 최고 성능 제공, 전력 소비도 가장 큼 </br>
> PS1 : 성능과 전력 절감 사이의 절충 상태 </br>
> PS2 : NAND, DRAM, Controller 일부 비활성화 </br>
> PS3 : 거의 모든 회로가 꺼짐, 전력 소비 수 µW 수준 </br>

✏️ APST(Autonomous Power State Transition)
> SSD 내부의 자율 전력 상태 전환 기능 </br>
> SSD가 유휴 상태일 때 Host 명령 없이도 PS 상태(PS0~PSn)로 자동 진입 </br>

---

#### 🔆 3가지 states는 서로 독립적으로 정의되어 있지만 실제 동작 시에는 서로 연관성이 있음
- 예를 들어 Host가 D3로 진입하면 Device나 Link도 각각 PS3/4, L1.2등으로 진입
- PCIe가 L1.2로 진입하면 Device가 Host의 명령을 못 받으므로 Idle로 인식하고 state 낮춤
- 이론적으로 독립적이므로 다양한 경우의 수가 가능하지만 현실 세계에서는 그렇지 않음 
