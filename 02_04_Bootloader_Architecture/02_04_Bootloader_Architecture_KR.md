**Volume 02 Embedded Software Engineering**

# 04. Bootloader Architecture

## 04.01 Bootloader Role and Boot Sequence Principles

![](images/image1.png){width="7.268055555555556in" height="7.268055555555556in"}

부트로더(Bootloader)는 임베디드 프로세서(Embedded Processor)가 리셋(Reset) 상태를 벗어난 직후 제어된 실행 환경을 구축하는 최초의 소프트웨어 구성요소(Software Component)이다. 부트로더의 목적은 단순히 메인 애플리케이션(Main Application)을 시작하는 것이 아니라, 하드웨어에 의해 정의되는 리셋 동작과 소프트웨어에 의해 정의되는 시스템 동작 사이를 연결하는 것이다. 로봇 임베디드 시스템(Robotic Embedded System)에서는 컨트롤러(Controller), 센서(Sensor), 통신 인터페이스(Communication Interface), 안전 회로(Safety Circuit), 상위 컴퓨팅 모듈(Computing Module)이 서로 조정된 시작 과정에 의존할 수 있으므로 이러한 전환 과정은 예측 가능해야 한다.

전원이 인가되거나 리셋 이벤트(Reset Event)가 발생하면 프로세서는 아키텍처에서 정의된 리셋 위치(Reset Location)에서 실행을 시작한다. 이 단계에서는 애플리케이션 소프트웨어(Application Software)가 일반적으로 기대하는 대부분의 런타임 환경(Runtime Environment)이 아직 존재하지 않는다. 클록 소스(Clock Source)는 보수적인 기본 설정을 사용할 수 있고, 외부 메모리(External Memory)는 사용할 수 없으며, 주변장치(Peripheral)는 비활성화되어 있을 수 있고, 휘발성 메모리(Volatile Memory)에는 신뢰할 수 있는 애플리케이션 상태가 존재하지 않는다. 따라서 초기 부트 코드(Boot Code)는 엄격한 가정 아래에서 동작하며 이후의 실행을 안전하고 결정론적으로 만들기 위해 필요한 초기화만 수행한다.

기본적인 부트 시퀀스(Boot Sequence)는 리셋 처리(Reset Handling), 하드웨어 초기화(Hardware Initialization), 이미지 검증(Image Validation), 실행 준비(Execution Preparation), 애플리케이션 제어권 전달(Application Transfer)로 이어지는 과정으로 이해할 수 있다. 구체적인 구현 방식은 마이크로컨트롤러(Microcontroller), 애플리케이션 프로세서(Application Processor), 임베디드 PC(Embedded PC)에 따라 달라지지만 아키텍처 원리는 유사하다. 각 단계는 다음 단계에 필요한 조건을 확립하며, 전체 펌웨어(Firmware), 운영체제(Operating System), 또는 로봇 애플리케이션 환경이 정상적으로 실행될 수 있을 때까지 불확실성을 점진적으로 감소시킨다.

초기 초기화(Early Initialization)는 일반적으로 프로세서 모드(Processor Mode), 스택 구성(Stack Configuration), 인터럽트 동작(Interrupt Behavior), 클록 설정(Clocking), 메모리 접근(Memory Access), 필수 하드웨어 상태(Essential Hardware State)를 설정한다. 단순한 마이크로컨트롤러에서는 이러한 작업 이후 거의 즉시 애플리케이션 시작 단계로 이동할 수 있다. 반면 고성능 프로세서는 대규모 소프트웨어 이미지(Software Image)를 로드하기 전에 DRAM 컨트롤러(DRAM Controller), 캐시(Cache), 메모리 관리 하드웨어(Memory Management Hardware), 저장장치 인터페이스(Storage Interface), 보드별 장치(Board-Specific Device)를 초기화해야 하는 경우가 많다. 이러한 차이로 인해 부트로더는 매우 작은 MCU 프로그램부터 복잡한 다단계 소프트웨어 시스템(Multi-Stage Software System)까지 다양한 형태를 갖는다.

부트로더는 실행 이미지(Executable Image)를 어떻게 검색하고 표현할 것인지도 정의한다. 펌웨어는 내부 플래시(Internal Flash)에 직접 저장될 수 있으며, 더 큰 임베디드 시스템은 eMMC, SD 카드(SD Card), NAND 또는 NOR 플래시(Flash), NVMe 저장장치(Storage), USB 장치(Device), 네트워크(Network)로부터 이미지를 로드할 수 있다. 부트로더는 지정된 이미지를 찾고 해당 이미지가 어디에서 실행되어야 하는지 판단할 수 있을 정도로 저장장치 구조(Storage Layout)를 이해해야 한다. 따라서 파티션 테이블(Partition Table), 고정 오프셋(Fixed Offset), 이미지 헤더(Image Header), 메타데이터(Metadata), 구성 레코드(Configuration Record)가 전체 부트 아키텍처(Boot Architecture)의 일부가 된다.

이미지 검증(Image Validation)은 손상되거나 호환되지 않는 소프트웨어로 제어권을 전달할 경우 로봇이 예측할 수 없는 상태에 놓일 수 있기 때문에 또 하나의 핵심 책임이다. 기본적인 구현에서는 매직 값(Magic Value), 이미지 길이(Image Length), 버전 필드(Version Field), 체크섬(Checksum), CRC 등을 검증할 수 있다. 더욱 발전된 시스템에서는 암호학적 해시(Cryptographic Hash)와 디지털 서명(Digital Signature)까지 검증한다. 이러한 메커니즘은 이미지가 단순히 읽을 수 있는지를 확인하는 문제와 이미지의 무결성(Integrity), 호환성(Compatibility), 실행 권한(Authorization)을 확인하는 보다 중요한 문제를 분리한다.

부트로더는 자신의 실행 컨텍스트(Execution Context)와 애플리케이션의 실행 컨텍스트 사이에 명확한 경계를 유지해야 한다. 제어권을 전달하기 전에 애플리케이션의 벡터 테이블(Vector Table), 초기 스택 포인터(Initial Stack Pointer), 프로세서 권한 상태(Processor Privilege State), 메모리 매핑(Memory Mapping), 인터럽트 상태(Interrupt State), 부트 매개변수(Boot Parameter)를 구성할 수 있다. 부트로더가 남긴 설정은 의도적으로 상속되거나 사전에 합의된 상태로 복원되어야 한다. 그렇지 않으면 애플리케이션의 동작이 이전 부트 단계에서 하드웨어를 어떻게 구성했는지에 따라 달라질 수 있다.

하나의 프로그램이 모든 시작 기능을 효율적으로 수행할 수 없는 경우 부트 아키텍처는 여러 단계(Multiple Stages)를 사용하는 경우가 많다. 작고 변경되지 않거나 높은 신뢰성을 가진 1단계 부트로더(First-Stage Bootloader)는 더 큰 2단계 부트로더(Second-Stage Bootloader)를 검색하고 검증하는 데 필요한 자원만 초기화할 수 있다. 이후 2단계에서는 저장장치 드라이버(Storage Driver), 구성 처리(Configuration Processing), 네트워킹(Networking), 진단(Diagnostics), 복구 기능(Recovery Function), 운영체제 로딩(Operating-System Loading) 등을 제공할 수 있다. 이러한 다단계 모델(Staged Model)은 초기 신뢰 코드(Trusted Code)의 복잡성을 줄이면서 후속 단계에서 보다 풍부한 플랫폼 기능을 지원할 수 있게 한다.

모든 부팅이 동일한 시스템 상태에서 시작되는 것은 아니므로 리셋 원인(Reset Cause)도 중요하다. 전원 인가 리셋(Power-On Reset), 워치독 리셋(Watchdog Reset), 소프트웨어 리셋(Software Reset), 브라운아웃(Brownout), 보안 리셋(Security Reset), 복구 요청(Recovery Request)은 서로 다른 동작을 요구할 수 있다. 견고한 부트로더는 리셋 원인을 기록하거나 해석하여 적절한 부트 경로(Boot Path)를 선택한다. 예를 들어 반복되는 워치독 리셋은 현재 애플리케이션이 정상적으로 시작하지 못한다는 것을 의미할 수 있으며, 명시적인 유지보수 요청(Maintenance Request)은 정상 동작을 시작하는 대신 업데이트(Update) 또는 진단 모드(Diagnostic Mode)에 머물도록 시스템에 지시할 수 있다.

부트로더는 펌웨어 업데이트 아키텍처(Firmware Update Architecture)와 밀접하게 연결된다. 실행 중인 애플리케이션이 항상 자기 자신을 안전하게 교체할 수 있는 것은 아니므로 부트로더는 새로운 소프트웨어를 설치하거나 활성화하기 위한 독립적인 환경을 제공할 수 있다. 업데이트 메타데이터(Update Metadata)는 활성 이미지(Active Image), 후보 이미지(Candidate Image), 버전(Version), 무결성 정보(Integrity Information), 이전의 정상 상태(Known-Good State)를 식별할 수 있다. 이러한 분리는 원격으로 배치된 로봇에서 특히 중요하며, 펌웨어 업데이트 실패로 인해 컨트롤러가 영구적으로 동작하지 않거나 즉각적인 현장 정비가 필요한 상황을 방지하는 데 기여한다.

따라서 신뢰성 높은 설계는 이미지 설치(Image Installation)와 성공적인 활성화(Successful Activation)를 구분한다. 새롭게 기록된 이미지는 먼저 검증되고 후보 상태로 표시된 다음 통제된 조건에서 부팅될 수 있다. 애플리케이션이 성공적으로 시작되었다는 것을 입증한 이후에만 시스템은 해당 이미지를 유효한 이미지로 확정해야 한다. 부팅 시도가 반복적으로 실패하면 부트로더는 이전 이미지 또는 복구 환경(Recovery Environment)으로 돌아갈 수 있다. 이러한 원리는 신뢰성 높은 OTA 시스템에서 사용하는 이중 파티션 A/B 부트 아키텍처(Dual-Partition A/B Boot Architecture)와 롤백 메커니즘(Rollback Mechanism)의 개념적 기반을 제공한다.

보안(Security)은 부트로더를 단순한 시작 메커니즘에서 플랫폼 신뢰(Platform Trust)를 강제하는 지점으로 확장한다. 공격자가 정상적인 보안 제어가 활성화되기 전에 실행 소프트웨어를 교체할 수 있다면 운영체제나 애플리케이션에서 구현된 보호 기능을 우회할 수 있다. 따라서 보안 부트 체인(Secure Boot Chain)은 다음 실행 단계를 검증하는 신뢰 요소(Trusted Element)에서 시작되며, 검증된 단계가 다시 다음 단계를 검증하는 방식으로 이어진다. 즉, 시스템이 시작된 이후 신뢰를 단순히 가정하는 것이 아니라 전체 부트 시퀀스를 따라 신뢰가 연속적으로 전달된다.

로봇 시스템(Robotic System)은 소프트웨어 시작 과정이 물리적 액추에이터(Physical Actuator)와 상호작용하기 때문에 부트 동작에 추가적인 요구사항을 가진다. 모터 드라이버(Motor Driver), 브레이크(Brake), 매니퓰레이터(Manipulator), 추진 시스템(Propulsion System) 및 기타 위험 가능성이 있는 출력은 이를 제어하는 소프트웨어가 검증되고 초기화될 때까지 정의된 안전 상태(Safe State)를 유지해야 한다. 부트로더 또는 관련 시작 하드웨어(Startup Hardware)는 의도하지 않은 출력 변화를 방지해야 하며, 정상 동작에 대한 제어 권한을 전달하기 전에 워치독(Watchdog), 안전 컨트롤러(Safety Controller), 전원 시퀀싱 로직(Power Sequencing Logic)과 협조해야 한다.

부트 시점 진단(Boot-Time Diagnostics)은 부트로더를 완전한 애플리케이션 환경으로 만들지 않으면서 유지보수성을 향상시킬 수 있다. 메모리 검사(Memory Check), 저장장치 상태(Storage Status), 펌웨어 메타데이터(Firmware Metadata), 리셋 이력(Reset History), 보드 식별(Board Identification), 필수 통신 진단(Communication Diagnostics)은 시스템이 시작되지 못한 원인을 파악하는 데 도움을 준다. 그러나 기능이 추가될 때마다 부트로더의 복잡성과 공격 표면(Attack Surface)도 증가한다. 따라서 좋은 아키텍처는 초기 부트 기능을 최소화하고 필수적이지 않은 진단과 서비스를 이후의 교체 가능한 소프트웨어 계층(Software Layer)에 배치한다.

부트 시간(Boot Time) 자체도 중요한 아키텍처 고려사항이다. 로봇 컨트롤러는 전원 중단이나 리셋 이후 빠르게 복구해야 할 수 있지만, 필요한 초기화 또는 검증을 생략하여 시작 시간을 단축하면 신뢰성이 저하될 수 있다. 대신 부트 최적화(Boot Optimization)는 불필요한 지연을 제거하고, 하드웨어가 허용하는 경우 독립적인 초기화를 병렬화하며, 중복된 장치 검색(Device Probing)을 줄이고, 중요하지 않은 서비스를 이후 단계로 연기해야 한다. 목표는 단순히 가장 짧은 부트 시간이 아니라 요구되는 안전성(Safety), 무결성(Integrity), 결정론적 동작(Deterministic Behavior)을 유지하면서 가능한 가장 짧은 시작 경로를 만드는 것이다.

잘 설계된 부트로더는 궁극적으로 불확실한 하드웨어 리셋 상태에서 신뢰할 수 있는 애플리케이션 실행(Trusted Application Execution)으로 이어지는 통제된 체인을 제공한다. 부트로더의 책임은 초기화(Initialization), 이미지 선택(Image Selection), 검증(Validation), 복구(Recovery), 업데이트 지원(Update Support), 보안 강제(Security Enforcement), 제어권 전달(Transfer of Control)을 포함한다. 전체 임베디드 소프트웨어 구조(Embedded Software Structure)에서 이러한 원리는 이후에 다루게 될 ARM 전용 부트 흐름(ARM-Specific Boot Flow), U-Boot 사용자 정의(U-Boot Customization), 보안 부트(Secure Boot), A/B 업데이트(A/B Update), 복구 메커니즘(Recovery Mechanism), UEFI, 부트 시간 최적화(Boot-Time Optimization), 로봇 전원 시퀀싱(Robot Power Sequencing), 부트로더 보안(Bootloader Security)을 이해하기 위한 기반을 형성한다.

## 04.02 ARM Boot Sequence: Reset Vector to Application

![](images/image2.png){width="7.268055555555556in" height="7.268055555555556in"}

ARM 기반 임베디드 시스템(ARM-based Embedded System)의 부트 시퀀스(Boot Sequence)는 리셋 로직(Reset Logic)이 프로세서를 정의된 아키텍처 상태(Architectural State)로 설정하고 최초 실행 주소를 결정하면서 시작된다. 프로세서는 즉시 로봇 애플리케이션이나 운영체제(Operating System)를 실행하지 않는다. 대신 하드웨어에 의해 정의된 리셋 동작이 초기 실행 컨텍스트(Execution Context)를 설정하고, 이후 시작 소프트웨어(Startup Software)가 메모리, 클록, 보안 상태, 인터럽트 및 후속 소프트웨어 단계에 필요한 자원을 구성할 수 있도록 한다.

ARM Cortex-M 마이크로컨트롤러(Microcontroller)에서는 벡터 테이블(Vector Table)이 리셋과 소프트웨어 실행을 연결하는 기본적인 역할을 한다. 벡터 테이블의 시작 부분에는 일반적으로 초기 메인 스택 포인터(Main Stack Pointer) 값과 리셋 핸들러(Reset Handler)의 주소가 저장된다. 리셋 이후 프로세서는 이러한 값을 가져와 리셋 루틴(Reset Routine)을 실행하기 시작한다. 이 간결한 메커니즘을 통해 마이크로컨트롤러는 하드웨어 리셋에서 결정론적인 펌웨어 초기화(Firmware Initialization) 과정으로 빠르게 전환할 수 있다.

리셋 핸들러(Reset Handler)는 컴파일된 C 또는 C++ 소프트웨어가 필요로 하는 런타임 조건(Runtime Condition)을 구축하는 역할을 담당한다. 이 시점에서는 초기화된 전역 변수(Global Variable)가 RAM에 존재한다고 단순히 가정할 수 없으며, 초기화되지 않은 변수에는 임의의 값이 포함될 수 있다. 따라서 시작 코드(Startup Code)는 비휘발성 메모리(Nonvolatile Memory)의 로드 위치에 저장된 초기화 데이터를 RAM으로 복사하고, BSS 영역(BSS Region)을 초기화하여 0으로 초기화되어야 하는 변수가 프로그래밍 언어의 런타임 가정을 충족하도록 한다.

애플리케이션 코드(Application Code)가 실행되기 전에 프로세서별 초기화 과정에서 클록 소스(Clock Source), 위상 고정 루프(Phase-Locked Loop, PLL), 플래시 대기 상태(Flash Wait State), 캐시 동작(Cache Behavior), 부동소수점 하드웨어(Floating-Point Hardware), 메모리 보호(Memory Protection), 인터럽트 우선순위(Interrupt Priority) 등을 구성할 수 있다. 플래시 타이밍이나 전압 조건을 설정하기 전에 프로세서 클록을 변경하면 불안정한 실행이 발생할 수 있으므로 초기화 순서는 중요하다. 따라서 시작 소프트웨어는 MCU, SoC, 보드 설계(Board Design), 선택된 동작 주파수(Operating Frequency)에 의해 정의된 의존 관계를 따라야 한다.

지원되는 시스템에서는 벡터 테이블 오프셋 레지스터(Vector Table Offset Register)를 사용하여 인터럽트 벡터 테이블(Interrupt Vector Table)을 원래의 부트 주소에서 다른 메모리 영역으로 재배치할 수 있다. 이 기능은 부트로더와 애플리케이션이 서로 다른 플래시 영역을 사용하는 경우 특히 중요하다. 부트로더는 시작 과정에서 자신의 벡터를 사용한 후 제어권이 애플리케이션으로 전달되면 예외(Exception)와 인터럽트가 애플리케이션의 벡터 테이블을 통해 처리되도록 프로세서를 구성할 수 있다.

ARM Cortex-A 코어(Core)를 기반으로 하는 애플리케이션 프로세서(Application Processor)는 일반적으로 Cortex-M 장치보다 복잡한 부트 경로(Boot Path)를 필요로 한다. 프로세서는 반도체 공급업체가 제공하는 변경 불가능한 부트 ROM(Boot ROM) 코드에서 실행을 시작할 수 있다. 이 ROM은 설정된 부트 소스(Boot Source)를 결정하고 최소한의 하드웨어를 초기화한 다음 실행 가능한 1단계 이미지(First-Stage Image)를 검색한다. 부트 소스에는 플래시, eMMC, SD 저장장치, USB 복구 인터페이스(Recovery Interface) 또는 기타 플랫폼별 메커니즘이 포함될 수 있다.

부트 ROM에 의해 로드되는 최초의 소프트웨어 단계는 일반적으로 해당 시점에서 사용할 수 있는 제한적인 하드웨어 환경의 제약을 받는다. 외부 DRAM을 아직 사용할 수 없을 수 있으므로 초기 코드는 내부 SRAM이나 다른 작은 메모리 영역에서 실행될 수 있다. 이 단계의 주요 역할은 이후의 더 큰 소프트웨어 단계를 실행할 수 있도록 충분한 저수준 초기화(Low-Level Initialization)를 수행하는 것이며, 특히 DRAM 초기화, 클록 구성, 핀 멀티플렉싱(Pin Multiplexing), 영구 저장장치(Persistent Storage) 접근이 중요하다.

DRAM을 사용할 수 있게 되면 더 큰 부트로더 단계(Bootloader Stage)가 운영체제 또는 애플리케이션을 로드하는 데 필요한 환경을 구축할 수 있다. 이 단계에서는 저장장치 컨트롤러(Storage Controller), 네트워크 인터페이스(Network Interface), 직렬 콘솔(Serial Console), 캐시(Cache), 메모리 관리 기능(Memory Management Facility), 보드별 주변장치(Board-Specific Peripheral) 등을 초기화할 수 있다. 임베디드 리눅스(Embedded Linux) 플랫폼에서는 U-Boot와 같은 소프트웨어가 일반적으로 이러한 역할을 담당하지만 구체적인 시퀀스는 프로세서 공급업체와 대상 시스템의 플랫폼 아키텍처에 따라 달라진다.

ARM 부트 흐름(Boot Flow)은 프로세서 권한(Processor Privilege)과 실행 레벨(Exception Level)도 고려해야 한다. Cortex-A 아키텍처는 보안 펌웨어(Secure Firmware), 하이퍼바이저(Hypervisor), 운영체제 커널(Operating-System Kernel), 애플리케이션을 분리하기 위한 여러 예외 레벨(Exception Level)을 제공할 수 있다. 초기 펌웨어는 후속 소프트웨어가 진입할 실행 상태를 결정하고 이에 따라 아키텍처 자원을 구성한다. 권한 전환(Privilege Transition)이 잘못되면 커널이 시작되지 않거나 격리되어야 할 자원이 의도하지 않게 노출될 수 있다.

TrustZone을 지원하는 ARM 플랫폼은 보안 실행 환경(Secure Execution Environment)과 비보안 실행 환경(Non-secure Execution Environment)을 분리함으로써 또 다른 계층을 제공한다. 보안 시작 코드(Secure Startup Code)는 낮은 권한의 소프트웨어에 실행을 전달하기 전에 메모리 속성(Memory Attribution), 주변장치 접근 권한(Peripheral Access Permission), 신뢰 펌웨어 서비스(Trusted Firmware Service), 보안 경계(Security Boundary)를 설정할 수 있다. 따라서 부트 시퀀스는 단순한 하드웨어 초기화 과정뿐만 아니라 전체 시스템 동작 동안 유지되는 보안 아키텍처(Security Architecture)를 확립하는 과정이 된다.

Cortex-M 시스템에 부트로더가 존재하는 경우 애플리케이션으로 제어권을 전달하는 작업은 일반적인 함수를 호출하는 것보다 복잡하다. 부트로더는 애플리케이션의 벡터 테이블을 식별하고, 애플리케이션의 초기 스택 포인터(Initial Stack Pointer)와 리셋 핸들러 주소를 가져온 다음, 전환에 필요한 프로세서 상태를 준비해야 한다. 또한 부트로더가 사용하던 인터럽트와 주변장치는 애플리케이션 실행 이후 예기치 않게 활성 상태로 남지 않도록 신중하게 처리해야 한다.

견고한 제어권 전달(Handoff)은 일반적으로 관련 인터럽트를 비활성화하거나 마스킹(Masking)하고, 불필요한 부트로더 서비스를 중지하며, 필요한 경우 대기 중인 인터럽트 조건(Pending Interrupt Condition)을 제거하고, 애플리케이션의 벡터 테이블 위치를 설정하는 과정을 포함한다. 이후 메인 스택 포인터(Main Stack Pointer)에 애플리케이션의 초기 스택 값을 로드하고 애플리케이션의 리셋 핸들러 또는 정의된 진입점(Entry Point)으로 실행을 분기한다. 애플리케이션의 관점에서 이렇게 구축된 환경은 해당 펌웨어 아키텍처가 가정하는 시작 조건과 최대한 일치해야 한다.

링커 구성(Linker Configuration)은 부트로더, 애플리케이션, 벡터 테이블, 실행 코드(Executable Code), 상수(Constant), 초기화 데이터(Initialized Data), 런타임 메모리 영역(Runtime Memory Region)의 위치를 정의하기 때문에 이 과정에서 핵심적인 역할을 한다. 부트로더가 플래시의 시작 부분을 사용한다면 애플리케이션은 일반적으로 다른 주소를 기준으로 링크되어야 한다. 따라서 부트로더와 애플리케이션은 동일한 물리적 주소를 독립적으로 사용한다고 가정하는 대신 명시적인 메모리 맵 계약(Memory-Map Contract)을 공유해야 한다.

ARM 애플리케이션 진입점(Application Entry Point)을 신뢰하기 전에 이미지 검증(Image Validation)을 수행해야 한다. 부트로더는 이미지 메타데이터(Image Metadata), 주소 범위(Address Range), 스택 포인터 유효성(Stack-Pointer Validity), 버전 정보(Version Information), 무결성 값(Integrity Value), 암호학적 인증 데이터(Cryptographic Authentication Data)를 검사할 수 있다. 잘못된 리셋 핸들러 주소나 유효하지 않은 스택 위치는 보다 심층적인 검증을 수행하기 전에도 이미지 손상을 나타낼 수 있다. 이러한 검사는 잘못된 이미지가 복구 가능한 부트 문제를 제어되지 않는 프로세서 실행으로 확대시키는 것을 방지한다.

전체 부트 시퀀스에서 워치독 동작(Watchdog Behavior)도 고려해야 한다. 부트 ROM, 안전 펌웨어(Safety Firmware), 또는 부트로더가 시작한 워치독은 후속 소프트웨어가 실행된 이후에도 활성 상태를 유지할 수 있다. 각 시작 단계는 정해진 시간 제약 내에서 워치독을 서비스하거나 그 책임을 다음 단계에 명시적으로 전달해야 한다. 로봇 컨트롤러(Robotic Controller)에서는 액추에이터 초기화 중 예상하지 못한 워치독 만료가 시스템을 반복적으로 리셋하고 정상적인 복구를 방해할 수 있으므로 이러한 소유권 전환(Ownership Transition)을 명확하게 정의해야 한다.

ARM 기반 로봇 시스템은 프로세서 부팅과 외부 전원 및 안전 시퀀싱(Safety Sequencing)을 함께 조정하는 경우가 많다. 메인 CPU는 모터 컨트롤러(Motor Controller), 센서, 통신 네트워크(Communication Network), 안전 프로세서(Safety Processor)가 아직 초기화되는 동안 먼저 시작될 수 있다. 따라서 소프트웨어는 프로세서 준비 상태(Processor Readiness)와 전체 시스템 준비 상태(System Readiness)를 구분해야 한다. 애플리케이션 진입점에 도달했다고 해서 액추에이터를 즉시 활성화할 수 있다는 의미는 아니며, 애플리케이션은 자체 검증을 완료하고 관련 하위 시스템과 안전한 통신을 구축해야 한다.

ARM 부트 실패(Boot Failure)를 디버깅하려면 실행이 정확히 어느 단계에서 중단되었는지를 파악해야 한다. 리셋 핸들러에 도달하기 전에 발생한 실패는 메모리 초기화, 벡터 재배치(Vector Relocation), DRAM 설정, 이미지 검증, 애플리케이션 제어권 전달 과정에서 발생한 오류와 서로 다른 원인을 가진다. SWD, JTAG, 직렬 추적(Serial Trace), 부트 상태 레지스터(Boot Status Register), 리셋 원인 레지스터(Reset-Cause Register), GPIO 표시 신호(GPIO Marker), 신중하게 배치된 진단 메시지(Diagnostic Message)를 이용하면 전체 애플리케이션 환경이 동작하지 않더라도 문제 분석에 필요한 증거를 확보할 수 있다.

따라서 전체 ARM 부트 시퀀스는 프로세서 상태를 단계적으로 변환하는 통제된 과정으로 이해할 수 있다. 하드웨어 리셋(Hardware Reset)은 아키텍처의 시작 상태를 설정하고, 리셋 벡터(Reset Vector)는 최초 소프트웨어 실행 위치를 선택하며, 시작 코드는 프로그래밍 언어와 메모리 실행 환경을 구축한다. 이후 부트로더 단계가 플랫폼을 초기화하고 검증하며, 최종 제어권 전달을 통해 애플리케이션의 실행 컨텍스트를 확립한다. 이러한 과정은 보안 부트(Secure Boot), 복구(Recovery), OTA 업데이트(OTA Update), 신뢰성 높은 로봇 시작(Reliable Robotic Startup)을 구현하기 위한 기반을 제공한다.

## 04.03 U-Boot Architecture and Customization [w/Code]

![](images/image3.png){width="7.268055555555556in" height="7.268055555555556in"}

U-Boot는 임베디드 시스템(Embedded System), 특히 임베디드 리눅스(Embedded Linux)를 실행하는 ARM 기반 SoC에서 널리 사용되는 오픈소스 부트로더(Open-Source Bootloader)이다. U-Boot는 저수준 프로세서 시작 과정(Low-Level Processor Startup)과 운영체제(Operating System) 사이의 전환 영역에서 동작하며, 하드웨어 초기화(Hardware Initialization), 이미지 로딩(Image Loading), 구성(Configuration), 진단(Diagnostics), 복구(Recovery) 기능을 제공한다. 로봇 시스템에서 U-Boot는 리눅스와 로봇 소프트웨어 서비스가 실행되기 전에 임베디드 컴퓨팅 보드(Embedded Computing Board)를 준비하는 플랫폼 수준의 제어 지점(Platform-Level Control Point)이 된다.

일반적인 U-Boot 부트 경로(Boot Path)는 프로세서가 초기에는 제한된 메모리와 하드웨어 자원에만 접근할 수 있기 때문에 여러 단계로 구분된다. 일반적으로 부트 ROM(Boot ROM)이 먼저 실행되고 작은 초기 U-Boot 구성요소를 내부 SRAM에 로드한다. 이 초기 단계는 DRAM과 기타 필수 자원을 활성화하는 데 필요한 최소한의 초기화를 수행한다. 외부 메모리를 사용할 수 있게 되면 훨씬 풍부한 기능을 갖춘 더 큰 U-Boot 이미지가 실행될 수 있다.

많은 플랫폼에서 작은 초기 구성요소를 SPL(Secondary Program Loader)이라고 한다. 일부 시스템에서는 내부 SRAM의 용량이 매우 제한되어 초기화 과정을 더 세분화해야 하는 경우 추가적인 TPL(Tertiary Program Loader) 단계를 사용한다. SPL은 일반적으로 클록(Clock), 핀 멀티플렉싱(Pin Multiplexing), DRAM, 그리고 전체 U-Boot 이미지를 찾는 데 필요한 저장장치 인터페이스(Storage Interface)를 초기화한다. 이 단계를 작게 유지하면 메모리 요구량을 줄이면서 필수 시작 로직과 기능이 풍부한 부트 서비스를 분리할 수 있다.

전체 U-Boot 단계(Full U-Boot Stage)는 명령 환경(Command Environment)과 저장장치(Storage), 네트워킹(Networking), 파일시스템(Filesystem), 장치 검색(Device Discovery), 이미지 처리(Image Handling), 환경 변수(Environment Variable), 운영체제 시작을 위한 재사용 가능한 서브시스템(Subsystem)을 제공한다. 모든 보드 동작을 하나의 거대한 루틴에 포함하는 대신 U-Boot는 드라이버(Driver), 명령(Command), 구성 옵션(Configuration Option), 아키텍처별 코드(Architecture-Specific Code)를 통해 기능을 구성한다. 이러한 모듈형 구조(Modular Organization)를 통해 동일한 프레임워크가 다양한 프로세서와 임베디드 보드를 지원할 수 있다.

현대의 U-Boot는 구조화된 클래스와 드라이버를 통해 하드웨어 장치를 표현하기 위해 드라이버 모델(Driver Model)을 점점 더 적극적으로 사용한다. GPIO 컨트롤러, 직렬 포트(Serial Port), I2C 버스, SPI 컨트롤러, 이더넷 인터페이스(Ethernet Interface), MMC 장치, 저장장치 컨트롤러(Storage Controller) 등을 공통 인터페이스를 통해 검색하고 초기화할 수 있다. 장치 트리(Device Tree) 정보는 하드웨어 설명을 실행 코드의 상당 부분과 분리하여 전체 부트로더 구현을 복제하지 않고도 보드 변형(Board Variation)을 보다 쉽게 유지할 수 있도록 한다.

보드 사용자 정의(Board Customization)는 일반적으로 대상 프로세서와 하드웨어 플랫폼에 가장 가까운 기존 구성을 기반으로 시작한다. 이후 개발자는 구성 옵션, 장치 트리 정의(Device Tree Definition), 보드 초기화 코드(Board Initialization Code), 저장장치 레이아웃(Storage Layout), 부트 명령(Boot Command), 환경 설정(Environment Setting)을 조정한다. 이러한 접근 방식은 완전히 독립적인 U-Boot 포트를 새로 만드는 것보다 바람직한데, 공급업체 및 업스트림(Upstream) 구현에 이미 포함된 프로세서 초기화, 주변장치 드라이버, 아키텍처 규칙을 재사용할 수 있기 때문이다.

U-Boot 빌드 시스템(Build System)은 아키텍처와 보드 구성 정보를 사용하여 최종 바이너리에 어떤 기능을 포함할지 결정한다. defconfig 파일은 기본 구성(Baseline Configuration)을 제공하며, Kconfig 옵션은 개별 서브시스템과 기능을 제어한다. 개발자는 제품에 필요한 드라이버, 명령, 파일시스템, 보안 기능(Security Function), 네트워킹 기능만 활성화할 수 있으며, 이를 통해 플래시 크기, RAM 사용량, 부트 시간(Boot Time), 유지보수 복잡성(Maintenance Complexity), 공격 표면(Attack Surface) 사이의 균형을 조절할 수 있다.

장치 트리 사용자 정의(Device Tree Customization)는 리눅스가 시작되기 전에 U-Boot가 보드별 하드웨어를 이해해야 하는 경우 특히 중요하다. 노드(Node)는 메모리, UART, GPIO 신호, 레귤레이터(Regulator), 이더넷 인터페이스, 저장장치 컨트롤러, 버스(Bus), 연결된 주변장치를 설명할 수 있다. U-Boot는 초기화 과정에서 자체 장치 트리를 사용하고 이후 장치 트리 블롭(Device Tree Blob, DTB)을 리눅스 커널에 전달할 수 있다. 따라서 부트로더 수준의 하드웨어 준비와 커널 수준의 장치 관리(Device Management) 사이의 일관성을 유지하려면 올바른 구성이 필요하다.

환경 변수(Environment Variable)는 전체 부트로더를 다시 빌드하지 않고 U-Boot 동작을 제어할 수 있는 유연한 메커니즘을 제공한다. 변수는 콘솔 매개변수(Console Parameter), 커널 명령줄(Kernel Command Line), 이미지 위치(Image Location), 네트워크 주소(Network Address), 파티션 식별자(Partition Identifier), 부트 대상(Boot Target), 스크립트(Script)를 정의할 수 있다. bootcmd 변수는 일반적으로 부트 지연(Boot Delay) 이후 실행되는 기본 시퀀스를 결정한다. 적절하게 설계된 환경을 사용하면 실행 가능한 부트로더 코드를 비교적 안정적으로 유지하면서 제품의 배포 또는 복구 동작을 변경할 수 있다.

부트 스크립트(Boot Script)는 U-Boot 명령을 결합하여 반복 가능한 시작 로직(Startup Logic)을 구성할 수 있게 한다. 스크립트는 저장장치를 검사하고, 파티션을 선택하고, 커널 이미지를 로드하고, 장치 트리를 로드하며, 부트 인자(Boot Argument)를 준비한 후 리눅스를 실행할 수 있다. 보다 발전된 로직은 실행할 이미지를 결정하기 전에 업데이트 상태(Update Status), 부트 카운터(Boot Counter), 하드웨어 변형(Hardware Variant), 복구 플래그(Recovery Flag)를 확인할 수 있다. 로봇 제품에서는 이를 통해 시작 정책(Startup Policy)을 명확하게 정의하고 정상, 유지보수, 공장 초기화, 복구 부트 경로가 공통 프레임워크를 공유할 수 있다.

U-Boot는 MMC, eMMC, SD 카드, NAND, NOR, USB, SATA, NVMe, 이더넷(Ethernet), 그리고 대상 구성에서 지원하는 네트워크 프로토콜(Network Protocol)을 포함한 다양한 저장 및 통신 메커니즘에서 소프트웨어를 로드할 수 있다. 파일시스템 지원을 통해 FAT, ext 계열 파일시스템 또는 기타 지원 형식에서 이미지를 로드할 수 있다. 이러한 유연성은 개발 과정에서는 매우 유용하지만, 제품 환경에서는 복잡성과 승인되지 않은 실행 경로(Unauthorized Execution Path)를 줄이기 위해 불필요한 부트 소스를 비활성화하는 것이 일반적으로 바람직하다.

리눅스 시작을 위해서는 U-Boot가 제어권을 전달하기 전에 여러 정보를 준비해야 한다. 플랫폼에 따라 커널(Kernel), 장치 트리 블롭(Device Tree Blob), 그리고 선택적으로 초기 RAM 파일시스템(Initial RAM Filesystem, initramfs)을 적절한 메모리 위치에 로드한다. 또한 커널 명령줄을 구성하고 대상 하드웨어에 관한 정보를 제공한다. 커널, 장치 트리, initramfs, 압축 해제 작업 영역(Decompression Workspace), U-Boot 런타임 데이터가 서로 덮어쓰지 않도록 메모리 주소를 신중하게 선택해야 한다.

U-Boot는 일반적인 커널 이미지뿐만 아니라 FIT(Flattened Image Tree)와 같은 구조화된 형식을 포함한 다양한 이미지 표현 방식을 지원한다. FIT는 여러 구성요소를 하나의 구조로 패키징하거나 설명하고 이를 해시(Hash) 또는 암호학적 서명(Cryptographic Signature)과 연결할 수 있다. 이는 제품이 커널, 장치 트리 변형(Device Tree Variant), 펌웨어 구성요소(Firmware Component), 구성 대안(Configuration Alternative)을 하나의 조정된 부트 산출물(Boot Artifact)로 관리해야 할 때 유용하다. 이미지 메타데이터(Image Metadata)를 이용하면 부트로더가 특정 하드웨어 구성에 적합한 소프트웨어를 선택할 수도 있다.

보안 제품(Secure Product)을 위한 사용자 정의는 단순히 올바른 이미지를 로드하는 것 이상의 작업을 요구한다. U-Boot는 이전 단계의 신뢰 펌웨어(Trusted Firmware)가 U-Boot를 인증하고, 다시 U-Boot가 자신이 로드하는 운영체제 산출물을 검증하는 검증된 부트 체인(Verified Boot Chain)에 참여할 수 있다. 공개 키(Public Key), 서명된 이미지(Signed Image), 보호된 구성(Protected Configuration), 제어된 부트 명령을 일관되게 통합해야 한다. 인증된 이미지 경로와 함께 제한되지 않은 콘솔이나 검증되지 않은 대체 부트 메커니즘이 존재하면 전체 보안성이 약화될 수 있다.

대화형 U-Boot 콘솔(Interactive U-Boot Console)은 엔지니어가 메모리를 검사하고, 저장장치를 시험하고, 네트워크 인터페이스를 구성하고, 장치 트리 정보를 확인하며, 이미지를 수동으로 로드할 수 있기 때문에 보드 초기 개발(Board Bring-Up) 과정에서 매우 유용하다. 그러나 동일한 기능은 배포된 로봇에서는 보안 문제가 될 수 있다. 따라서 제품용 사용자 정의에서는 콘솔 접근이 필요한지, 자동 부트 중단을 어떻게 제어할지, 어떤 명령을 허용할지, 환경 변경(Environment Modification)을 제한하거나 인증해야 하는지를 결정해야 한다.

U-Boot는 복구(Recovery) 및 OTA 업데이트 아키텍처(OTA Update Architecture)를 구현하기 위한 효과적인 기반도 제공한다. 부트 변수 또는 영구 메타데이터(Persistent Metadata)를 이용하여 활성 소프트웨어 슬롯(Active Software Slot)과 후보 슬롯(Candidate Slot)을 식별하고, 실패한 부트 횟수를 계산하며, 정상 시작이 반복적으로 실패하는 경우 복구 이미지를 선택할 수 있다. A/B 설계에서는 이전에 정상적으로 동작하던 소프트웨어를 유지하면서 두 시스템 파티션 가운데 하나를 선택할 수 있다. 이후 애플리케이션이 성공적으로 시작되면 후보 이미지를 확정하여 업데이트 트랜잭션(Update Transaction)을 완료할 수 있다.

로봇 플랫폼에서는 U-Boot와 실제 물리 시스템 동작 사이의 경계를 추가로 고려해야 한다. U-Boot가 GPIO 핀, 통신 컨트롤러, 레귤레이터 또는 액추에이터와 간접적으로 연결된 다른 인터페이스를 구성할 수 있기 때문이다. 따라서 보드 초기화 과정에서는 의도하지 않은 모터 활성화 신호(Motor Enable Signal)나 위험한 주변장치 상태가 발생하지 않도록 해야 한다. 상위 소프트웨어가 시스템 준비 상태를 확인할 때까지 안전 관련 출력은 전용 하드웨어 또는 명시적으로 정의된 안전 상태(Safe State)에 의해 제어되어야 한다.

사용자 정의된 U-Boot 구현을 디버깅(Debugging)할 때는 부트 경로를 관찰 가능한 단계로 구분하는 것이 효과적이다. 직렬 콘솔 출력(Serial Console Output)을 통해 SPL 실행, DRAM 초기화, 재배치(Relocation), 저장장치 접근, 장치 트리 처리, 이미지 로딩, 커널 제어권 전달 과정에서 발생한 실패를 식별할 수 있다. 콘솔 출력을 사용할 수 없는 경우 JTAG, 메모리 검사(Memory Inspection), GPIO 마커(GPIO Marker), 부트 타이밍 측정(Boot Timing Measurement)을 이용하여 추가적인 가시성을 확보할 수 있다. 복잡한 부트 스크립트나 운영체제 동작을 추가하기 전에 초기 단계의 실패부터 격리하여 해결해야 한다.

따라서 유지보수 가능한 U-Boot 아키텍처는 사용자 정의를 부트로더 소스 코드의 무제한 변경이 아니라 통제된 플랫폼 통합(Controlled Platform Integration) 과정으로 다루어야 한다. 프로세서별 초기화, SPL, 드라이버 모델(Driver Model), 장치 트리(Device Tree), Kconfig, 환경 변수(Environment Variable), 부트 스크립트(Boot Script), 이미지 검증(Image Verification), 복구 정책(Recovery Policy), 리눅스 제어권 전달(Linux Handoff)이 하나의 일관된 시작 설계(Startup Design)를 형성해야 한다. 이러한 아키텍처는 로봇 시스템에서 보안 부트(Secure Boot), A/B 업데이트(A/B Update), 복구 메커니즘(Recovery Mechanism), 부트 최적화(Boot Optimization), 신뢰성 높은 임베디드 리눅스 동작(Reliable Embedded Linux Operation)을 구현하기 위한 기반을 제공한다.

## 04.04 Secure Boot: RSA Signature Verification, HAB, TrustZone

![](images/image4.png){width="7.268055555555556in" height="7.268055555555556in"}

보안 부트(Secure Boot)는 변경할 수 없는 하드웨어 제어 시작점(Hardware-Controlled Starting Point)에서 시작하여 최종적으로 임베디드 시스템을 운영하는 소프트웨어까지 암호학적 신뢰 체인(Cryptographic Chain of Trust)을 구축한다. 플래시(Flash), eMMC 또는 다른 부트 매체(Boot Medium)에 저장된 코드가 신뢰할 수 있다고 단순히 가정하는 대신, 각 부트 단계(Boot Stage)는 다음 단계의 소프트웨어를 실행하기 전에 진위성(Authenticity)과 무결성(Integrity)을 검증한다. 로봇 플랫폼에서는 이를 통해 승인되지 않은 펌웨어가 시작 과정에서 센서, 네트워크, 안전 기능 및 액추에이터(Actuator)를 제어하는 것을 방지할 수 있다.

보안 부트의 기반은 신뢰 루트(Root of Trust)이며, 일반 소프트웨어가 자유롭게 변경할 수 없는 구성요소에 존재해야 한다. 프로세서에 따라 변경 불가능한 부트 ROM(Boot ROM), 일회성 프로그래밍 퓨즈(One-Time Programmable Fuse), 보안 키 저장소(Secure Key Storage), 하드웨어 보안 로직(Hardware Security Logic) 등이 사용될 수 있다. 신뢰 루트는 승인된 소프트웨어를 식별하는 데 필요한 정보를 포함하거나 생성하며, 이를 통해 프로세서는 잠재적으로 신뢰할 수 없는 코드가 실행되기 전에 최초의 보안 결정을 내릴 수 있다.

디지털 서명(Digital Signature)은 소프트웨어의 진위성을 단순한 손상 검출과 구분한다. 체크섬(Checksum)이나 CRC는 우발적인 변경을 검출할 수 있지만, 일반적으로 공격자는 이미지를 수정한 후 이를 다시 계산할 수 있다. 보안 부트에서는 대신 비대칭 암호화(Asymmetric Cryptography)를 사용하며, 통제된 소프트웨어 릴리스 과정에서 개인 키(Private Key)를 사용해 이미지에 서명하고 임베디드 장치는 이에 대응하는 신뢰된 공개 키(Public Key) 정보를 사용하여 이미지를 승인하기 전에 서명을 검증한다.

RSA는 보안 부트 구현에서 널리 사용되는 대표적인 공개 키 알고리즘(Public-Key Algorithm)이다. 이미지를 준비하는 과정에서는 펌웨어 또는 부트 이미지의 정의된 영역에 대해 SHA-256과 같은 암호학적 해시(Cryptographic Hash)를 계산한다. 서명 시스템(Signing System)은 보호된 RSA 개인 키를 이용하여 해당 다이제스트(Digest)에 연결된 디지털 서명을 생성한다. 대상 장치는 개인 키를 보유할 필요가 없으며 서명을 검증하는 데 필요한 신뢰된 공개 키 정보만 보유하면 된다.

검증 과정은 수신되거나 저장된 이미지에 대해 동일하게 정의된 해싱 절차(Hashing Procedure)를 사용하여 새로운 해시를 계산하면서 시작된다. 이후 부트 검증 로직(Boot Verification Logic)은 신뢰된 공개 키를 사용하여 RSA 서명을 평가하고 서명된 다이제스트가 새롭게 계산된 다이제스트와 일치하는지를 확인한다. 선택된 서명 방식(Signature Scheme)에 따라 두 값이 일치하면 이미지의 무결성과 진위성이 모두 확인된 것이다. 검증에 실패하면 정상적인 신뢰 부트 경로(Trusted Boot Path)를 통해 실행이 계속되어서는 안 된다.

따라서 키 관리(Key Management)는 RSA 계산 자체만큼 중요하다. 공격자가 서명용 개인 키(Signing Private Key)를 획득하면 악성 소프트웨어에 정상적인 서명을 생성하여 배포된 장치가 이를 합법적인 소프트웨어로 인식하도록 만들 수 있다. 따라서 개인 키는 통제된 서명 인프라(Signing Infrastructure)에서 보호해야 하며, 필요한 경우 하드웨어 기반 키 보호(Hardware-Backed Key Protection)를 사용하는 것이 바람직하다. 또한 개발 키(Development Key), 생산 키(Production Key), 복구 키(Recovery Key), 키 교체 정책(Key-Rotation Policy)을 제품 보안 수명주기(Security Lifecycle)에 따라 분리해야 한다.

신뢰된 공개 키(Trusted Public Key) 역시 장치에 안전하게 고정되어야 한다. 일부 프로세서는 완전한 공개 키 자체를 퓨즈에 저장하는 대신 승인된 공개 키의 해시(Hash)를 일회성 프로그래밍 메모리(One-Time Programmable Memory)에 저장한다. 부팅 과정에서는 제공된 공개 키의 해시를 계산하여 퓨즈에 저장된 신뢰 값과 비교한 후 서명 검증에 사용한다. 이러한 방식은 제조된 장치와 해당 장치의 소프트웨어를 제어하도록 승인된 소프트웨어 서명 기관(Software-Signing Authority) 사이에 간결한 하드웨어 기반 연결을 구축한다.

신뢰 체인(Chain of Trust)은 최초로 외부에서 로드되는 이미지 이후의 단계까지 검증을 확장한다. 변경 불가능한 부트 ROM은 1단계 부트로더(First-Stage Bootloader)를 인증하고, 1단계 부트로더는 2단계 부트로더(Second-Stage Bootloader)를 인증하며, 이후 커널(Kernel), 장치 트리(Device Tree), initramfs, 펌웨어 패키지(Firmware Package) 또는 다른 실행 구성요소를 검증할 수 있다. 검증된 각 단계는 이전 단계에서 확립된 신뢰를 유지할 책임을 가진다. 단 하나의 검증되지 않은 실행 경로만 존재하더라도 전체 부트 시퀀스의 보안 가정을 무너뜨릴 수 있다.

고신뢰 부트(High Assurance Boot), 즉 NXP i.MX 플랫폼과 주로 연관되는 HAB는 부트 이미지 인증을 위한 하드웨어 지원 프레임워크(Hardware-Supported Framework)를 제공한다. 프로세서의 부트 ROM은 장치에 프로비저닝(Provisioning)된 보안 구성과 신뢰 키 정보를 이용하여 검증 과정에 참여한다. 서명된 부트 이미지에는 인증에 필요한 구조가 포함되며, 이를 통해 ROM 수준의 코드가 외부에서 변경 가능한 메모리에 저장된 소프트웨어로 실행을 전달하기 전에 해당 이미지가 승인된 것인지 판단할 수 있다.

HAB 기반 설계(HAB-Based Design)는 일반적으로 이미지 생성(Image Generation), 서명(Signing), 키 프로비저닝(Key Provisioning), 장치 구성(Device Configuration), 부트 시점 인증(Boot-Time Authentication)을 조정된 제조 및 릴리스 프로세스로 통합한다. 특히 보안 퓨즈(Security Fuse) 프로비저닝은 많은 퓨즈 작업이 되돌릴 수 없기 때문에 매우 신중하게 수행해야 한다. 따라서 개발 시스템에서는 생산용 보안 구성을 영구적으로 적용하기 전에 전체 서명 및 인증 흐름을 검증해야 한다. 잘못된 프로비저닝은 정상적으로 동작하는 하드웨어조차 승인된 소프트웨어로 부팅하지 못하게 만들 수 있다.

보안 부트는 인증 실패(Authentication Failure)가 발생했을 때의 동작도 정의해야 한다. 이미지를 단순히 거부하는 것은 필요하지만 실제 로봇 운용 환경에서는 그것만으로 충분하지 않을 수 있다. 플랫폼은 제한된 복구 모드(Restricted Recovery Mode)로 진입하거나, 이전에 검증된 A/B 파티션(A/B Partition)을 시도하거나, 신뢰된 복구 이미지(Trusted Recovery Image)를 로드하거나, 안전한 비운용 상태(Safe Non-Operational State)를 유지할 수 있다. 복구 소프트웨어 역시 인증된 신뢰 체인에 포함되어야 하며, 그렇지 않으면 공격자가 의도적으로 인증 실패를 발생시킨 후 보호되지 않은 복구 경로를 악용할 수 있다.

ARM TrustZone은 인증된 소프트웨어가 실행되기 시작한 이후 하드웨어 기반 분리(Hardware-Supported Separation)를 제공함으로써 보안 부트를 보완한다. TrustZone은 ARM 아키텍처에 따라 시스템 자원을 보안 영역(Secure World)과 비보안 영역(Non-secure World) 또는 보안 상태(Security State)로 구분한다. 보안 펌웨어(Secure Firmware)는 암호 연산, 키 정보, 보안 구성, 특정 주변장치를 보호하면서 범용 운영체제와 애플리케이션은 보다 제한된 접근 권한을 가진 비보안 환경에서 실행되도록 할 수 있다.

보안 부트와 TrustZone은 서로 연관되어 있지만 서로 다른 문제를 해결한다. 보안 부트는 시스템 시작 과정에서 소프트웨어가 실행 권한을 가진 소프트웨어인지를 판단하는 반면, TrustZone은 실행이 시작된 이후 해당 소프트웨어가 어떤 자원에 접근할 수 있는지를 제어하는 데 도움을 준다. 올바르게 서명된 운영체제라고 해서 모든 보안 민감 자원(Security-Sensitive Resource)에 무제한 접근할 필요는 없다. 보안 영역 펌웨어는 키, 신뢰 서비스(Trusted Service), 보안 저장소(Secure Storage), 중요 구성의 소유권을 유지하면서 비보안 소프트웨어에는 통제된 인터페이스만 제공할 수 있다.

TrustZone 구성은 신뢰할 수 없는 소프트웨어가 보호 자원에 일시적으로라도 접근하지 못하도록 충분히 이른 시점에 수행되어야 한다. 메모리 영역(Memory Region), 주변장치 보안 속성(Peripheral Security Attribute), 인터럽트 라우팅(Interrupt Routing), 실행 상태(Execution State)는 비보안 운영체제로 제어권을 전달하기 전에 구성해야 한다. Cortex-A 시스템에서는 일반적으로 신뢰 펌웨어(Trusted Firmware)가 이러한 경계를 설정하고 보안 상태 사이의 전환을 관리하며, Cortex-M TrustZone 구현에서도 마이크로컨트롤러급 시스템에 적합한 관련 격리 메커니즘(Isolation Mechanism)을 제공한다.

롤백 방지(Anti-Rollback Protection)는 보안 부트의 또 다른 중요한 확장 기능이다. 올바르게 서명된 이미지라도 과거의 취약점을 포함할 수 있으므로 서명 검증만으로 해당 소프트웨어 버전을 실행해도 안전하다고 보장할 수 없다. 보안 버전 카운터(Secure Version Counter), 단조 증가 값(Monotonic Value), 보호된 롤백 인덱스(Protected Rollback Index)를 이용하면 최소 승인 보안 버전보다 오래된 소프트웨어의 설치 또는 실행을 차단할 수 있다. 이를 통해 전체 서명 아키텍처를 폐기하지 않고도 취약한 과거 릴리스를 무효화할 수 있다.

보안 OTA 업데이트(Secure OTA Update)는 최초 부팅에서 사용한 것과 동일한 신뢰 원칙을 유지해야 한다. 업데이트 패키지(Update Package)는 활성화 전에 인증되어야 하고 통제된 저장장치에 기록되어야 하며, 필요에 따라 실행 전에 다시 검증되어야 한다. A/B 아키텍처에서는 후보 소프트웨어(Candidate Software)가 성공적으로 부팅될 때까지 기존의 신뢰된 이미지를 유지할 수 있다. 서명 검증, 롤백 방지, 부트 카운터(Boot Counter), 복구 정책(Recovery Policy)을 함께 사용하면 업데이트 실패가 보안 우회(Security Bypass)나 영구적인 가용성 장애(Availability Failure)로 이어지는 것을 방지할 수 있다.

로봇 시스템에서는 손상된 시작 소프트웨어가 실제 움직임과 주변 환경과의 물리적 상호작용에 영향을 줄 수 있으므로 보안 부트 설계가 물리적 결과(Physical Consequence)와 직접 연결된다. 인증이 완료되지 않은 동안 액추에이터 활성화 신호(Actuator-Enable Signal)는 안전 상태(Safe State)를 유지해야 하며, 부트 검증 실패가 브레이크 해제, 모터 구동 또는 독립 안전 컨트롤러(Independent Safety Controller)의 우회로 이어져서는 안 된다. 따라서 보안 실패 처리(Security Failure Handling)는 로봇의 전원 시퀀싱(Power Sequencing) 및 기능 안전 아키텍처(Functional Safety Architecture)와 조정되어야 한다.

디버그 인터페이스(Debug Interface)와 개발 기능(Development Feature)도 동일하게 주의해야 한다. JTAG, SWD, 직렬 다운로드 모드(Serial Download Mode), USB 복구, 제한되지 않은 U-Boot 콘솔, 대체 부트 소스(Alternative Boot Source)는 강력한 검증 부트 체인을 우회하는 경로가 될 수 있다. 생산 장치(Production Device)는 이러한 메커니즘에 대한 명시적인 정책을 설정하고 서비스 요구사항에 따라 비활성화, 제한, 인증 또는 영구 구성을 적용해야 한다. 보안 부트는 모든 대체 실행 경로가 일관되게 통제될 때만 효과적으로 동작한다.

따라서 완전한 보안 부트 아키텍처(Secure Boot Architecture)는 하드웨어 신뢰 루트(Hardware Root of Trust), 보호된 키 프로비저닝(Protected Key Provisioning), 암호학적 해싱(Cryptographic Hashing), RSA 서명 검증(RSA Signature Verification), 인증된 부트 단계(Authenticated Boot Stage), 통제된 복구(Controlled Recovery), 롤백 방지(Rollback Protection), 런타임 격리(Runtime Isolation)를 결합한다. HAB와 같은 기술은 초기 부트 과정에서 신뢰를 확립하고, ARM TrustZone은 시작 이후에도 보안 경계를 유지할 수 있도록 한다. 이러한 메커니즘의 결합은 단순한 소프트웨어 로딩 과정이었던 부트를 신뢰할 수 있는 하드웨어에서 인증되고 격리된 로봇 소프트웨어 실행으로 이어지는 통제된 전환 과정으로 발전시킨다.

## 04.05 Dual Partition A/B Boot Design for OTA

![](images/image5.png){width="7.268055555555556in" height="7.268055555555556in"}

이중 파티션 A/B 부트 아키텍처(Dual-Partition A/B Boot Architecture)는 무선 업데이트(Over-the-Air, OTA) 과정에서 발생할 수 있는 중단이나 소프트웨어 장애에도 안정적으로 대응할 수 있도록 설계된 신뢰성 메커니즘(Reliability Mechanism)이다. 현재 실행 중인 시스템을 직접 덮어쓰는 대신 장치는 일반적으로 슬롯 A(Slot A)와 슬롯 B(Slot B)라고 하는 두 개의 독립적으로 부팅 가능한 소프트웨어 슬롯(Software Slot)을 유지한다. 하나의 슬롯이 활성 상태를 유지하는 동안 다른 슬롯에 후보 업데이트(Candidate Update)를 설치함으로써 업데이트 과정에서도 정상 부팅 가능한 시스템을 보존한다.

핵심 원리는 현재 실행 중인 이미지(Running Image)와 수정되는 이미지를 분리하는 것이다. 슬롯 A가 현재 활성 상태라면 OTA 서비스는 슬롯 A를 덮어쓰지 않고 비활성 슬롯 B(Inactive Slot B)에 새로운 소프트웨어를 다운로드하고 설치한다. 따라서 전원 손실, 네트워크 중단, 저장장치 오류 또는 불완전한 설치가 발생하더라도 기존의 정상 동작 이미지를 유지할 수 있다. 후보 슬롯이 완전히 기록되고 검증된 이후에만 시스템은 다음 부트 대상(Boot Target)을 변경하는 것을 고려한다.

A/B 저장장치 레이아웃(Storage Layout)은 시스템 요구사항에 따라 커널(Kernel), 루트 파일시스템(Root Filesystem), 장치 트리(Device Tree), 부트 펌웨어(Boot Firmware) 또는 기타 플랫폼 구성요소를 복제할 수 있다. 일부 설계에서는 두 슬롯 모두에 완전한 시스템 이미지(System Image)를 유지하는 반면, 다른 설계에서는 원자적으로 업데이트되어야 하는 구성요소만 복제한다. 공유 데이터 파티션(Shared Data Partition)에는 슬롯 전환 이후에도 유지되어야 하는 구성, 로그, 로봇 맵(Robot Map), 보정 데이터(Calibration Data), 영구 애플리케이션 정보(Persistent Application Information)를 저장할 수 있다.

파티션 설계(Partition Design)에서는 변경되지 않는 소프트웨어 산출물(Immutable Software Artifact)과 영구 상태(Persistent State)를 신중하게 구분해야 한다. 애플리케이션 데이터가 슬롯별 루트 파일시스템 내부에 직접 포함되어 있으면 슬롯을 전환할 때 이전 구성으로 되돌아가거나 새롭게 생성된 정보가 손실될 수 있다. 따라서 영구적인 로봇 식별 정보(Robot Identity), 보정 정보, 업데이트 메타데이터(Update Metadata), 운영 데이터는 이전 버전과 새로운 버전 모두와의 호환성을 고려하여 명시적으로 관리되는 저장 영역에 배치해야 한다.

OTA 처리는 일반적으로 인증된 업데이트 패키지(Authenticated Update Package)를 획득하고 해당 패키지가 대상 하드웨어 및 현재 소프트웨어 상태에 적합한지를 판단하면서 시작한다. 업데이터(Updater)는 제품 식별자(Product Identifier), 하드웨어 개정판(Hardware Revision), 소프트웨어 버전, 암호학적 서명(Cryptographic Signature), 해시(Hash), 패키지 의존성(Package Dependency), 필요한 저장공간을 검증할 수 있다. 호환되지 않거나 손상된 이미지가 다음 부트 후보가 되지 않도록 이러한 검사는 활성화 이전에 수행되어야 한다.

비활성 슬롯(Inactive Slot)에 이미지를 기록하는 작업은 업데이트가 성공했다는 증거가 아니라 트랜잭션 작업(Transactional Operation)으로 처리해야 한다. 개별 파티션이나 이미지 구성요소를 기록하고 저장장치에 동기화한 후 다시 검증할 수 있다. 모든 필수 구성요소가 무결성 검사(Integrity Check)를 통과하기 전까지 새로운 슬롯을 부팅 가능 상태로 표시해서는 안 된다. 업데이트 메타데이터는 슬롯이 비어 있음, 불완전함, 후보 상태, 활성 상태, 확인됨, 무효 상태 또는 기타 부트 선택 가능 상태인지를 정확하게 나타내야 한다.

설치가 성공적으로 완료되면 다음 재부팅 과정에서 후보 슬롯(Candidate Slot)을 시도하도록 부트로더(Bootloader)에 지시한다. 이러한 결정은 일반적으로 부트로더 환경 변수(Environment Variable), 전용 메타데이터, 부트 제어 구조(Boot-Control Structure), 또는 플랫폼별 영구 상태를 통해 표현된다. 부트로더가 새로운 커널로 성공적으로 제어권을 전달했다는 이유만으로 활성 슬롯을 영구적으로 교체해서는 안 된다. 애플리케이션 진입점(Application Entry Point)에 도달하는 것은 업데이트의 실제 동작 유효성을 판단하는 과정의 시작에 불과하다.

따라서 시험 부트(Trial Boot)는 부팅 가능한 소프트웨어(Bootable Software)와 확인된 소프트웨어(Confirmed Software)를 구분한다. 후보 이미지는 암호학적으로 유효하고 구조적으로 완전하더라도 드라이버 비호환성(Driver Incompatibility), 서비스 시작 오류(Service Startup Error), 구성 마이그레이션 문제(Configuration Migration Problem), 자원 부족(Resource Exhaustion), 애플리케이션 장애로 인해 정상 동작하지 않을 수 있다. 시험 기간 동안 부트로더 또는 업데이트 프레임워크는 후보 이미지가 아직 성공적인 시스템 동작을 입증하지 못했다는 사실을 판단할 수 있는 상태 정보를 유지한다.

부트 카운터(Boot Counter)는 결함이 있는 후보 이미지를 반복적으로 시작하는 것을 제한하기 위한 실용적인 메커니즘을 제공한다. 시험 슬롯을 부팅하기 전에 영구 메타데이터에 시도 횟수를 기록하거나 남은 부트 시도 횟수(Remaining-Attempt Counter)를 감소시킬 수 있다. 운영체제가 확인 조건에 도달하지 못하고 반복적으로 리셋되면 부트로더는 결국 해당 슬롯의 선택을 중단한다. 이후 이전에 확인된 슬롯(Previously Confirmed Slot)을 자동으로 복원하여 현장 로봇에 물리적으로 접근하지 않고도 시스템을 복구할 수 있다.

확인(Confirmation)은 단순히 커널 시작 후 일정 시간이 경과했다는 의미가 아니라 실질적인 시스템 상태(System Health)를 나타내야 한다. 로봇 플랫폼에서는 새로운 이미지를 정상 상태로 선언하기 전에 중요 드라이버, 저장장치, 네트워크 인터페이스, 위치추정 구성요소(Localization Component), 안전 컨트롤러(Safety Controller), 필수 애플리케이션 서비스가 성공적으로 초기화되어야 할 수 있다. 확인 정책(Confirmation Policy)은 사용할 수 없는 업데이트를 감지할 만큼 엄격해야 하지만 선택적 서비스에 대한 불필요한 의존성 때문에 롤백이 발생하지 않도록 설계해야 한다.

후보 이미지가 정의된 상태 기준(Health Criteria)을 충족하면 소프트웨어는 영구 부트 제어 상태(Persistent Boot-Control State)에 성공적인 부트 확인을 기록한다. 이후 후보 슬롯은 새롭게 확인된 활성 슬롯(Confirmed Active Slot)이 되고, 이전 슬롯은 일정 기간 동안 또는 다음 업데이트가 수행될 때까지 폴백(Fallback) 대상으로 유지될 수 있다. 이러한 명시적인 커밋 작업(Commit Operation)을 통해 OTA 트랜잭션이 완료되며, 이후 발생하는 관련 없는 리셋이 업데이트 실패로 잘못 해석되는 것을 방지한다.

롤백(Rollback)은 후보 이미지를 확인할 수 없을 때 서비스를 복원하는 상호보완적인 동작이다. 부트로더는 이전의 정상 확인 슬롯(Previous Known-Good Slot)을 선택하고 후보 슬롯의 실패 상태를 기록한 후 신뢰된 소프트웨어를 사용하여 동작을 재개한다. 롤백 정책(Rollback Policy)은 시스템이 두 개의 결함 상태 사이를 끝없이 반복하는 상황을 방지해야 한다. 따라서 실패한 슬롯, 부트 시도 횟수, 복구 조건(Recovery Condition), 운영자 개입 요구사항(Operator Intervention Requirement)을 명확하게 표현해야 한다.

A/B 부트 아키텍처는 독립적인 신뢰성 메커니즘으로 동작하는 것이 아니라 보안 부트(Secure Boot)와 통합되어야 한다. 슬롯 A와 슬롯 B 모두 동일한 인증 정책(Authentication Policy)을 적용받아야 하며, 파티션 전환이 서명 검증(Signature Verification)을 우회하는 경로를 제공해서는 안 된다. 부트로더는 선택된 이미지를 실행하기 전에 인증해야 하며, 롤백 방지 규칙(Anti-Rollback Rule)은 올바르게 서명되었지만 오래된 버전이 현재 보안 정책에서 여전히 허용되는지를 결정해야 한다.

OTA 업데이트는 플래시나 eMMC에 대규모 데이터를 기록하는 작업을 자주 포함하므로 전원 장애 내성(Power-Failure Tolerance)이 필수적이다. 메타데이터 업데이트는 작업 중단으로 인해 어떤 슬롯이 활성 상태이고 확인된 상태인지에 대한 모순된 정보가 남지 않도록 설계해야 한다. 중복 메타데이터 복사본(Redundant Metadata Copy), 체크섬(Checksum), 시퀀스 번호(Sequence Number), 원자적 저장 작업(Atomic Storage Operation), 신중하게 정렬된 상태 전환(State Transition)을 사용하면 설치 또는 활성화 과정의 어느 시점에서 예기치 않은 전원 손실이 발생하더라도 부트로더가 일관된 결정을 복원하는 데 도움이 된다.

부트로더와 업데이터의 책임(Responsibility)은 명확하게 분리해야 한다. OTA 애플리케이션은 일반적으로 패키지를 다운로드하고 상위 수준 검증을 수행하며 비활성 슬롯에 기록한 후 활성화를 요청한다. 부트로더는 작고 견고한 메타데이터를 이용하여 최종 부트 대상을 선택하고 시도 횟수 제한, 인증, 폴백 정책을 적용해야 한다. 부트 결정을 새롭게 설치된 운영체제로부터 독립시키면 해당 운영체제가 정상적으로 초기화되지 못하더라도 시스템을 복구할 수 있다.

U-Boot 기반 플랫폼은 환경 변수, 부트 스크립트(Boot Script), 부트 카운터, 파티션 식별자(Partition Identifier), 검증된 이미지 로딩 명령(Verified Image-Loading Command)을 이용하여 A/B 선택을 구현할 수 있다. 부트 스크립트는 슬롯 상태를 검사하고 후보 또는 확인된 파티션을 선택하며 해당 커널과 장치 트리를 로드하고 실행 전에 시도 정보를 갱신할 수 있다. 부트 변수의 조작을 통해 실행 경로를 변경하거나 업데이트 정책을 무력화할 수 있으므로 제품 환경에서는 이러한 상태 정보를 승인되지 않은 변경으로부터 보호해야 한다.

A/B 업데이트에서는 데이터 형식 호환성(Data-Format Compatibility)도 신중하게 처리해야 한다. 새로운 소프트웨어 버전이 공유 데이터베이스나 구성 형식을 변경하면 롤백 이후 이전 슬롯이 해당 영구 상태를 이해하지 못할 수 있다. 따라서 업데이트 설계에서는 하위 호환 가능한 마이그레이션(Backward-Compatible Migration), 버전이 지정된 데이터 형식(Versioned Data Format), 되돌릴 수 없는 변환의 지연(Deferred Irreversible Conversion), 또는 복구 절차를 고려해야 한다. 복원된 이전 버전이 현재의 영구 데이터를 사용할 수 있을 때에만 소프트웨어 롤백이 실질적인 의미를 가진다.

로봇 시스템은 업데이트 수명주기(Update Lifecycle)에 추가적인 운영 안전 요구사항(Operational Safety Requirement)을 가진다. 일반적으로 로봇은 움직임을 안전하게 중단할 수 있고, 충분한 배터리 또는 외부 전원을 확보했으며, 일시적인 서비스 중단을 허용할 수 있을 때 설치 및 재부팅을 수행해야 한다. 시험 부트와 롤백 과정에서도 액추에이터 활성화(Actuator Enablement)는 정상적인 안전 검증(Safety Validation)의 통제를 받아야 한다. OTA 메커니즘은 성공적인 소프트웨어 설치를 물리적 동작을 시작할 수 있는 권한으로 해석해서는 안 된다.

플릿 배포(Fleet Deployment)에서는 단계적 A/B 활성화(Staged A/B Activation)를 통해 추가적인 이점을 얻을 수 있다. 새로운 릴리스를 먼저 소수의 로봇에 설치하여 부트 성공 여부와 런타임 상태(Runtime Health)를 관찰한 후 점진적으로 더 큰 그룹에 배포할 수 있다. 슬롯 상태, 업데이트 버전, 확인 상태, 롤백 이벤트(Rollback Event), 실패 원인을 플릿 관리 시스템(Fleet Management System)에 보고해야 한다. 이를 통해 A/B 부트는 단순한 장치 수준 복구 기능을 넘어 전체 배포 보증 전략(Deployment Assurance Strategy)의 일부로 확장된다.

따라서 견고한 이중 파티션 설계(Dual-Partition Design)는 OTA를 통제된 상태 전환(Controlled State Transition) 과정으로 취급한다. 업데이트 패키지를 다운로드하고 인증한 후 비활성 슬롯에 기록하고, 내용을 검증하여 후보 상태로 지정하고, 제한된 시험 부트를 수행한 다음 시스템 상태를 평가하여 새로운 슬롯을 확인하거나 이전의 정상 이미지(Known-Good Image)로 복귀한다. 전체 과정에서 하나의 신뢰할 수 있는 부팅 가능 시스템을 지속적으로 보존함으로써 A/B 아키텍처는 원격으로 유지보수되는 임베디드 및 로봇 플랫폼의 핵심적인 신뢰성 기반을 제공한다.

## 04.06 Bootloader-Based Recovery Mechanism Design

![](images/image6.png){width="7.268055555555556in" height="7.268055555555556in"}

부트로더 기반 복구 메커니즘(Bootloader-Based Recovery Mechanism)은 정상 애플리케이션(Application), 운영체제(Operating System), 또는 업데이트 프로세스(Update Process)가 더 이상 안정적으로 시작될 수 없을 때 임베디드 시스템(Embedded System)을 복원하기 위한 독립적인 경로를 제공한다. 부트로더(Bootloader)는 메인 소프트웨어 스택(Main Software Stack)보다 먼저 실행되므로 손상된 소프트웨어에 의존하지 않고 시스템 상태를 평가하고 복구 동작을 선택할 수 있다. 로봇 시스템에서는 이러한 기능이 현장 정비를 줄이고 업데이트 실패로 인해 배치된 로봇이 영구적으로 작동 불능 상태가 되는 것을 방지하는 데 필수적이다.

복구 설계(Recovery Design)는 정상적인 애플리케이션 실행 전에 어떤 장애를 감지할 수 있는지를 정의하는 것에서 시작한다. 대표적인 지표에는 유효하지 않은 애플리케이션 이미지(Application Image), 암호학적 검증(Cryptographic Verification) 실패, 손상된 저장장치 메타데이터(Storage Metadata), 반복적인 워치독 리셋(Watchdog Reset), 실패한 시험 부트(Trial Boot), 명시적인 복구 요청(Recovery Request), 부팅 가능한 파티션(Bootable Partition)의 부재 등이 포함된다. 부트로더는 이러한 신호와 영구 상태(Persistent State)를 결합하여 정상 시작을 계속해도 안전한지 또는 통제된 복구 경로(Controlled Recovery Path)로 실행을 전환해야 하는지를 판단한다.

견고한 아키텍처(Robust Architecture)는 일시적인 부트 실패(Transient Boot Failure)와 지속적인 소프트웨어 장애(Persistent Software Failure)를 구분한다. 한 번의 예기치 않은 리셋은 일시적인 전원 불안정으로 발생할 수 있으므로 반드시 복구를 실행해야 하는 것은 아니지만, 여러 차례 연속되는 실패는 활성 소프트웨어가 정상적인 운영 상태에 도달하지 못하고 있음을 강하게 나타낸다. 영구 부트 카운터(Persistent Boot Counter), 리셋 원인 정보(Reset-Cause Information), 애플리케이션 확인 플래그(Application Confirmation Flag), 워치독 이력(Watchdog History)을 이용하면 부트로더가 프로세서 리셋 이후에도 유지되는 증거를 기반으로 이러한 상태를 구분할 수 있다.

복구 결정(Recovery Decision)은 서로 느슨하게 연결된 부트 명령의 집합이 아니라 결정론적 상태 머신(Deterministic State Machine)으로 구현해야 한다. 상태는 확인된 소프트웨어(Confirmed Software), 후보 소프트웨어(Candidate Software), 실패한 부트 시도(Failed Boot Attempt), 롤백 가능 상태(Rollback Eligibility), 복구 모드(Recovery Mode), 복구 불가능 장애(Unrecoverable Failure) 등을 나타낼 수 있다. 명확하게 정의된 상태 전환(State Transition)은 각각의 리셋에서 예측 가능한 결정을 보장하고 시스템이 통제되지 않는 재부팅 루프(Reboot Loop)에 빠지거나 이미 결함이 있는 것으로 확인된 이미지를 반복적으로 선택하는 것을 방지한다.

복구에는 플랫폼 아키텍처(Platform Architecture)에 따라 여러 소스를 사용할 수 있다. 이중 파티션 시스템(Dual-Partition System)은 이전에 확인된 A/B 슬롯(A/B Slot)으로 복귀할 수 있으며, 다른 설계에서는 최소 운영 환경(Minimal Operating Environment)을 포함하는 전용 복구 파티션(Dedicated Recovery Partition)을 유지할 수 있다. 일부 시스템은 인증된 USB, 네트워크(Network), SD 카드(SD Card), 서비스 도구(Service Tool)를 이용한 복구도 지원한다. 어떤 소스를 선택하더라도 장애 상황에서도 안정적으로 동작할 수 있도록 복구 경로는 정상 런타임 환경보다 단순하고 강력하게 보호되어야 한다.

전용 복구 이미지(Dedicated Recovery Image)는 플랫폼을 진단하고 복원하는 데 필요한 기능만 포함해야 한다. 전체 로봇 애플리케이션 스택을 로드하지 않고 저장장치 접근(Storage Access), 네트워킹(Networking), 업데이트 설치(Update Installation), 이미지 검증(Image Verification), 로깅(Logging), 하드웨어 식별(Hardware Identification), 제한된 진단 서비스(Diagnostic Service)를 제공할 수 있다. 복구 환경을 최소화하면 의존성과 공격 표면(Attack Surface)을 줄이고 기본 소프트웨어 구성이 불일치 상태에 빠진 경우에도 복구 환경이 정상적으로 동작할 가능성을 높일 수 있다.

이미지 무결성(Image Integrity)과 진위성(Authenticity)은 정상 부트에서와 마찬가지로 복구 과정에서도 반드시 강제되어야 한다. 임의의 서명되지 않은 소프트웨어(Unsigned Software)를 허용하는 복구 메커니즘은 보안 부트(Secure Boot)를 직접 우회하는 경로가 될 수 있다. 부트로더는 동일한 신뢰 키 계층(Trusted Key Hierarchy) 또는 명시적으로 승인된 복구 키(Recovery Key)를 이용하여 복구 이미지, 업데이트 패키지(Update Package), 교체 시스템 이미지를 인증해야 한다. 보안 검증 실패는 제한 없는 실행이 아니라 다른 신뢰 가능한 상태로 시스템을 전환해야 한다.

복구 메타데이터(Recovery Metadata)는 애플리케이션 저장장치가 손상된 경우에도 부트로더가 신뢰할 수 있는 정보를 필요로 하기 때문에 매우 중요하다. 메타데이터는 활성 이미지(Active Image), 마지막 확인 이미지(Last Confirmed Image), 후보 상태(Candidate State), 부트 시도 횟수(Boot Attempt Count), 복구 요청, 소프트웨어 버전(Software Version), 검증 결과(Validation Result)를 나타낼 수 있다. 이 정보는 작고 간결해야 하며 체크섬(Checksum)을 적용하고 전원 장애 내성(Power-Failure-Tolerant)을 가진 방식으로 저장해야 한다. 중복 복사본(Redundant Copy)과 시퀀스 번호(Sequence Number)를 사용하면 하나의 메타데이터 업데이트가 중단되더라도 가장 최근의 유효한 상태를 복원하는 데 도움이 된다.

워치독 통합(Watchdog Integration)을 사용하면 복구 로직이 실행은 시작했지만 정상 상태에 도달하지 못한 소프트웨어를 감지할 수 있다. 애플리케이션을 시작하기 전에 부트로더는 애플리케이션이 정의된 체크포인트(Checkpoint)에 도달하여 성공적인 시작을 확인해야 한다는 조건을 설정할 수 있다. 확인 전에 워치독이 프로세서를 리셋하면 영구 시도 카운터(Persistent Attempt Counter)에 또 하나의 실패한 부트가 기록된다. 설정된 임계값(Threshold)을 초과하면 부트로더는 결함 이미지를 더 이상 재시도하지 않고 자동으로 롤백(Rollback) 또는 복구 과정으로 진입할 수 있다.

복구 시스템은 성공적인 부트(Successful Boot)와 성공적인 시스템 동작(Successful System Operation)을 구분해야 한다. 리눅스 커널(Linux Kernel)이 정상적으로 시작되더라도 호환되지 않는 드라이버, 손상된 구성, 누락된 센서 인터페이스(Sensor Interface), 애플리케이션 초기화 오류 등으로 인해 핵심 로봇 서비스가 실패할 수 있다. 따라서 복구 아키텍처는 필수 서비스가 정상적으로 동작한 이후에만 상위 수준 소프트웨어가 상태 확인(Health Confirmation)을 보고하도록 해야 한다. 정의된 정책 내에서 이러한 확인이 이루어지지 않으면 실패한 시험 부트로 판단할 수 있다.

수동 복구 트리거(Manual Recovery Trigger)는 제조, 개발 및 현장 서비스 과정에서도 유용하다. 하드웨어 버튼(Hardware Button), GPIO 스트랩(GPIO Strap), 서비스 커넥터(Service Connector), 인증된 콘솔 명령(Authenticated Console Command), 영구 소프트웨어 플래그(Persistent Software Flag)를 이용하여 다음 부팅 시 복구를 요청할 수 있다. 이러한 트리거는 통제되지 않은 우회 경로가 아니라 의도적으로 설계되어야 한다. 제품용 로봇에서는 물리적 또는 논리적 복구 진입점이 공격자에게 정상적인 인증 및 부트 제한을 쉽게 우회할 수 있는 수단을 제공해서는 안 된다.

네트워크 복구(Network Recovery)는 현장 서비스 접근이 어려운 원격 배치 시스템에서 유용할 수 있다. 최소 복구 환경은 통제된 네트워크 연결을 설정하고 인증된 소프트웨어 패키지를 획득한 후 호환성과 서명을 검증하여 부팅 가능한 슬롯을 복원할 수 있다. 그러나 네트워킹은 복잡성을 크게 증가시키므로 복구 구현에서는 프로토콜(Protocol), 엔드포인트(Endpoint), 자격증명(Credential), 명령을 신뢰된 복원에 필요한 최소 기능으로 제한해야 한다.

저장장치 복구(Storage Recovery)는 부분 기록(Partial Write)과 파일시스템 손상(Filesystem Corruption)을 고려해야 한다. 가능하면 부트로더의 핵심적인 복구 결정은 복잡한 쓰기 가능한 파일시스템보다 단순한 이미지 레이아웃(Image Layout)과 간결한 메타데이터에 의존해야 한다. 특정 파티션을 마운트할 수 없거나 이미지를 검증할 수 없더라도 다른 부트 소스를 선택하는 데 필요한 독립적인 정보는 유지되어야 한다. 하나의 저장 구조에서 발생한 장애가 모든 복원 경로를 파괴하지 않을 때 가장 강력한 복구 아키텍처를 구축할 수 있다.

복구 과정 중의 전원 손실(Power Loss)은 특히 중요하게 고려해야 하는 설계 조건이다. 남아 있는 유일한 정상 이미지를 다시 기록하다가 전원이 차단되면 복구 가능한 장치가 완전히 사용할 수 없는 상태로 바뀔 수 있다. 따라서 교체 이미지가 완전히 기록되고 검증될 때까지 최소 하나의 신뢰된 부트 경로(Trusted Boot Path)를 보존해야 한다. A/B 파티션(A/B Partition), 전용 복구 이미지, 트랜잭션 방식의 메타데이터 업데이트(Transactional Metadata Update), 기록 후 검증(Write-Then-Verify) 절차를 사용하면 예기치 않은 중단에서도 마지막 정상 상태(Known-Good State)를 잃지 않고 복구 작업을 수행할 수 있다.

부트로더 복구는 OTA 업데이트 아키텍처(OTA Update Architecture)와 자연스럽게 통합된다. 업데이터(Updater)는 후보 이미지를 설치하고, 부트로더는 통제된 시험 부트를 수행하며, 애플리케이션은 상태 검사(Health Check)를 통과한 이후에만 성공적인 동작을 확인한다. 확인이 이루어지지 않으면 복구 로직은 이전의 정상 슬롯(Previous Known-Good Slot)을 선택한다. 이러한 책임 분리를 통해 업데이트 소프트웨어는 다양한 기능을 유지하면서도 최종 복구 결정은 더 작고 신뢰성이 높은 애플리케이션 실행 이전 구성요소(Pre-Application Component)에서 수행할 수 있다.

U-Boot 기반 시스템은 부트 카운터, 환경 변수(Environment Variable), 스크립트(Script), 파티션 선택(Partition Selection), 검증된 이미지 명령(Verified Image Command), 전용 복구 대상(Dedicated Recovery Target)을 통해 복구 정책을 구현할 수 있다. 제품 설계에서는 이러한 제어 요소가 승인되지 않은 방식으로 변경되지 않도록 보호하고 지나치게 복잡한 스크립트를 피해야 한다. 복구 로직은 정상 부트, 후보 이미지 부트 실패, 롤백, 손상된 이미지, 복구 요청, 반복적인 복구 실패를 포함한 모든 중요한 상태 전환을 테스트할 수 있을 정도로 명확하고 이해하기 쉬워야 한다.

로봇 복구 메커니즘(Robotic Recovery Mechanism)은 소프트웨어 복원 과정과 물리적 안전(Physical Safety)을 함께 조정해야 한다. 복구 모드에 진입하면 일반적으로 동작 명령(Motion Command)을 차단하고 액추에이터를 비활성화하며 브레이크(Brake) 또는 안전 릴레이(Safety Relay)를 정의된 상태로 유지하고 독립적인 안전 컨트롤러(Independent Safety Controller)와 적절하게 통신해야 한다. 소프트웨어를 복구하고 있는 로봇은 소프트웨어 무결성과 시스템 준비 상태(System Readiness)가 정상적인 안전 검증 절차를 통해 다시 확립될 때까지 비운용 상태(Non-Operational State)로 간주해야 한다.

운영자가 정상 부트 실패의 원인을 파악하기 위해서는 진단(Diagnostics)도 복구 시스템의 중요한 보조 기능이다. 리셋 원인, 부트 카운터, 이미지 버전(Image Version), 검증 결과, 저장장치 상태, 업데이트 이력(Update History), 오류 코드(Error Code)를 로컬 또는 원격 검사에 사용할 수 있도록 유지할 수 있다. 로그(Log)는 제한된 크기로 관리해야 하며 진단용 저장장치의 장애가 복구 자체를 방해하지 않도록 설계해야 한다. 가장 중요한 목표는 시스템 복원이며, 진단 정보는 동일한 장애의 재발을 방지하기 위한 정보를 제공한다.

복구 테스트(Recovery Testing)는 아키텍처가 견딜 수 있다고 정의한 장애를 의도적으로 재현해야 한다. 이미지 기록 및 메타데이터 업데이트 중 전원을 차단하고, 후보 이미지를 손상시키며, 워치독 리셋을 발생시키고, 부트 시도 횟수를 소진하고, 서명을 무효화하며, 선택된 파티션을 손상시키고, 수동 복구 요청을 시험해야 한다. 정상적인 경로가 동작한다는 이유만으로 복구 메커니즘을 신뢰할 수 있다고 판단해서는 안 되며, 의도적으로 주입된 장애(Fault Injection)에서도 예측 가능한 동작을 보일 때 그 신뢰성을 입증할 수 있다.

따라서 잘 설계된 부트로더 복구 메커니즘은 운영체제와 로봇 애플리케이션 아래에서 동작하는 보호된 제어 계층(Protected Control Layer)을 형성한다. 지속적인 시작 실패를 감지하고, 신뢰된 메타데이터를 평가하며, 재시도 횟수를 제한하고, 사용 가능한 이미지를 인증하며, 롤백 또는 복구 경로를 선택하고, 정상 상태의 시스템을 보존하면서 복원 과정에서 물리적 안전을 유지한다. 보안 부트(Secure Boot) 및 A/B 업데이트(A/B Update)와 결합하면 원격으로 배치된 임베디드 및 로봇 플랫폼이 보안이나 운영 제어(Operational Control)를 포기하지 않고 소프트웨어 장애에서 스스로 복구할 수 있도록 한다.

## 04.07 UEFI Boot Architecture for x86 Edge PC

![](images/image7.png){width="7.268055555555556in" height="7.268055555555556in"}

통합 확장 펌웨어 인터페이스(Unified Extensible Firmware Interface, UEFI)는 최신 x86 엣지 PC(Edge PC)를 초기화하고 플랫폼 하드웨어(Platform Hardware)에서 운영체제(Operating System)로 제어권을 전달하는 펌웨어 아키텍처(Firmware Architecture)를 정의한다. 고정된 부트 규칙과 리얼 모드 호환성(Real-Mode Compatibility)에 크게 의존하는 레거시 BIOS(Legacy BIOS)와 달리 UEFI는 구조화된 인터페이스, 실행 가능한 펌웨어 애플리케이션(Firmware Application), 표준화된 부트 서비스(Boot Services), 영구 변수(Persistent Variable), 운영체제 부트로더를 검색하고 실행하기 위한 유연한 메커니즘을 제공한다.

x86 부트 프로세스(Boot Process)는 프로세서 리셋 직후 리눅스(Linux), 윈도우(Windows), 또는 로봇 미들웨어(Robotic Middleware)가 사용 가능해지기 전에 시작된다. 펌웨어는 프로세서 실행 환경(Processor Execution Environment)을 구축하고 메모리와 필수 칩셋 자원(Chipset Resource)을 사용할 수 있도록 플랫폼별 초기화를 수행한다. 산업용 엣지 PC(Industrial Edge PC)에서는 CPU 구성, 메모리 컨트롤러(Memory Controller), PCIe 토폴로지(PCIe Topology), 저장장치 인터페이스(Storage Interface), 워치독 하드웨어(Watchdog Hardware), 관리 컨트롤러(Management Controller), 보드별 전원 시퀀싱(Power Sequencing)을 고려해야 한다.

현대적인 펌웨어 구현은 일반적으로 초기 초기화 과정을 여러 논리적 단계(Logical Phase)로 구분한다. 보안 중심의 초기 단계에서는 신뢰할 수 있는 실행 기반(Trusted Execution Foundation)을 구축하고, 사전 EFI 초기화(Pre-EFI Initialization)에서는 저수준 프로세서, 칩셋, 메모리 초기화를 수행한다. 충분한 DRAM을 사용할 수 있게 되면 드라이버 실행 환경(Driver Execution Environment)이 펌웨어 드라이버(Firmware Driver)를 로드하고 플랫폼 장치를 열거(Enumeration)할 수 있다. 이러한 단계는 시스템을 리셋 상태에서 부트 대상을 검색하고 실행할 수 있는 상태로 점진적으로 전환한다.

드라이버 실행 환경(Driver Execution Environment)에서 UEFI는 하드웨어를 검색하고 이후 부트 구성요소가 필요로 하는 서비스를 구축한다. 펌웨어 드라이버는 PCI Express 장치, NVMe 컨트롤러, SATA 인터페이스, USB 컨트롤러, 네트워크 어댑터(Network Adapter), 그래픽 하드웨어(Graphics Hardware), 기타 플랫폼 자원을 초기화할 수 있다. 이러한 하드웨어 검색은 가속기 카드(Accelerator Card), 산업용 네트워크 인터페이스, GPU 장치, 고속 저장장치가 운영체제의 로딩과 구성에 영향을 줄 수 있는 엣지 PC에서 특히 중요하다.

UEFI는 부트 서비스(Boot Services)와 런타임 서비스(Runtime Services)를 분리한다. 부트 서비스는 펌웨어가 플랫폼을 제어하는 동안 필요한 메모리 할당(Memory Allocation), 프로토콜 검색(Protocol Discovery), 장치 접근(Device Access), 이미지 로딩(Image Loading), 타이머(Timer) 등의 기능을 제공한다. 런타임 서비스는 운영체제가 제어권을 획득한 이후에도 사용할 수 있는 더 작은 인터페이스 집합으로, 펌웨어 변수, 시스템 시간(System Time), 일부 플랫폼 동작을 지원한다. 이러한 분리는 펌웨어 초기화와 운영체제의 하드웨어 소유권 사이에 명확한 경계를 형성한다.

UEFI 부트 관리자(Boot Manager)는 영구적인 펌웨어 구성(Persistent Firmware Configuration)에 따라 어떤 실행 이미지를 시작할 것인지 결정한다. 부트 항목(Boot Entry)은 비휘발성 변수(Nonvolatile Variable)에 저장되며 특정 장치, 파티션, 파일 또는 대체 복구 경로(Recovery Path)를 지정할 수 있다. BootOrder는 우선 실행 순서를 정의하고 개별 Boot#### 항목은 특정 부트 대상을 설명한다. 이를 통해 엣지 PC는 정상, 유지보수, 복구, 이동식 미디어(Removable Media) 부트 옵션을 명시적인 펌웨어 정책 아래에서 유지할 수 있다.

현대적인 x86 시스템은 일반적으로 GUID 파티션 테이블(GUID Partition Table, GPT)과 EFI 시스템 파티션(EFI System Partition, ESP)을 함께 사용한다. EFI 시스템 파티션에는 UEFI 펌웨어가 직접 접근할 수 있는 파일시스템에 부트 애플리케이션과 관련 파일이 저장된다. 기존 BIOS 설계처럼 고정된 디스크 섹터에서 실행 코드를 읽는 대신 UEFI는 운영체제 부트 관리자와 같이 정의된 실행 파일을 찾아 펌웨어가 인식할 수 있는 애플리케이션으로 로드할 수 있다.

리눅스 기반 엣지 컴퓨터에서는 선택된 EFI 실행 파일(EFI Executable)이 GRUB, systemd-boot, 공급업체 부트 관리자(Vendor Boot Manager), 또는 직접 부팅 가능한 EFI 호환 커널 구성일 수 있다. 부트 관리자는 선택된 운영체제를 식별하고 커널 매개변수(Kernel Parameter)를 준비하며 커널과 선택적인 초기 RAM 파일시스템(Initial RAM Filesystem)을 찾는다. 이후 제어권은 UEFI 펌웨어에서 운영체제 로더(OS Loader)로, 다시 커널로 전달되며 커널은 자체 드라이버를 초기화하고 시스템에 대한 제어권을 획득한다.

펌웨어에서 운영체제로의 전환은 부트 서비스 종료(ExitBootServices)를 통해 공식적으로 이루어진다. 이 호출 이전에는 운영체제 로더가 메모리, 장치, 프로토콜 작업을 위해 UEFI 부트 서비스를 사용할 수 있다. 로더는 펌웨어 메모리 맵(Firmware Memory Map)을 획득하고 커널 환경을 준비한 후 제어권을 인수할 준비가 완료되면 ExitBootServices를 호출한다. 이후 일반적인 부트 서비스는 더 이상 사용할 수 없으며 운영체제가 하드웨어 자원 관리의 책임을 담당한다.

UEFI 보안 부트(UEFI Secure Boot)는 실행 이미지 로딩 과정에 인증(Authentication)을 추가한다. 펌웨어는 신뢰할 수 있는 서명 데이터베이스(Trusted Signature Database)를 유지하고 보안 부트 정책에 따라 EFI 실행 파일을 허용하기 전에 이를 검증한다. 이를 통해 플랫폼 펌웨어에서 운영체제 부트로더와 이후의 검증된 구성요소까지 이어지는 신뢰 체인(Chain of Trust)을 구축할 수 있다. 로봇 엣지 컴퓨터에서는 로봇 소프트웨어 보안 스택이 활성화되기 전에 승인되지 않은 부트 소프트웨어가 높은 권한의 제어권을 획득하는 것을 방지하는 것이 목적이다.

보안 부트는 단순한 부트 순서가 아니라 키 및 서명 데이터베이스의 계층 구조(Hierarchy)에 의존한다. 플랫폼 키(Platform Key), 키 교환 키(Key Exchange Key), 승인된 서명 데이터베이스(Authorized Signature Database), 금지 또는 폐기된 서명 정보(Revoked Signature Information)가 어떤 실행 파일을 신뢰하거나 거부할지를 정의한다. 제품 프로비저닝(Production Provisioning)에서는 이러한 값을 보호하고 키를 어떻게 업데이트하거나 폐기할지를 정의해야 한다. 잘못된 키 관리는 부트 보안을 약화시키거나 정상적인 현장 소프트웨어가 시작되지 못하게 만들 수 있다.

신뢰 플랫폼 모듈(Trusted Platform Module, TPM)은 측정 부트(Measured Boot)를 통해 UEFI 보안 부트를 보완할 수 있다. 실행 파일의 실행 허용 여부만 판단하는 대신 측정 부트는 펌웨어 및 부트 구성요소의 암호학적 측정값(Cryptographic Measurement)을 보호된 TPM 플랫폼 구성 레지스터(Platform Configuration Register, PCR)에 기록한다. 이후 로컬 또는 원격 소프트웨어는 이러한 측정값을 평가하여 시스템이 예상된 구성으로 시작되었는지 확인할 수 있으며, 이를 통해 장치 증명(Device Attestation)과 플릿 보안 모니터링(Fleet Security Monitoring)을 지원할 수 있다.

UEFI 구성은 산업용 엣지 하드웨어의 신뢰성에도 영향을 준다. PCIe 자원 할당(Resource Allocation), 4G 초과 디코딩(Above 4G Decoding), IOMMU 설정, 크기 조정 가능 BAR(Resizable BAR), NVMe 구성, 가상화 확장(Virtualization Extension), 워치독 동작, 전원 복구 정책(Power Restoration Policy)은 부팅 이후 GPU 및 가속기의 사용 가능 여부에 영향을 줄 수 있다. 따라서 고성능 GPU 또는 AI 가속기(AI Accelerator)를 포함하는 엣지 플랫폼에서는 펌웨어 설정을 단순한 기술자의 임의 설정이 아니라 배포된 시스템 구성의 일부로 관리해야 한다.

배포된 로봇에서는 부트 장치 선택(Boot-Device Selection)을 신중하게 통제해야 한다. USB, 네트워크 PXE, 이동식 미디어, 내부 NVMe, 복구 장치가 모두 기술적으로 부팅 가능할 수 있지만 모든 경로를 활성화하면 운영상의 불확실성과 보안 노출(Security Exposure)이 증가한다. 제품용 펌웨어는 제한된 부트 순서를 정의하고 어떤 대체 경로를 허용할 것인지 명확하게 결정해야 한다. 유지보수 유연성을 유지하면서도 통제되지 않은 소프트웨어가 신뢰된 운영 환경을 대체하지 못하도록 해야 한다.

복구 아키텍처(Recovery Architecture)는 UEFI 부트 항목을 사용하여 독립적인 복원 경로(Restoration Path)를 유지할 수 있다. 정상 동작에서는 기본 운영체제 항목을 우선 선택하고, 반복적인 장애가 발생하거나 승인된 유지보수 요청이 있을 때 다른 EFI 애플리케이션 또는 복구 파티션(Recovery Partition)을 선택할 수 있다. 복구 환경은 정상적인 로봇 애플리케이션이 동작하지 않더라도 저장장치를 진단하고 검증된 시스템 이미지를 재설치하며 부트 구성을 복구하거나 이전의 정상 소프트웨어 상태(Known-Good Software State)를 복원할 수 있다.

x86 엣지 PC의 OTA 업데이트(OTA Update)는 운영체제 파티션과 이를 선택하는 UEFI 부트 구성을 함께 고려해야 한다. A/B 시스템 파티션(A/B System Partition)을 독립적인 EFI 부트 항목과 결합하면 기존의 정상 시스템을 즉시 제거하지 않고 후보 이미지(Candidate Image)를 시험할 수 있다. 성공적인 시작 이후 후보 이미지를 확인(Confirm)할 수 있으며, 반복적인 실패가 발생하면 이후의 부팅을 이전 슬롯 또는 전용 복구 환경으로 전환할 수 있다.

펌웨어 업데이트(Firmware Update)는 UEFI 자체가 운영체제보다 하위 계층에서 동작하기 때문에 또 다른 위험 요소를 발생시킨다. 중단되거나 호환되지 않는 펌웨어 업데이트는 엣지 PC가 운영체제 복구 메커니즘에 도달하는 것 자체를 방해할 수 있다. 따라서 산업용 플랫폼에서는 서명된 펌웨어 캡슐(Signed Firmware Capsule), 보호된 업데이트 절차(Protected Update Procedure), 지원되는 경우 롤백 또는 중복 펌웨어(Redundant Firmware), 펌웨어 변경 중 안정적인 외부 전원을 사용하는 것이 바람직하다. 펌웨어 수명주기 관리(Firmware Lifecycle Management)는 일반적인 애플리케이션 OTA 정책과 분리하여 관리해야 한다.

고속 부트(Fast Boot) 옵션은 로봇 시스템에서 신중하게 평가해야 한다. POST(Power-On Self-Test)와 장치 초기화를 줄이면 전원 재인가 이후의 복구 시간을 단축할 수 있지만 지나친 최적화는 신뢰성 있는 동작에 필요한 하드웨어 검색이나 진단 동작을 생략할 수 있다. 독립형 GPU, PCIe 가속기, 여러 NVMe 장치 또는 산업용 인터페이스를 사용하는 엣지 PC에서는 공격적인 시작 최적화를 활성화하기 전에 콜드 부트(Cold Boot), 웜 리부트(Warm Reboot), 워치독 리셋, 예상하지 못한 전원 손실 조건에서 검증해야 한다.

UEFI 진단(UEFI Diagnostics)은 운영체제를 사용할 수 있기 전에 엣지 PC의 부팅이 실패한 경우 중요한 증거를 제공한다. 펌웨어 이벤트 로그(Firmware Event Log), POST 표시 정보, 지원되는 경우 직렬 출력(Serial Output), 부트 관리자 상태, TPM 측정값, 저장장치 감지 결과, PCIe 열거 정보를 이용하면 펌웨어 장애와 운영체제 장애를 구분할 수 있다. 로봇 배포 환경에서 오프라인으로 보이는 시스템이 실제로는 리눅스 또는 플릿 모니터링 소프트웨어가 애플리케이션 로그를 생성하기 훨씬 이전 단계에서 실패했을 수 있으므로 이러한 구분은 중요하다.

로봇 엣지 PC에서는 부트 완료(Boot Completion)와 실제 운영 준비 상태(Operational Readiness)를 구분해야 한다. UEFI가 운영체제를 성공적으로 로드하더라도 GPU 드라이버, 네트워크 링크(Network Link), 위치추정 서비스(Localization Service), 안전 통신(Safety Communication), 로봇 미들웨어가 여전히 초기화 중일 수 있다. 따라서 액추에이터 승인(Actuator Authorization)은 펌웨어의 부트 성공 여부가 아니라 로봇 안전 아키텍처(Robot Safety Architecture)에 의해 관리되어야 한다. 전체 소프트웨어 및 안전 스택이 검증된 준비 상태(Verified Ready State)를 확립한 이후에만 시스템을 실제 운용 상태로 전환해야 한다.

따라서 잘 설계된 x86 엣지 PC용 UEFI 아키텍처는 구조화된 펌웨어 초기화(Structured Firmware Initialization), 하드웨어 열거(Hardware Enumeration), GPT 및 EFI 시스템 파티션 관리, 통제된 부트 항목, 보안 부트(Secure Boot), 선택적인 TPM 측정 부트(TPM Measured Boot), 복구 경로, 신뢰성 높은 운영체제 제어권 전달(OS Handoff)을 결합한다. 이를 A/B 업데이트, 워치독 복구(Watchdog Recovery), 펌웨어 보안(Firmware Security), 로봇 안전 제어(Robotic Safety Control)와 통합하면 UEFI는 고성능 엣지 컴퓨팅 플랫폼(High-Performance Edge Computing Platform)을 위한 유지보수 가능하고 신뢰할 수 있는 시작 아키텍처의 기반이 된다.

## 04.08 Embedded Linux Boot Optimization: Reducing Boot Time

![](images/image8.png){width="7.268055555555556in" height="7.268055555555556in"}

임베디드 리눅스 부트 최적화(Embedded Linux Boot Optimization)는 하드웨어 리셋(Hardware Reset)부터 장치가 의도된 애플리케이션 기능을 수행할 수 있는 시점까지의 시간을 체계적으로 단축하는 과정이다. 목표는 단순히 로그인 프롬프트(Login Prompt)를 더 빨리 표시하는 것이 아니라 필요한 드라이버, 통신 링크, 서비스 및 로봇 기능이 준비된 운영 준비 상태(Operational-Ready State)에 도달하는 것이다. 따라서 최적화는 펌웨어(Firmware), 부트로더(Bootloader), 커널(Kernel), 루트 파일시스템(Root Filesystem), 사용자 공간 초기화(Userspace Initialization), 애플리케이션 시작을 하나의 연속적인 시퀀스로 다루어야 한다.

부트 시간(Boot Time)은 하나의 숫자로 취급하기보다 먼저 측정 가능한 단계로 분해해야 한다. 일반적인 시퀀스는 부트 ROM(Boot ROM) 또는 펌웨어 초기화, 부트로더 실행, 커널 로딩 및 압축 해제, 커널 초기화, 루트 파일시스템 마운트(Root Filesystem Mounting), 초기화 시스템(Init System) 시작, 서비스 활성화(Service Activation), 애플리케이션 준비 상태(Application Readiness)로 구성된다. 이러한 단계 사이의 경계를 측정하면 실제로 시간이 소비되는 위치를 파악하고 전체 부트 시간에 거의 영향을 주지 않는 구성요소에 최적화 노력을 낭비하는 것을 방지할 수 있다.

효과적인 최적화 과정에서는 실제 하드웨어 조건에서 반복 가능한 기준선(Baseline)을 먼저 설정해야 한다. 직렬 콘솔 타임스탬프(Serial-Console Timestamp), GPIO 전환, 커널 타임스탬프(Kernel Timestamp), systemd-analyze, 부트차트 방식 추적(Bootchart-Style Trace), 애플리케이션 수준 준비 상태 마커(Readiness Marker)를 이용하여 주요 시간 경계를 식별할 수 있다. 콜드 부트(Cold Boot), 웜 리부트(Warm Reboot), 워치독 리셋(Watchdog Reset), 복구 부트(Recovery Boot)는 저장장치 초기화, 주변장치 상태, 메모리 트레이닝(Memory Training), 하드웨어 열거(Hardware Enumeration)에 따라 시작 동작이 크게 달라질 수 있으므로 각각 별도로 측정해야 한다.

부트로더 최적화(Bootloader Optimization)는 제품 운영에 기여하지 않는 지연 요소를 제거하는 것에서 시작한다. 대화형 부트 카운트다운(Interactive Boot Countdown), 불필요한 콘솔 대기, 중복 하드웨어 테스트, 사용하지 않는 저장장치 검색, 네트워크 검색(Network Discovery), 개발 전용 진단 기능은 상당한 지연을 발생시킬 수 있다. U-Boot 구성에서는 필요한 명령과 드라이버만 활성화하고, 부트 시퀀스는 기술적으로 사용 가능한 모든 부트 소스를 반복적으로 검색하는 대신 지정된 저장장치와 이미지를 직접 선택하도록 구성해야 한다.

하드웨어 초기화(Hardware Initialization)는 리눅스가 시작되기 전에 반드시 필요한 작업과 안전하게 지연할 수 있는 작업으로 구분해야 한다. DRAM 초기화와 부트 저장장치 접근은 일반적으로 필수적이지만 일부 USB 컨트롤러, 네트워크 인터페이스, 디스플레이 또는 선택적 주변장치는 부트로더 내부에서 완전하게 초기화할 필요가 없을 수 있다. 펌웨어, 부트로더, 커널 사이에서 중복되는 초기화를 방지하면 각 장치의 명확한 소유권 경계(Ownership Boundary)를 유지하면서 부트 지연을 줄일 수 있다.

커널 이미지 로딩 시간(Kernel Image Loading Time)은 저장장치 처리량(Storage Throughput), 이미지 크기, 압축 형식(Compression Format), 메모리 대역폭(Memory Bandwidth), 압축 해제 성능(Decompression Performance)에 따라 달라진다. 작은 커널은 일반적으로 읽어야 하는 데이터 양이 적지만 과도한 압축은 CPU의 압축 해제 시간을 증가시킬 수 있다. 따라서 최적의 구성은 대상 프로세서와 저장 매체에 따라 달라지며, 가장 작은 압축 이미지가 자동으로 가장 빠른 시작을 제공한다고 가정하기보다 전체 로딩 시간과 압축 해제 시간을 함께 측정하여 비교해야 한다.

커널 구성(Kernel Configuration)은 가장 효과적인 최적화 영역 중 하나이다. 범용 리눅스 커널(General-Purpose Linux Kernel)은 임베디드 제품에서 전혀 사용하지 않을 수 있는 수많은 파일시스템, 버스, 프로토콜, 장치 및 디버깅 기능을 지원한다. 불필요한 기능을 제거하면 커널 크기와 초기화 작업을 줄일 수 있다. 제품별 Kconfig는 필요한 플랫폼 기능만 유지하면서 현장 운용의 신뢰성을 위해 필요한 진단, 보안, 복구 및 업데이트 기능을 보존하도록 구성해야 한다.

장치 프로빙(Device Probing)은 커널 시작 과정에서 상당하고 때로는 예측하기 어려운 지연을 발생시킬 수 있다. 드라이버가 버스를 검색하거나 존재하지 않는 장치를 기다리고, 통신을 재시도하거나 하드웨어가 준비될 때까지 차단될 수 있다. 장치 트리(Device Tree)는 실제 설치된 하드웨어를 정확하게 표현하여 커널이 존재하지 않는 구성요소를 초기화하지 않도록 해야 한다. 특히 저장장치, 네트워크, USB, I2C, SPI, PCIe, GPU, 산업용 통신 장치의 긴 프로브 지연(Probe Delay)은 개별적으로 조사해야 한다.

내장 드라이버(Built-In Driver)와 로드 가능한 커널 모듈(Loadable Kernel Module)은 중요한 시작 성능 절충 관계(Startup Tradeoff)를 가진다. 루트 저장장치, 콘솔 접근, 안전 통신 또는 필수 로봇 하드웨어에 즉시 필요한 드라이버는 초기 사용 가능성을 보장하기 위해 커널에 직접 포함할 수 있다. 중요도가 낮은 기능은 모듈 형태로 유지하여 나중에 로드할 수 있다. 그러나 모든 드라이버를 모듈로 전환한다고 자동으로 시작 속도가 향상되는 것은 아니며, 모듈 검색, 파일시스템 접근, 의존성 해결(Dependency Resolution), 순차적 로딩이 추가 지연을 발생시킬 수 있다.

커널 명령줄 매개변수(Kernel Command-Line Parameter)는 시작 동작에 영향을 줄 수 있으므로 최적화 구성의 일부로 관리해야 한다. 콘솔 설정, 루트 파일시스템 선택, 로깅 수준(Logging Level), 장치별 매개변수, 초기화 옵션 등이 부트 시간에 영향을 줄 수 있다. 느린 직렬 콘솔을 통한 과도한 진단 출력은 일부 플랫폼에서 시작 시간을 눈에 띄게 증가시킬 수 있다. 제품용 로깅은 결함 분석에 충분한 정보를 유지하면서도 상세한 개발 추적 정보가 핵심 부트 경로(Critical Boot Path)를 지배하지 않도록 해야 한다.

루트 파일시스템도 커널이 사용자 공간 프로그램과 라이브러리를 시작하기 전에 이를 검색하고 마운트하며 접근해야 하므로 부트 성능에 영향을 준다. 불필요한 패키지, 파일, 라이브러리, 로케일(Locale), 문서, 개발 도구를 제거하면 저장장치 접근과 파일시스템 메타데이터 처리량을 줄일 수 있다. Yocto 또는 Buildroot를 이용하면 범용 데스크톱 배포판의 광범위한 패키지 구성을 그대로 사용하는 대신 제품별 루트 파일시스템(Product-Specific Root Filesystem)을 생성할 수 있다.

사용자 공간 최적화(Userspace Optimization)는 단순히 서비스를 삭제하는 것이 아니라 서비스 사이의 의존성을 이해해야 한다. systemd와 같은 초기화 시스템은 서로 독립적인 서비스를 병렬로 시작할 수 있지만 잘못 정의된 의존성은 작업을 불필요하게 직렬화(Serialization)할 수 있다. 서비스는 실제로 필요한 자원만 의존성으로 선언해야 한다. 예를 들어 CAN과 안전 컨트롤러가 필요한 로봇 애플리케이션은 초기화를 시작하기 전에 관련 없는 클라우드 서비스, 그래픽 환경, 패키지 관리자 또는 선택적 네트워크 인터페이스까지 기다릴 필요가 없을 수 있다.

따라서 모든 서비스를 동일하게 최적화하는 것보다 임계 경로 분석(Critical-Path Analysis)이 더 중요하다. 시작 의존성 그래프(Startup Dependency Graph)를 분석하여 사용자 공간 초기화에서 운영 준비 상태까지 이어지는 가장 긴 작업 시퀀스를 찾아야 한다. 임계 경로 밖에서 실행되는 서비스를 500밀리초 단축해도 전체 부트 시간에는 거의 영향을 주지 않을 수 있지만, 직렬화된 의존성 체인에서 200밀리초의 지연을 제거하면 실제 준비 시간을 직접 단축할 수 있다. 따라서 최적화는 최종 준비 시점을 결정하는 의존성 체인에 집중해야 한다.

의존성이 허용하는 경우 병렬 초기화(Parallel Initialization)는 시작 시간을 크게 줄일 수 있다. 네트워크 설정, 센서 검색, 로깅, 애플리케이션 준비, 비필수 하드웨어 초기화를 순차적으로 수행하지 않고 동시에 실행할 수 있다. 그러나 병렬화는 실제 자원 의존성과 경쟁 상태(Race Condition)를 고려해야 한다. 로봇 프로세스를 일찍 시작하는 것은 해당 프로세스가 필요로 하는 인터페이스가 안정적인 소유권, 예측 가능한 장치 이름, 유효한 구성 및 충분히 초기화된 하드웨어를 확보한 경우에만 의미가 있다.

지연 초기화(Deferred Initialization)는 중요하지 않은 작업을 운영 준비 상태 이후로 이동시키는 방식이다. 로그 순환(Log Rotation), 원격 측정 업로드(Telemetry Upload), 패키지 유지보수, 선택적 진단, 보조 네트워크 검색, 사용자 인터페이스 구성요소, 비필수 데이터베이스 처리는 기본적인 로봇 기능의 시작을 지연시킬 필요가 없을 수 있다. 안전 및 기능 요구사항에서 어떤 기능이 실제 동작 전에 필수적인지 명확하게 정의한다면 시스템은 최소 운영 상태(Minimal Operational State)에 먼저 도달한 후 보조 기능을 점진적으로 활성화할 수 있다.

애플리케이션 시작(Application Startup) 자체가 최종적으로 가장 큰 지연 원인이 되는 경우도 많다. 대규모 로봇 프레임워크(Robotic Framework)는 준비 상태를 보고하기 전에 플러그인(Plugin)을 로드하고, 구성을 분석하며, 미들웨어 피어(Middleware Peer)를 검색하고, GPU 라이브러리를 초기화하고, 메모리를 할당하며, AI 모델과 센서 파이프라인을 로드할 수 있다. 따라서 부트 최적화는 리눅스 초기화를 넘어 로봇 애플리케이션까지 확장되어야 하며, 프로파일링(Profiling)을 통해 모델 로딩, 미들웨어 검색, 저장장치 접근, GPU 컨텍스트 생성, 순차적 센서 초기화 중 전체 운영 준비 시간을 지배하는 요소를 찾아야 한다.

엣지 AI 로봇(Edge AI Robot)에서는 부트 완료(Boot-Complete)와 AI 준비 상태(AI-Ready)를 서로 다른 이정표(Milestone)로 정의할 필요가 있다. 리눅스가 사용자 공간에 빠르게 도달하더라도 GPU 드라이버, CUDA 관련 런타임 구성요소, 추론 엔진(Inference Engine), 신경망 가중치(Neural-Network Weight), 카메라 파이프라인(Camera Pipeline), 인식 서비스(Perception Service)는 계속 초기화될 수 있다. 따라서 부트 성능 사양은 단일 운영체제 부트 시간만 보고하기보다 커널 준비, 네트워크 준비, 미들웨어 준비, 인식 준비, 안전 준비, 임무 준비(Mission-Ready)와 같은 단계별 시점을 정의해야 한다.

사전 로딩(Preloading)과 캐싱(Caching)은 반복적인 초기화 비용을 줄일 수 있지만 임베디드 신뢰성을 고려하여 신중하게 설계해야 한다. 자주 사용하는 구성 데이터 또는 애플리케이션 산출물은 효율적인 순차 접근을 위해 구성할 수 있으며 대형 AI 모델은 빠른 로딩에 최적화된 형식으로 저장할 수 있다. 영구 캐시(Persistent Cache)는 버전 관리와 검증을 적용하여 OTA 업데이트 이후 단순히 시작 시간을 줄이기 위해 호환되지 않는 데이터를 재사용하지 않도록 해야 한다. 작은 시간 개선보다 시스템 신뢰성이 우선되어야 한다.

보안 작업(Security Operation)은 측정 가능한 지연을 발생시키더라도 신뢰된 시작 경로(Trusted Startup Path) 안에 유지되어야 한다. 보안 부트(Secure Boot) 서명 검증, 무결성 검사, 롤백 방지(Anti-Rollback) 평가, 암호화 저장장치 설정(Encrypted Storage Setup), 키 초기화(Key Initialization)를 단순히 부트 성능을 높이기 위해 비활성화해서는 안 된다. 대신 암호 가속(Cryptographic Acceleration), 효율적인 이미지 구성, 적절한 서명 범위(Signature Scope), 하드웨어 신뢰 루트(Hardware Root of Trust)를 활용하여 보안 보장을 유지하면서 처리 시간을 줄여야 한다.

복구(Recovery) 및 A/B 업데이트 메커니즘(A/B Update Mechanism) 역시 최적화 이후에도 유지되어야 한다. 부트 카운터(Boot Counter), 이미지 검증(Image Validation), 폴백 검사(Fallback Check), 복구 결정을 제거하면 정상 조건에서는 뛰어난 부트 시간을 얻을 수 있지만 실제 배치된 시스템의 복원력(Resilience)은 저하될 수 있다. 최적화된 부트로더는 여전히 실패한 후보 이미지를 인식하고, 정상 이미지(Known-Good Image)를 선택하며, 소프트웨어를 인증하고, 필요한 경우 복구 모드로 진입할 수 있어야 한다. 빠른 시작은 안전하고 복구 가능한 현장 운용과 양립할 때에만 가치가 있다.

로봇 시스템에는 최적화 과정에서도 제거할 수 없는 안전 경계(Safety Boundary)가 존재한다. 커널 또는 애플리케이션의 시작 속도를 높이더라도 안전 검증이 완료되기 전에 모터, 브레이크, 릴레이 또는 액추에이터가 동작 상태로 진입해서는 안 된다. 안전 컨트롤러(Safety Controller)와 중요 출력은 전체 부트 과정에서 정의된 안전 상태(Safe State)를 유지해야 한다. 최종 준비 신호(Final Readiness Signal)는 단순히 가장 빠른 실행 경로가 완료되었다는 의미가 아니라 필요한 소프트웨어 및 안전 체인(Safety Chain)의 초기화가 성공적으로 완료되었음을 의미해야 한다.

마지막으로 부트 시간 최적화는 회귀 관리되는 엔지니어링 지표(Regression-Controlled Engineering Metric)로 지속적으로 관리해야 한다. 커널 변경, 드라이버 추가, 서비스 의존성, OTA 기능, 보안 메커니즘 또는 애플리케이션 업데이트가 누적되면서 시작 시간이 점진적으로 증가할 수 있다. 자동화된 테스트(Automated Test)를 통해 단계별 시간을 기록하고 정의된 시간 예산(Timing Budget)과 비교할 수 있다. 펌웨어, 부트로더, 커널, 사용자 공간 및 로봇 준비 상태를 지속적으로 측정함으로써 빠른 부트(Fast Boot)를 일회성 최적화 결과가 아니라 지속적으로 유지되는 시스템 특성(System Property)으로 관리할 수 있다.

## 04.09 Robot Power Sequence and Bootloader Integration

![](images/image9.png){width="7.268055555555556in" height="7.268055555555556in"}

로봇 전원 시퀀싱(Robot Power Sequencing)은 전기 전원 도메인(Electrical Power Domain), 프로세서, 센서, 통신 장치, 안전 컨트롤러(Safety Controller), 액추에이터(Actuator)가 전원을 공급받고 동작 상태로 전환되는 제어된 순서를 정의한다. 부트로더 통합(Bootloader Integration)은 이러한 물리적 전원 공급 과정과 소프트웨어 시작을 연결하여 컴퓨팅 시스템이 각 단계에서 어떤 하드웨어 자원을 사용할 수 있는지 이해하도록 한다. 따라서 신뢰성 높은 로봇은 전기적 준비 상태(Electrical Readiness), 프로세서 부팅, 소프트웨어 초기화, 동작 승인(Motion Authorization)을 서로 독립된 이벤트가 아니라 하나의 통합된 과정으로 조정해야 한다.

시퀀스는 일반적으로 배터리 팩(Battery Pack)이나 외부 DC 전원(External DC Supply)과 같은 주 에너지원(Primary Energy Source)에서 시작한다. 메인 컴퓨팅 시스템이 시작되기 전에 보호 회로(Protection Circuit)는 전압, 전류, 온도, 극성 및 배터리 관리 조건(Battery-Management Condition)을 평가할 수 있다. 이후 전원 관리 하드웨어(Power-Management Hardware)는 프로세서, 메모리, 저장장치, 센서, 통신 장치에 필요한 안정화된 전원 레일(Regulated Power Rail)을 생성한다. 전원 정상 신호(Power-Good Signal)는 각 전압 도메인이 충분히 안정화되어 이후 구성요소가 리셋 상태에서 벗어날 수 있음을 나타낸다.

리셋 제어(Reset Control)는 프로세서의 전원, 클록 또는 메모리 환경이 불안정한 상태에서 코드가 실행되어서는 안 되기 때문에 전원 시퀀싱과 밀접하게 연관된다. 전원 관리 IC(Power-Management IC), 리셋 감시기(Reset Supervisor), FPGA, MCU 또는 전용 안전 컨트롤러가 필요한 전원 레일이 정의된 임계값을 충족할 때까지 메인 프로세서를 리셋 상태로 유지할 수 있다. 리셋이 해제되면 프로세서는 하드웨어에 정의된 부트 경로(Boot Path)를 시작하고 프로그래밍 가능한 부트로더에 도달하기 전에 부트 ROM(Boot ROM) 또는 리셋 벡터 코드(Reset-Vector Code)를 실행한다.

로봇 시스템은 동시에 시작되지 않는 여러 컴퓨팅 도메인(Computing Domain)을 포함하는 경우가 많다. 중앙 CPU 또는 엣지 컴퓨터(Edge Computer)는 모터 제어 MCU(Motor-Control MCU), 안전 컨트롤러, 센서 프로세서(Sensor Processor), GPU 모듈, 네트워크 스위치(Network Switch), 지능형 액추에이터(Intelligent Actuator)와 함께 동작할 수 있다. 일부 구성요소는 수 밀리초 이내에 준비되는 반면 다른 장치는 수 초가 필요할 수 있다. 따라서 시작 아키텍처(Startup Architecture)는 메인 프로세서가 부트로더에 도달했을 때 모든 주변장치가 이미 동작한다고 가정하지 말고 장치 사이의 의존성을 명시적으로 정의해야 한다.

부트로더는 GPIO 출력, 레귤레이터(Regulator), 활성화 신호(Enable Signal), 리셋 라인(Reset Line), 통신 인터페이스를 제어함으로써 전원 시퀀싱에 참여할 수 있다. 예를 들어 핵심 메모리 초기화가 완료된 이후에만 센서 전원 레일을 활성화하거나 필요한 클록이 안정화된 이후 주변장치의 리셋을 해제할 수 있다. 이러한 동작은 최소화되고 결정론적(Deterministic)이어야 하며, 부트 과정 자체에 필요한 하드웨어가 아니라면 복잡한 장치 초기화는 일반적으로 운영체제가 담당하는 것이 바람직하다.

부트로더의 하드웨어 소유권(Hardware Ownership)은 운영체제의 소유권과 명확하게 구분되어야 한다. U-Boot가 GPIO, 레귤레이터, 이더넷 컨트롤러(Ethernet Controller), 통신 버스를 구성한 후 리눅스가 동일한 자원을 다른 방식으로 초기화하면 전환 과정에서 글리치(Glitch) 또는 의도하지 않은 상태 변화가 발생할 수 있다. 부트 아키텍처는 제어권 전달 이후에도 유지되어야 하는 구성, 리눅스가 다시 초기화할 수 있는 자원, 일반적인 부트로더나 커널 동작에만 의존해서는 안 되는 안전 필수 출력(Safety-Critical Output)을 명확하게 정의해야 한다.

액추에이터 전원(Actuator Power)은 일반적인 컴퓨팅 전원보다 더욱 엄격하게 제어해야 한다. 모터, 유압 밸브(Hydraulic Valve), 조향 시스템(Steering System), 매니퓰레이터(Manipulator) 및 기타 물리적 동작을 발생시키는 장치는 펌웨어와 소프트웨어가 시작되는 동안 일반적으로 비활성 상태를 유지해야 한다. 프로세서의 성공적인 부팅 자체가 동작을 허가하는 것은 아니다. 전체 로봇 제어 시스템이 요구되는 안전 조건을 충족했음을 확인할 때까지 하드웨어 인터록(Hardware Interlock), 안전 릴레이(Safety Relay), 모터 활성화 회로(Motor-Enable Circuit), 독립 안전 컨트롤러가 안전 상태(Safe State)를 유지해야 한다.

따라서 유용한 아키텍처는 컴퓨팅 전원(Compute Power), 센서 전원(Sensor Power), 통신 전원(Communication Power), 액추에이터 전원을 논리적인 도메인으로 분리한다. 진단 및 제어 로직을 먼저 사용할 수 있도록 컴퓨팅 자원을 우선 시작하고, 이후 필수 센서와 통신 네트워크를 활성화하여 검증할 수 있다. 액추에이터 전원 또는 토크 승인(Torque Authorization)은 위치추정(Localization), 안전 통신, 제어 루프(Control Loop), 필수 감독 메커니즘(Supervision Mechanism)이 정의된 준비 상태에 도달한 이후에 수행된다.

전원 시퀀싱에서는 높은 돌입 전류(High Inrush Current)를 발생시키는 장치도 고려해야 한다. GPU, 모터 드라이브(Motor Drive), LiDAR, 네트워크 장비, 저장장치를 동시에 활성화하면 상당한 과도 전류(Transient Current)가 발생할 수 있다. 전원 활성화 시점을 단계적으로 분산하면 전압 강하(Voltage Sag)를 줄이고 보호 회로가 정상적인 시작 전류를 장애로 잘못 판단하는 것을 방지할 수 있다. 부트로더 또는 감독 컨트롤러(Supervisory Controller)가 이러한 타이밍 제어에 참여할 수 있지만, 하드 실시간 전원 보호(Hard Real-Time Power Protection)는 전용 전기 또는 안전 하드웨어에서 수행해야 한다.

통신 준비 상태(Communication Readiness)는 전원과 부트 사이에 또 다른 의존성을 형성한다. CAN, EtherCAT, Ethernet, 직렬 버스(Serial Bus) 및 기타 로봇 네트워크에는 중앙 컴퓨터가 이미 실행되는 동안 여전히 리셋 중인 장치가 연결되어 있을 수 있다. 소프트웨어는 인터페이스 컨트롤러 자체가 초기화된 상태와 해당 네트워크에 연결된 모든 노드(Node)가 실제로 동작 가능한 상태를 구분해야 한다. 부트 로직은 기본적인 통신을 구축하고 상위 수준 소프트웨어는 장치 검색(Device Discovery), 식별 확인(Identity Check), 동기화(Synchronization), 상태 검증(Health Validation)을 수행할 수 있다.

워치독 아키텍처(Watchdog Architecture)는 전체 전원 및 부트 시퀀스에 걸쳐 동작해야 한다. 하드웨어 워치독(Hardware Watchdog)은 리눅스가 시작되기 전에 동작을 시작하여 부트로더 실행이나 커널 시작 과정에서 프로세서가 멈추는 상황을 감지할 수 있다. 워치독 서비스 책임은 필요에 따라 펌웨어에서 부트로더, 커널, 최종적으로 로봇 애플리케이션으로 명시적으로 전달되어야 한다. 이러한 전환이 불명확하면 개별 소프트웨어 구성요소가 정상적으로 초기화되는 것처럼 보여도 예상하지 못한 리셋이 발생할 수 있다.

영구적인 리셋 원인 정보(Persistent Reset-Cause Information)는 부트와 전원 관리를 조정할 때 유용하다. 부트로더는 현재 시작이 정상적인 전원 인가(Power-On), 워치독 만료, 브라운아웃(Brownout), 소프트웨어 리셋(Software Reset), 열 보호(Thermal Protection), 또는 다른 하드웨어 이벤트 이후 발생했는지 확인할 수 있다. 서로 다른 리셋 원인은 서로 다른 대응을 요구할 수 있다. OTA 시험 이미지(OTA Trial Image)에서 발생한 워치독 리셋은 롤백(Rollback)을 유발할 수 있으며, 브라운아웃은 정상 동작을 다시 시도하기 전에 추가적인 전원 안정화 또는 진단 로깅(Diagnostic Logging)을 요구할 수 있다.

브라운아웃 처리는 프로세서가 완전히 전원을 잃기 전에 실행이 불안정해질 수 있는 배터리 기반 로봇에서 특히 중요하다. 전압 감시기(Voltage Supervisor)는 공급 전압이 보장된 동작 범위를 벗어나기 전에 컴퓨팅 장치를 리셋하거나 종료해야 한다. 부트로더는 중단된 기록 작업 이후 영구 저장장치(Persistent Storage)가 정상이라고 가정해서는 안 된다. A/B 파티션(A/B Partition), 트랜잭션 메타데이터(Transactional Metadata), 파일시스템 보호(Filesystem Protection), 이미지 검증(Image Verification)은 소프트웨어 업데이트나 구성 변경 중 전원이 차단되었을 때 로봇을 복구하는 데 도움을 준다.

시작 시퀀스는 하나의 부트 완료(Boot-Complete) 이벤트 대신 명시적인 준비 상태 이정표(Readiness Milestone)를 정의해야 한다. 유용한 상태에는 전원 안정(Power Stable), 프로세서 실행(Processor Running), 부트로더 검증(Bootloader Verified), 커널 실행(Kernel Running), 통신 준비(Communication Ready), 센서 준비(Sensors Ready), 위치추정 준비(Localization Ready), 안전 준비(Safety Ready), 제어 준비(Control Ready), 임무 준비(Mission Ready) 등이 포함될 수 있다. 이러한 이정표는 의존성을 관찰 가능하게 만들고 시스템 감독기가 모든 시작 지연을 일반적인 부트 실패로 취급하는 대신 로봇이 준비되지 않은 정확한 원인을 판단할 수 있도록 한다.

부트로더 진단(Bootloader Diagnostics)은 운영체제가 관찰할 수 없는 초기 전기적 문제를 확인하는 데 활용할 수 있다. GPIO 마커(GPIO Marker), 직렬 메시지(Serial Message), 전원 정상 입력(Power-Good Input), 리셋 상태 레지스터(Reset-Status Register), 레귤레이터 상태, 워치독 정보, 부트 타임스탬프(Boot Timestamp)를 이용하면 DRAM 초기화 이전, 주변장치 활성화 과정, 운영체제 제어권 전달 이후 중 어느 단계에서 장애가 발생했는지 식별할 수 있다. 이러한 신호는 애플리케이션 로깅을 사용할 수 있기 전에 로봇이 반복적으로 리셋되는 상황에서 특히 유용하다.

복구 모드(Recovery Mode)는 정상 운용과 다른 전원 정책(Power Policy)을 유지해야 한다. 펌웨어를 복구하거나 운영체제 이미지를 복원하는 로봇은 일반적으로 컴퓨팅, 저장장치, 필요에 따라 네트워크 연결은 요구하지만 액추에이터 전원은 필요하지 않다. 따라서 부트로더와 복구 환경은 진단 및 복원에 필요한 자원만 제공하면서 물리적 동작을 발생시키는 시스템을 비활성 상태로 유지해야 한다. 이를 통해 유지보수 과정에서 에너지 소비와 물리적 위험을 모두 줄일 수 있다.

OTA 업데이트(OTA Update)는 플래시 또는 eMMC 기록 도중 전원이 차단되면 후보 소프트웨어나 메타데이터가 손상될 수 있으므로 로봇의 전원 상태와 조정되어야 한다. 업데이트 정책은 설치를 시작하기 전에 충분한 배터리 충전 상태, 외부 전원, 정지 상태(Stationary Condition), 액추에이터 비활성화를 요구할 수 있다. 재부팅 이후 부트로더가 후보 이미지를 선택하더라도 정상적인 전원 시퀀싱을 계속 적용하여 새로운 소프트웨어가 부트 검증(Boot Validation)과 운영 상태 검사(Operational Health Check)를 모두 통과하기 전까지 물리적 동작을 방지해야 한다.

전원 차단 시퀀싱(Power-Off Sequencing) 역시 제어되지 않은 종료가 다음 부팅에 영향을 줄 수 있으므로 부트 아키텍처의 일부이다. 운영체제는 전원 레일이 제거되기 전에 로봇 동작을 중지하고, 액추에이터를 안전 상태로 전환하고, 파일시스템을 동기화(Synchronize)하며, 필수 메타데이터를 보존하고, 전원 관리 하드웨어에 종료 준비 상태를 알려야 한다. 감독 MCU(Supervisory MCU)는 종료 확인을 수신하거나 정의된 타임아웃(Timeout)이 만료될 때까지 최종 전원 차단을 지연할 수 있다. 이를 통해 전원 인가부터 운용, 통제된 전원 종료까지 조정된 수명주기(Lifecycle)를 구축할 수 있다.

장애 처리(Fault Handling)는 반복적이고 통제되지 않은 재시작보다 안전한 성능 저하 상태(Safe Degraded State)를 우선해야 한다. 필수 센서, 안전 컨트롤러 또는 전원 도메인이 허용된 시간 내에 준비되지 않으면 시스템은 임무 실행을 차단한 상태에서 진단을 계속할 수 있다. 장애 유형에 따라 감독 시스템은 주변장치를 다시 시도하거나, 중요하지 않은 전원 도메인을 재인가(Power Cycle)하거나, 컴퓨팅 서브시스템을 재부팅하거나, 복구 모드에 진입하거나, 액추에이터를 활성화하지 않은 상태에서 정비를 요구할 수 있다.

타이밍 최적화(Timing Optimization)는 이러한 안전 의존성을 유지해야 한다. 측정을 통해 하드웨어가 이미 안정된 것으로 확인된 경우 지연 시간을 단축하는 것은 유용하지만 전원 안정화 간격(Power-Settling Interval)이나 리셋 타이밍을 임의로 제거하면 재현하기 어려운 간헐적 장애(Intermittent Failure)가 발생할 수 있다. 불필요하게 긴 고정 지연(Fixed Delay)보다 전원 정상 신호, 장치 사양(Device Specification), 측정된 시작 동작, 명시적인 핸드셰이크(Explicit Handshake)를 활용하는 것이 바람직하며, 이를 통해 결정론적인 하드웨어 준비 상태를 유지하면서 시작 시간을 단축할 수 있다.

따라서 견고한 로봇 시작 아키텍처(Robot Startup Architecture)는 에너지 입력(Energy Input), 안정화된 전원 도메인, 리셋 제어, 부트 ROM, 부트로더, 운영체제, 로봇 서비스, 안전 검증(Safety Validation), 액추에이터 승인(Actuator Authorization)을 포괄하는 통합된 상태 머신(Coordinated State Machine)을 형성한다. 부트로더는 하드웨어 전원 상태와 소프트웨어 실행을 연결하는 중요한 가교 역할을 하지만 물리적 안전은 독립적으로 강제되어야 한다. 이러한 계층을 통합함으로써 예측 가능한 시작, 진단 가능한 장애, 안전한 복구, 신뢰성 높은 OTA 동작, 로봇의 물리적 움직임으로 이어지는 통제된 전환을 구현할 수 있다.

## 04.10 Bootloader Security Vulnerabilities and Countermeasures

![](images/image10.png){width="7.268055555555556in" height="7.268055555555556in"}

부트로더 보안(Bootloader Security)은 부트로더(Bootloader)가 운영체제(Operating System)보다 먼저 실행되고 이후의 모든 구성요소가 동작할 소프트웨어 환경을 구축하기 때문에 매우 중요하다. 이 단계의 취약점(Vulnerability)은 공격자가 운영체제의 보호 기능을 우회하고, 신뢰된 소프트웨어를 변경하며, 민감한 정보를 추출하거나 지속적인 특권 제어(Persistent Privileged Control)를 획득하도록 만들 수 있다. 임베디드 및 로봇 시스템에서는 이러한 침해가 궁극적으로 센서, 통신 인터페이스, 안전 기능 및 물리적 액추에이터(Actuator)에까지 영향을 미칠 수 있다.

부트로더는 시스템 아키텍처(System Architecture)에서 매우 강력한 위치를 차지한다. 일반적으로 제한되지 않은 프로세서 권한(Processor Privilege)으로 실행되며 플래시 메모리(Flash Memory), RAM, 하드웨어 레지스터(Hardware Register), 암호화 엔진(Cryptographic Engine), 부트 구성(Boot Configuration), 주변장치 인터페이스(Peripheral Interface)에 접근할 수 있다. 많은 운영체제 보안 메커니즘이 아직 활성화되지 않은 단계이므로 부트로더 수준의 실행 권한을 획득한 공격자는 일반적인 엔드포인트 방어(Endpoint Defense)보다 하위 계층에서 동작하면서 운영체제를 재설치한 이후에도 유지되는 변경을 구축할 수 있다.

주요 취약점 중 하나는 승인되지 않은 부트 이미지(Unauthorized Boot Image)의 실행이다. 부트로더가 임의의 커널, 펌웨어, 복구 이미지(Recovery Image), 운영체제 로더를 허용하면 물리적 접근이나 업데이트 경로에 대한 원격 접근 권한을 획득한 공격자가 정상 소프트웨어를 악성 코드로 교체할 수 있다. 보안 부트(Secure Boot)는 실행 이미지로 제어권을 전달하기 전에 암호학적 인증(Cryptographic Authentication)을 요구하여 이러한 위협에 대응하며, 변경 불가능한 하드웨어 신뢰 루트(Hardware Root of Trust)에서 이후 소프트웨어 단계까지 신뢰 체인(Chain of Trust)을 구축한다.

취약한 서명 검증(Weak Signature Verification)은 암호화 기능이 활성화되어 있는 것처럼 보이더라도 보안 부트를 무력화할 수 있다. 잘못된 파싱(Parsing), 불완전한 이미지 검증 범위, 부적절한 오류 처리, 취약한 알고리즘 또는 비정상적인 메타데이터(Malformed Metadata)의 허용은 인증을 우회할 수 있는 경로를 만들 수 있다. 검증 코드는 실패 시 차단(Fail Closed)되어야 하며 모든 보안 관련 이미지 영역을 인증하고 승인된 암호 알고리즘과 매개변수를 사용하며 신뢰되지 않은 코드가 실행되기 전에 모호하거나 부분적으로만 검증된 구조를 거부해야 한다.

개인 서명 키(Private Signing Key)의 탈취는 올바르게 구현된 검증 기능만으로는 합법적인 소프트웨어와 탈취된 승인 키로 서명된 악성 소프트웨어를 구별할 수 없다는 점에서 다른 유형의 위협을 발생시킨다. 따라서 제품용 서명 키(Production Signing Key)는 개발 시스템과 분리하고 통제된 서명 인프라(Controlled Signing Infrastructure), 필요한 경우 하드웨어 보안 모듈(Hardware Security Module, HSM), 접근 제어(Access Control), 감사(Auditing), 키 교체 절차(Key-Rotation Procedure)를 통해 보호해야 한다. 개발, 제품, 복구를 위한 신뢰 도메인(Trust Domain) 역시 의도적으로 분리해야 한다.

롤백 공격(Rollback Attack)은 정상적으로 서명되어 있지만 알려진 취약점을 포함하는 이전 소프트웨어를 악용한다. 공격자는 현재 이미지를 이전에 승인된 릴리스로 교체한 다음 이미 수정된 과거의 취약점을 이용할 수 있다. 롤백 방지(Anti-Rollback Protection)는 신뢰된 버전 정보(Trusted Version Information), 단조 증가 카운터(Monotonic Counter), 보안 버전 번호(Security-Version Number), 보호된 퓨즈(Protected Fuse)를 사용하여 허용 가능한 최소 소프트웨어 버전을 설정한다. 따라서 서명의 유효성과 버전의 승인 여부를 함께 평가해야 한다.

부트 구성 조작(Boot Configuration Manipulation)도 중요한 공격 표면(Attack Surface)이다. U-Boot 환경 변수(Environment Variable), UEFI 부트 항목(Boot Entry), 커널 명령줄 매개변수(Kernel Command-Line Parameter), 파티션 선택자(Partition Selector), 복구 플래그(Recovery Flag)는 어떤 소프트웨어를 실행할지와 보안 제어를 어떻게 적용할지를 변경할 수 있다. 이러한 값을 승인 없이 변경할 수 있다면 공격자가 부트를 대체 이미지로 전환하거나 보호 기능을 비활성화하고 루트 파일시스템을 변경하거나 특권 유지보수 모드(Privileged Maintenance Mode)에 진입할 수 있다. 따라서 보안에 민감한 부트 구성은 인증하거나 보호하고 엄격하게 제한해야 한다.

대화형 부트로더 콘솔(Interactive Bootloader Console)은 강력한 관리자 명령을 노출할 수 있다. 개발용 U-Boot 셸(U-Boot Shell)은 임의의 메모리 읽기 및 쓰기, 플래시 수정, 환경 변수 변경, 네트워크 로딩, 장치 검사, 지정된 주소에서의 직접 실행 등을 허용할 수 있다. 이러한 기능은 개발 과정에서는 유용하지만 제품 환경에서는 위험하다. 제품 시스템에서는 불필요한 콘솔 접근을 비활성화하고 사용 가능한 명령을 제한하며 서비스 접근이 필요한 경우 인증을 적용하고 승인되지 않은 사용자가 정상 부트 경로를 중단하지 못하도록 해야 한다.

JTAG 및 SWD와 같은 디버그 인터페이스(Debug Interface) 역시 제한 없이 남겨두면 소프트웨어 보안을 우회할 수 있다. 이러한 인터페이스에 물리적으로 접근하면 프로세서 제어, 메모리 검사, 플래시 프로그래밍, 중단점(Breakpoint) 삽입 또는 민감한 정보 추출이 가능할 수 있다. 제품 수명주기 구성(Product Lifecycle Configuration)에서는 디버그 접근을 비활성화할지, 영구적으로 잠글지, 암호학적으로 인증할지 또는 통제된 서비스 절차에서만 허용할지를 명확하게 결정해야 한다. 단순히 커넥터를 장착하지 않는 것만으로 충분한 보안 경계(Security Boundary)가 형성된다고 간주해서는 안 된다.

대체 부트 소스(Alternative Boot Source)는 숨겨진 우회 경로를 만들 수 있다. USB, SD 카드, 네트워크 PXE, UART 다운로드 모드(UART Download Mode), 복구 포트(Recovery Port), 제조사별 직렬 부트 메커니즘(Serial Boot Mechanism)은 정상적인 신뢰 부트 시퀀스를 따르지 않고 소프트웨어를 실행하도록 만들 가능성이 있다. 프로세서와 보드가 지원하는 모든 부트 소스를 위협 모델(Threat Model)에 포함해야 한다. 제품 구성에서는 불필요한 소스를 비활성화하거나 허용되는 모든 소스에 동일한 수준의 인증 및 권한 부여 요구사항(Authentication and Authorization Requirement)을 적용해야 한다.

복구 메커니즘(Recovery Mechanism)은 정상적인 부트가 실패했을 때 의도적으로 동작하기 때문에 특별한 주의가 필요하다. 제한되지 않은 복구 이미지, 서명되지 않은 업데이트 인터페이스(Unsigned Update Interface), 특권 유지보수 명령은 기본 부트 경로보다 더 쉽게 악용될 수 있다. 따라서 복구 소프트웨어도 동일한 신뢰 체인 내부에 유지하고 인증된 교체 이미지만 허용하며 명령과 인터페이스를 제한하고 액추에이터 안전 상태(Actuator-Safe State)를 유지해야 한다. 복구 과정은 가용성을 회복하기 위해 보안을 중단하는 것이 아니라 신뢰를 복원해야 한다.

부트로더 코드 내부의 메모리 안전 취약점(Memory-Safety Vulnerability)도 특권 실행으로 이어지는 또 다른 경로를 제공할 수 있다. 부트로더는 운영체제가 활성화되기 전에 파일시스템 구조, 네트워크 패킷, 장치 디스크립터(Device Descriptor), 환경 데이터, 이미지 헤더(Image Header), 압축 콘텐츠, 업데이트 패키지를 파싱하는 경우가 많다. 따라서 버퍼 오버플로(Buffer Overflow), 정수 오류(Integer Error), 해제 후 사용(Use-After-Free), 비정상 입력 처리 실패가 보안상 치명적일 수 있다. 파서 복잡성(Parser Complexity)을 줄이고 외부에서 제어되는 모든 길이와 데이터 구조를 검증하면 공격 표면을 줄일 수 있다.

네트워크 기능을 지원하는 부트로더(Network-Enabled Bootloader)는 추가적인 방어 설계를 필요로 한다. TFTP, DHCP, HTTP, 원격 콘솔(Remote Console), 사용자 정의 업데이트 프로토콜(Custom Update Protocol)은 부트 단계의 코드를 잠재적으로 적대적인 네트워크 트래픽에 노출할 수 있다. 제품 시스템에서는 배포 또는 복구에 실제로 필요한 프로토콜만 활성화하고 격리된 로봇 네트워크가 항상 신뢰할 수 있다고 가정해서는 안 된다. 인증된 업데이트 채널(Authenticated Update Channel), 엄격한 입력 검증, 제한된 엔드포인트(Restricted Endpoint), 최소화된 운영체제 이전 네트워킹(Pre-OS Networking)을 통해 원격 부트로더 공격 가능성을 줄여야 한다.

민감한 정보(Sensitive Information)가 부트로더 인터페이스를 통해 유출될 가능성도 있다. 메모리 덤프 명령(Memory-Dump Command), 상세 로그, 충돌 출력(Crash Output), 환경 변수, 제조 데이터, 적절하게 삭제되지 않은 RAM은 키, 자격증명(Credential), 장치 식별 정보(Device Identity), 주소, 보안 구성을 노출할 수 있다. 따라서 제품용 로깅(Production Logging)은 진단 요구사항과 정보 노출 위험 사이의 균형을 유지해야 한다. 비밀 정보는 가능한 경우 하드웨어 기반 보호 영역(Hardware-Backed Protected Location)에 저장하고 유지보수 명령이나 로그를 통해 불필요하게 출력하거나 노출해서는 안 된다.

플래시 및 영구 저장장치 보호(Persistent-Storage Protection) 역시 중요하다. 공격자가 부트로더 자체 또는 신뢰된 메타데이터(Trusted Metadata)를 변경하려고 시도할 수 있기 때문이다. 하드웨어 쓰기 보호(Hardware Write Protection), 보호된 플래시 영역(Protected Flash Region), 인증된 업데이트, 저장장치 잠금(Storage Locking), 권한 분리(Privilege Separation)를 통해 승인되지 않은 변경을 제한할 수 있다. 활성 슬롯 선택, 부트 카운터(Boot Counter), 신뢰 버전(Trusted Version), 복구 상태와 같은 핵심 메타데이터도 예상하지 못한 전원 손실에 대한 복원력을 유지하면서 손상과 조작으로부터 보호해야 한다.

결함 주입(Fault Injection)은 일반적인 소프트웨어 검증만으로 대응하기 어려운 위협을 발생시킨다. 전압 글리치(Voltage Glitch), 클록 조작(Clock Manipulation), 전자기적 교란(Electromagnetic Disturbance), 정밀하게 조정된 리셋을 통해 부트 과정에서 프로세서가 검증 절차를 건너뛰거나 보안 결정을 손상시킬 가능성이 있다. 높은 보증 수준(High Assurance)이 필요한 시스템에서는 전압 및 클록 모니터링, 리셋 감시(Reset Supervision), 하드웨어 보안 기능, 중복 검증(Redundant Check), 실패 안전 검증 로직(Fail-Safe Verification Logic)을 결합할 수 있다. 보안에 중요한 결정은 하나의 취약한 비교 연산이나 일시적인 소프트웨어 상태에만 의존해서는 안 된다.

검사 시점과 사용 시점(Time-of-Check to Time-of-Use, TOCTOU) 문제는 이미지가 검증된 이후 실행되기 전에 변경되는 경우 발생할 수 있다. 부트 아키텍처는 인증과 실제 사용 사이의 시간 및 변경 가능한 저장공간을 최소화해야 한다. 검증된 이미지는 통제된 메모리(Controlled Memory)에 로드하고 지원되는 경우 관련 메모리 영역을 보호하며 DMA 가능 주변장치(DMA-Capable Peripheral)가 신뢰된 실행 콘텐츠를 예상하지 못하게 변경하지 못하도록 해야 한다. 지원 가능한 플랫폼에서는 IOMMU 및 하드웨어 접근 제어 메커니즘(Hardware Access-Control Mechanism)을 통해 이러한 경계를 강화할 수 있다.

부트로더 자체의 업데이트도 상당한 위험을 발생시킨다. 실패하거나 악의적인 업데이트가 이후 모든 소프트웨어를 검증하는 구성요소 자체를 손상시킬 수 있기 때문이다. 따라서 부트로더 업데이트 패키지는 반드시 인증되고 신중하게 통제된 절차를 통해 설치되어야 한다. 중복 부트로더 복사본(Redundant Bootloader Copy), 변경 불가능한 1단계 로더(Immutable First-Stage Loader), 하드웨어 복구 경로(Hardware Recovery Path), 트랜잭션 메타데이터, 전원 장애 안전 업데이트 메커니즘(Power-Failure-Safe Update Mechanism)을 통해 업데이트 중단으로 플랫폼이 영구적으로 작동하지 못하게 될 가능성을 줄일 수 있다.

로봇 플랫폼에서는 부트 보안 실패(Boot Security Failure)를 물리적 안전 정책(Physical Safety Policy)과 연결해야 한다. 인증 실패, 손상된 펌웨어, 예상하지 못한 롤백 또는 잘못된 부트 메타데이터가 무제한적인 액추에이터 동작으로 이어져서는 안 된다. 모터 활성화(Motor Enable), 안전 릴레이, 브레이크 및 기타 동작 승인 메커니즘(Motion-Authorizing Mechanism)은 신뢰된 소프트웨어의 초기화가 완료되고 안전 준비 상태(Safety Readiness)가 독립적으로 확립될 때까지 정의된 안전 상태를 유지해야 한다. 따라서 보안 실패는 로봇을 불확실한 동작이 아니라 안전한 복구(Safe Recovery) 상태로 전환해야 한다.

보안 테스트(Security Testing)는 정상적인 부트 동작뿐 아니라 적대적 경로(Adversarial Path)를 검증해야 한다. 손상된 서명, 비정상 이미지(Malformed Image), 조작된 환경 변수, 승인되지 않은 복구 요청, 오래된 서명 소프트웨어, 중단된 업데이트, 비활성화된 저장장치, 반복적인 워치독 리셋, 예상하지 못한 이동식 미디어(Removable Media), 디버그 인터페이스 접근 시도 등을 테스트해야 한다. 이미지 파서(Image Parser)와 프로토콜 처리기(Protocol Handler)에 대한 퍼징(Fuzzing)을 수행하면 일반적인 기능 테스트에서는 실행되기 어려운 취약점을 발견할 수 있다.

부트로더는 제품 출시 이후 변경하지 않는 엔지니어링 기반으로 취급하는 것이 아니라 제품 보안 수명주기(Product Security Lifecycle)의 일부로 지속적으로 관리해야 한다. 취약점 모니터링(Vulnerability Monitoring), 의존성 추적(Dependency Tracking), 보안 권고(Security Advisory), 암호 정책 변경(Cryptographic-Policy Change), 키 폐기(Key Revocation), 통제된 펌웨어 업데이트, 회귀 테스트(Regression Testing)가 전체 배포 기간 동안 필요하다. 제품 출시 당시 안전한 설계라도 새롭게 발견된 취약점을 패치할 수 없거나 영향을 받은 신뢰 자격증명(Trust Credential)을 교체할 수 없다면 시간이 지나면서 취약한 시스템으로 변할 수 있다.

따라서 견고한 대응 전략(Countermeasure Strategy)은 하드웨어 신뢰 루트(Hardware Root of Trust), 보안 부트(Secure Boot), 롤백 방지(Anti-Rollback Protection), 보호된 키(Protected Key), 제한된 부트 구성(Constrained Boot Configuration), 제한된 디버그 및 콘솔 접근, 인증된 복구(Authenticated Recovery), 메모리 안전 설계 방식(Memory-Safe Design Practice), 보호된 영구 저장장치, 보안 업데이트 메커니즘(Secure Update Mechanism)을 결합한다. 로봇 시스템에서는 이러한 보호 기능을 독립적인 안전 제어(Independent Safety Control)와 추가로 통합하여 손상되었거나 검증할 수 없는 소프트웨어가 물리적 움직임에 대한 제어 권한을 절대로 획득하지 못하도록 해야 한다.
