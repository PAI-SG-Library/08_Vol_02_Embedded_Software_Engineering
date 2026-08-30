**Volume 02 Embedded Software Engineering**

# 11. Embedded Security

## 11.01 Embedded Security Threat Model and Attack Vectors

![](images/image1.png){width="7.268055555555556in" height="7.268055555555556in"}

임베디드 보안(Embedded Security)은 무엇을 보호해야 하는지, 누가 시스템을 공격할 수 있는지, 공격자가 어떤 경로를 통해 접근할 수 있는지, 그리고 공격으로 인해 어떤 결과가 발생할 수 있는지를 정의하는 위협 모델(Threat Model)에서 시작한다. 로봇 또는 임베디드 제어기(Embedded Controller)에서 보호해야 할 자산(Asset)에는 펌웨어(Firmware), 암호화 키(Cryptographic Key), 구성 데이터(Configuration Data), 센서 측정값(Sensor Measurement), 액추에이터 명령(Actuator Command), 진단 정보(Diagnostic Information), 통신 자격 증명(Communication Credential), 부팅 과정(Boot Process)의 무결성(Integrity) 등이 포함된다.

일반적인 정보 시스템(Information System)과 달리 임베디드 장치(Embedded Device)는 물리적 프로세스(Physical Process)와 직접 상호작용한다. 따라서 성공적인 공격은 기밀성(Confidentiality)과 데이터 무결성(Data Integrity)뿐만 아니라 모션(Motion), 전력 제어(Power Control), 센싱(Sensing), 제동(Braking), 조작(Manipulation), 비상 기능(Emergency Function)에도 영향을 줄 수 있다. 특히 임베디드 제어기가 모터, 배터리, 안전 인터페이스(Safety Interface), 자율 로봇 기능(Autonomous Robotic Function)을 제어하는 경우 사이버 영향(Cyber Impact)과 물리적 영향(Physical Impact)을 함께 고려해야 한다.

실용적인 위협 모델(Threat Model)은 신뢰 경계(Trust Boundary)를 정의하는 것에서 시작한다. MCU, 보안 저장소(Secure Storage), 부트로더(Bootloader), 운영체제(Operating System), 애플리케이션 펌웨어(Application Firmware), 외부 센서(External Sensor), 통신 버스(Communication Bus), 유지보수 인터페이스(Maintenance Interface), 엣지 컴퓨터(Edge Computer), 원격 서버(Remote Server)는 서로 다른 신뢰 영역(Trust Domain)에 속할 수 있다. 이러한 영역 사이의 모든 전환 지점은 데이터, 명령, 실행 코드, 신원 정보 또는 자격 증명을 신뢰하기 전에 인증(Authentication)하고 검증(Validation)해야 하는 공격 표면(Attack Surface)을 형성한다.

물리적 접근(Physical Access)은 배포된 장비가 보호된 데이터센터 외부에 위치하는 경우가 많기 때문에 가장 중요한 임베디드 공격 벡터(Embedded Attack Vector) 중 하나이다. JTAG 및 SWD와 같은 디버그 인터페이스(Debug Interface), UART 콘솔(Console), 이동식 저장장치(Removable Storage), 노출된 테스트 패드(Test Pad), USB 포트, 프로그래밍 커넥터(Programming Connector)는 높은 권한의 접근 경로를 제공할 수 있다. 공격자는 이러한 인터페이스를 통해 펌웨어 추출(Firmware Extraction), 메모리 검사(Memory Inspection), 구성 변경(Configuration Modification), 자격 증명 복구(Credential Recovery), 비인가 재플래싱(Unauthorized Reflashing)을 시도할 수 있다.

펌웨어(Firmware) 자체도 주요 공격 표면(Attack Surface)을 구성한다. 메모리 손상(Memory Corruption), 버퍼 오버플로(Buffer Overflow), 정수 오류(Integer Error), 안전하지 않은 포인터 연산(Unsafe Pointer Operation), 명령 주입(Command Injection), 안전하지 않은 역직렬화(Insecure Deserialization), 불충분한 입력 검증(Input Validation), 부적절한 권한 처리(Privilege Handling)는 비정상적인 입력을 의도하지 않은 코드 실행으로 연결할 수 있다. 따라서 임베디드 소프트웨어(Embedded Software)는 외부에서 제공되는 패킷, 센서 메시지, 파일, 구성 값, 유지보수 명령이 단순히 잘못된 것이 아니라 의도적으로 악의적일 수 있다고 가정해야 한다.

통신 인터페이스(Communication Interface)는 공격자가 물리적으로 장치에 접근하지 않더라도 위협 모델(Threat Model)의 범위를 확장시킨다. Ethernet, Wi-Fi, Bluetooth, CAN, UART, SPI, I2C, EtherCAT, 셀룰러 링크(Cellular Link), 독자적인 무선 채널(Proprietary Radio Channel)은 각각 서로 다른 기밀성, 무결성, 인증, 가용성(Availability) 위험을 노출할 수 있다. 공격자는 도청(Eavesdropping), 스푸핑(Spoofing), 재전송 공격(Replay Attack), 메시지 변조(Message Modification), 세션 하이재킹(Session Hijacking), 비인가 명령 주입(Unauthorized Command Injection), 서비스 거부 공격(Denial-of-Service)을 시도할 수 있다.

로봇 내부 네트워크(Robot Internal Network)는 기존 필드버스(Fieldbus)가 기본적으로 제한된 보안 기능만 제공하는 경우가 많기 때문에 특별한 주의가 필요하다. 공격자가 신뢰된 네트워크 구간(Trusted Network Segment)에 접근하면 위조된 센서 값이나 액추에이터 명령이 수신 제어기에 정상적인 메시지처럼 보일 수 있다. 따라서 네트워크상의 위치만으로 신원(Identity)이 증명된다고 가정해서는 안 된다. 메시지 인증(Message Authentication), 최신성 확인(Freshness Checking), 순서 검증(Sequence Validation), 권한 부여(Authorization), 네트워크 분할(Segmentation), 동작 타당성 검사(Behavioral Plausibility Check)를 추가적인 방어 계층으로 적용해야 한다.

부팅 체인(Boot Chain)과 업데이트 체인(Update Chain)은 일반적인 애플리케이션 수준의 방어보다 하위 계층에서 지속적인 공격 기반을 형성할 수 있기 때문에 가치가 높은 공격 대상이다. 변조된 부트로더(Modified Bootloader), 서명되지 않은 펌웨어(Unsigned Firmware), 취약한 이전 버전으로의 롤백(Rollback), 악성 복구 이미지(Malicious Recovery Image), 손상된 OTA 패키지는 장치에 장기간 지속되는 제어권을 제공할 수 있다. 위협 모델은 변경 불가능하거나 하드웨어로 보호되는 신뢰 앵커(Trust Anchor)에서 부팅 검증(Boot Verification), 펌웨어 로딩(Firmware Loading), 업데이트 설치(Update Installation), 롤백, 복구(Recovery)에 이르는 전체 체인을 추적해야 한다.

키 관리(Key Management)가 취약하면 암호화 보호(Cryptographic Protection)도 실패할 수 있다. 하드코딩된 비밀번호(Hard-coded Password), 여러 장치가 공유하는 플릿 자격 증명(Fleet Credential), 노출된 개인 키(Private Key), 예측 가능한 난수(Predictable Random Number), 보호되지 않은 인증서(Certificate), 안전하지 않은 키 프로비저닝(Key Provisioning), 읽을 수 있는 플래시 메모리에 직접 저장된 자격 증명은 강력한 암호화 알고리즘의 효과를 무력화할 수 있다. 따라서 공격 분석(Attack Analysis)은 제조, 프로비저닝, 저장, 사용, 교체(Rotation), 폐기(Revocation), 서비스 작업, 수명 종료(End-of-Life)에 이르는 전체 자격 증명 수명주기(Credential Lifecycle)를 검토해야 한다.

공급망 위협(Supply-Chain Threat)은 배포된 장치의 범위를 넘어 보안 분석(Security Analysis)을 확장시킨다. 손상된 서드파티 라이브러리(Third-Party Library), 악성 빌드 도구(Build Tool), 취약한 의존성(Dependency), 변조된 펌웨어 패키지, 위조 부품(Counterfeit Component), 유출된 서명 자격 증명(Signing Credential), 제조 과정에서의 비인가 변경은 배포 이전부터 취약점을 시스템에 삽입할 수 있다. 소프트웨어 출처(Provenance), 통제된 빌드 환경(Build Environment), 서명된 산출물(Signed Artifact), 의존성 목록, 소프트웨어 자재 명세서(SBOM), 추적 가능한 릴리스 프로세스(Release Process)를 유지하면 숨겨진 침해 가능성을 줄일 수 있다.

가용성 공격(Availability Attack)은 실시간 임베디드 시스템(Real-Time Embedded System)에서 특히 중요하다. 네트워크 플러딩(Network Flooding), 인터럽트 폭주(Interrupt Storm), 비정상 패킷 반복 처리, 자원 고갈(Resource Exhaustion), 저장공간 포화(Storage Saturation), 워치독 조작(Watchdog Manipulation), 과도한 로깅(Excessive Logging), 계산 비용이 높은 요청은 중요한 태스크(Task)가 요구되는 타이밍을 충족하지 못하게 만들 수 있다. 따라서 보안 제어(Security Control) 자체가 허용할 수 없는 지연시간(Latency), 우선순위 역전(Priority Inversion), 메모리 압박(Memory Pressure), 결정론적 제어 동작(Deterministic Control Behavior)의 손실을 발생시키지 않도록 설계해야 한다.

위협 모델링(Threat Modeling)은 서로 다른 능력과 목적을 가진 공격자(Attacker)도 고려해야 한다. 기회주의적 공격자(Opportunistic Attacker)는 외부에 노출된 서비스를 악용할 수 있으며, 전문적인 공격자는 하드웨어, 펌웨어 아키텍처(Firmware Architecture), 통신 프로토콜(Communication Protocol), 유지보수 절차(Maintenance Procedure)를 상세하게 이해할 수 있다. 내부자(Insider) 또는 침해된 서비스 도구(Compromised Service Tool)는 이미 합법적인 접근 권한을 보유할 수도 있다. 공격자 능력, 필요한 접근 수준, 공격 복잡도(Attack Complexity), 탐지 가능성(Detectability), 지속성(Persistence), 잠재적인 물리적 영향을 구분하면 현실적인 위협의 우선순위를 결정할 수 있다.

위험 평가(Risk Evaluation)는 각각의 공격 경로(Attack Path)를 보호 자산과 시스템 결과에 연결한다. 효과적인 분석은 진입 지점(Entry Point)에서 시작하여 취약점(Vulnerability), 권한 변화(Privilege Change), 측면 이동(Lateral Movement), 영향을 받는 기능을 거쳐 최종적인 피해까지 추적한다. 로봇 시스템에서는 데이터 유출(Data Disclosure), 가용성 상실(Loss of Availability), 잘못된 인식(Incorrect Perception), 비인가 이동(Unauthorized Movement), 위치추정 성능 저하(Degraded Localization), 액추에이터 동작 조작, 안전 기능(Safety Function) 비활성화, 지속적인 플릿 침해(Fleet Compromise) 등이 최종 결과가 될 수 있다.

보안 제어(Security Control)는 하나의 메커니즘에 의존하지 않고 심층 방어(Defense-in-Depth) 원칙을 따라야 한다. 하드웨어 신뢰 루트(Hardware Root of Trust), 보안 부팅(Secure Boot), 메모리 보호(Memory Protection), 최소 권한(Least Privilege), 인증된 통신(Authenticated Communication), 안전한 키 저장(Secure Key Storage), 서명된 업데이트(Signed Update), 네트워크 분할(Network Segmentation), 입력 검증(Input Validation), 런타임 모니터링(Runtime Monitoring), 속도 제한(Rate Limiting), 로깅(Logging), 통제된 디버그 접근(Controlled Debug Access)은 공격의 서로 다른 단계에 대응한다. 목표는 가능한 경우 침해를 방지하면서, 침해가 발생했을 때에는 확산을 제한하고 분석에 필요한 증거를 보존하는 것이다.

위협 모델(Threat Model)은 초기 설계 단계에서 한 번 작성하고 끝나는 문서가 아니라 지속적으로 갱신되는 엔지니어링 산출물(Living Engineering Artifact)이어야 한다. 새로운 펌웨어, 인터페이스, 센서, 통신 프로토콜, 서드파티 패키지, OTA 메커니즘, 서비스 절차가 추가될 때마다 새로운 신뢰 경계와 공격 경로가 만들어질 수 있다. 따라서 아키텍처가 변경되거나 새로운 취약점이 발견되거나 실제 운영 과정에서 예상하지 못한 동작이 확인될 때마다 보안 가정을 재검토하고, 위협 모델링을 개발(Development), 시험(Testing), 배포(Deployment), 모니터링(Monitoring), 사고 대응(Incident Response)의 전체 수명주기와 지속적으로 연결해야 한다.

## 11.02 Hardware Security Module (HSM) and TPM Usage

![](images/image2.png){width="7.268055555555556in" height="7.268055555555556in"}

하드웨어 보안 모듈(Hardware Security Module, HSM)과 신뢰 플랫폼 모듈(Trusted Platform Module, TPM)은 일반적인 애플리케이션 소프트웨어(Application Software)만으로는 안정적으로 제공하기 어려운 암호화 연산(Cryptographic Operation)과 키 관리(Key Management)를 위한 보호된 환경을 제공한다. 임베디드 및 로봇 시스템(Embedded and Robotic System)에서는 장치 신원(Device Identity), 보안 부팅(Secure Boot), 암호화 통신(Encrypted Communication), 펌웨어 검증(Firmware Verification), 자격 증명 보호(Credential Protection), 시스템 증명(System Attestation)을 위한 하드웨어 기반 신뢰(Hardware-backed Trust)를 구축한다. 핵심 목적은 상위 소프트웨어 계층이 침해되더라도 중요한 비밀 정보가 격리된 상태로 유지되도록 하는 것이다.

하드웨어 보안 모듈(Hardware Security Module, HSM)은 보호된 하드웨어 경계(Protected Hardware Boundary) 내부에서 암호화 키(Cryptographic Key)를 생성하고 저장하며 사용하도록 설계된 전용 보안 구성요소(Security Component)이다. 개인 키(Private Key)를 외부로 내보낼 수 없는 형태(Non-exportable)로 유지할 수 있으므로 애플리케이션은 키 자체를 전달받지 않고 서명(Signing), 복호화(Decryption), 인증(Authentication) 연산을 요청한다. 이러한 구조는 민감한 키를 일반 파일, 구성 데이터(Configuration Data), 읽기 가능한 플래시 메모리(Flash Memory)에 저장하는 방식보다 노출 위험을 크게 줄인다.

신뢰 플랫폼 모듈(Trusted Platform Module, TPM)은 플랫폼 무결성(Platform Integrity)과 밀접하게 연계된 표준화된 하드웨어 기반 보안 기능을 제공한다. TPM은 일반적으로 보호된 키 생성(Protected Key Generation), 암호화 연산, 난수 생성(Random-number Generation), 플랫폼 측정(Platform Measurement), 봉인 저장소(Sealed Storage), 증명(Attestation)을 지원한다. 임베디드 리눅스 컴퓨터(Embedded Linux Computer), 산업용 PC(Industrial PC), 엣지 AI 시스템(Edge AI System), 로봇 메인 컴퓨터(Robot Main Computer)는 개별 TPM(Discrete TPM) 또는 플랫폼에 통합된 구현을 사용하여 측정 가능하고 검증 가능한 시스템 신원(System Identity)을 구축할 수 있다.

HSM과 TPM의 차이는 단순한 암호화 기능의 차이보다는 주된 사용 목적(Intended Role)에 있다. HSM은 안전한 키 보관(Secure Key Custody)과 암호화 서비스(Cryptographic Service)에 중점을 두는 반면, TPM은 플랫폼 신뢰(Platform Trust), 측정 부팅(Measured Boot), 보호된 자격 증명(Protected Credential), 증명(Attestation)에 중점을 둔다. 시스템 아키텍처(System Architecture)에 따라 임베디드 장치는 보안 요소(Secure Element), TPM, MCU 보안 서브시스템(MCU Security Subsystem), 외부 HSM을 사용할 수 있으며, 이러한 기술을 반드시 상호 배타적인 대안으로 볼 필요는 없다.

가능한 경우 키 생성(Key Generation)은 보호된 하드웨어 내부에서 수행되어야 한다. 개인 키(Private Key)를 외부에서 생성한 후 보안 하드웨어로 복사하면 추가적인 프로비저닝 과정(Provisioning Process)이 새로운 공격 표면(Attack Surface)이 된다. 반면 하드웨어에서 생성된 장치 키(Device Key)는 영구적으로 보호하면서 해당 공개 키(Public Key) 또는 인증서(Certificate)만 제조, 플릿(Fleet), OTA, 클라우드 인프라(Cloud Infrastructure)에 등록할 수 있다. 이를 통해 각각의 로봇이나 제어기에 고유한 암호학적 신원(Cryptographic Identity)을 부여할 수 있다.

보호된 키(Protected Key)는 전체 운영 수명주기(Operational Lifecycle)에 걸쳐 장치 인증(Device Authentication)을 지원할 수 있다. 로봇이 온프레미스 서버(On-premise Server), 플릿 관리 시스템(Fleet Management System), 유지보수 스테이션(Maintenance Station), 클라우드 서비스(Cloud Service)에 연결될 때 보안 하드웨어 내부에서 암호화 연산을 수행하여 개인 키의 소유를 증명할 수 있다. 원격 시스템은 해당 인증서 또는 공개 키를 검증하므로 공유 비밀번호(Shared Password)에 대한 의존성을 줄이고 일반 애플리케이션 저장소에서 재사용 가능한 자격 증명이 추출되는 것을 방지할 수 있다.

하드웨어 기반 신뢰(Hardware-backed Trust)는 보안 부팅(Secure Boot)과 펌웨어 검증(Firmware Verification)에서 특히 중요하다. 부트로더(Bootloader)는 보호된 하드웨어 또는 변경 불가능한 저장소(Immutable Storage)에 고정된 신뢰할 수 있는 공개 키나 키 해시(Key Hash)를 이용하여 펌웨어 서명(Firmware Signature)을 검증할 수 있다. 이렇게 형성된 신뢰 체인(Chain of Trust)은 하드웨어 신뢰 루트(Hardware Root of Trust)를 부트 펌웨어(Boot Firmware), 운영체제 구성요소, 애플리케이션으로 연결한다. 비인가 또는 변조된 소프트웨어는 보안에 민감한 시스템 자원의 제어권을 획득하기 전에 검증 단계에서 차단되어야 한다.

TPM 기반 측정 부팅(TPM-based Measured Boot)은 부팅 과정에 참여하는 구성요소의 측정값을 기록하여 서명 검증(Signature Verification)을 보완한다. 일반적으로 PCR(Platform Configuration Register)이라고 하는 플랫폼 구성 레지스터는 부팅 단계와 시스템 구성을 나타내는 암호학적 측정값(Cryptographic Measurement)을 누적한다. 측정 부팅은 단순히 소프트웨어의 실행 허용 여부만 결정하는 것이 아니라 실제로 어떤 소프트웨어가 실행되었는지를 나타내는 증거를 제공하여 이후 무결성 평가(Integrity Evaluation)와 원격 증명(Remote Attestation)을 가능하게 한다.

봉인 저장소(Sealed Storage)는 보호된 정보를 특정 플랫폼 상태(Platform Condition)에 결합한다. TPM은 지정된 PCR 값 또는 인증 조건(Authorization Condition)이 만족될 때에만 자격 증명을 사용할 수 있도록 보호할 수 있다. 예를 들어 로봇의 통신 자격 증명(Communication Credential)은 승인된 소프트웨어 구성(Approved Software Configuration)을 통해 장치가 부팅되었을 때만 사용할 수 있도록 설정할 수 있다. 중요한 부팅 구성요소가 변경되면 측정값도 달라지므로 공격자가 저장 매체(Storage Media)를 확보하더라도 보호된 비밀 정보에 접근하지 못하도록 할 수 있다.

원격 증명(Remote Attestation)은 하드웨어 신뢰를 로컬 장치 외부로 확장한다. 임베디드 컴퓨터(Embedded Computer)는 측정된 플랫폼 상태에 관한 암호학적으로 보호된 증거를 인증된 검증자(Authorized Verifier)에게 제공할 수 있다. 플릿 관리 또는 보안 서비스(Security Service)는 이 정보를 이용하여 로봇이 예상된 소프트웨어 구성을 실행하고 있는지 판단한 후 네트워크 접근 허용, 운영 명령 수신, 민감한 데이터 배포 여부를 결정할 수 있다. 따라서 증명(Attestation)은 장치 무결성(Device Integrity)을 시스템 수준의 권한 결정(System-level Authorization Decision)과 연결한다.

HSM과 TPM 기능은 OTA 펌웨어 업데이트(OTA Firmware Update) 시스템에서도 유용하다. 업데이트 패키지(Update Package)는 보호된 조직 서명 키(Organizational Signing Key)를 사용하여 서명하고, 장치는 신뢰할 수 있는 검증 정보(Verification Material)를 이용해 서명을 검증할 수 있다. 장치 측 보안 하드웨어는 업데이트 자격 증명, 롤백 정보(Rollback Information), 인증 키(Authentication Key)를 보호할 수 있다. 운영용 서명 키(Production Signing Key)는 개발용 자격 증명과 분리하고 통제된 암호화 인프라(Cryptographic Infrastructure)를 통해 보호해야 한다. 서명 키 하나가 침해되면 배포된 전체 플릿에 영향을 줄 수 있기 때문이다.

임베디드 통신 보안(Embedded Communication Security)에서도 TLS, DTLS, VPN 또는 애플리케이션 수준 인증(Application-level Authentication)을 위해 하드웨어로 보호된 자격 증명을 사용할 수 있다. 개인 키를 애플리케이션 메모리(Application Memory)에 직접 로드하는 대신 플랫폼과 소프트웨어 스택(Software Stack)이 지원하는 경우 암호화 연산을 보안 구성요소에 위임할 수 있다. 이는 정상 운영 중 키 노출을 줄이지만 보안 하드웨어만으로 프로토콜 구현(Protocol Implementation), 인증서 검증(Certificate Validation), 접근 제어(Access Control), 애플리케이션 로직(Application Logic)의 취약점까지 제거할 수 있는 것은 아니다.

통합 과정에서는 성능(Performance)과 실시간 동작(Real-time Behavior)을 신중하게 고려해야 한다. 특히 보안 장치가 SPI, I2C 또는 다른 주변장치 인터페이스(Peripheral Interface)를 통해 연결된 경우 하드웨어 암호화 연산으로 통신 오버헤드(Communication Overhead)와 실행 지연(Execution Latency)이 발생할 수 있다. 설계자는 어떤 연산에 하드웨어 보호가 반드시 필요한지, 어떤 연산을 소프트웨어에서 안전하게 수행할 수 있는지 구분해야 한다. 고주파 제어 루프(High-frequency Control Loop)가 결정론적 타이밍(Deterministic Timing)을 위반할 수 있는 느린 보안 트랜잭션(Security Transaction)에 예기치 않게 의존해서는 안 된다.

프로비저닝(Provisioning)은 보안 하드웨어 자체만큼 중요하다. 제조 과정에서는 공장 도구(Factory Tool)나 로그를 통해 비밀 정보가 노출되지 않도록 장치 신원, 소유권 정보(Ownership Information), 인증서, 권한 정책(Authorization Policy), 복구 메커니즘(Recovery Mechanism)을 안전하게 설정해야 한다. 프로비저닝 기록(Provisioning Record)은 암호학적 신원과 특정 장치를 연결해야 하며, 자격 증명 교체(Credential Rotation) 및 폐기(Revocation) 메커니즘은 유지보수, 소유권 변경, 침해된 장치, 하드웨어 교체, 최종적인 수명 종료 폐기(End-of-life Decommissioning)를 지원해야 한다.

HSM과 TPM의 적용은 궁극적으로 독립적인 보안 기능이 아니라 보다 광범위한 하드웨어 신뢰 루트 아키텍처(Hardware Root-of-Trust Architecture)의 일부로 다루어야 한다. 보안 부팅(Secure Boot), 측정 부팅(Measured Boot), 보호된 키(Protected Key), 증명(Attestation), 암호화 통신(Encrypted Communication), 서명된 OTA 업데이트(Signed OTA Update), 접근 제어(Access Control), 모니터링(Monitoring), 복구 메커니즘(Recovery Mechanism)은 서로를 보완한다. 이러한 기능을 임베디드 수명주기 전체에 통합하면 하드웨어 기반 보안은 로봇이 진짜 장치인지, 실행 중인 소프트웨어를 신뢰할 수 있는지, 중요한 자격 증명이 안전하게 보호되고 있는지를 확인하기 위한 지속적인 신뢰 기반을 제공한다.

## 11.03 Secure Boot and Root of Trust

![](images/image3.png){width="7.268055555555556in" height="7.268055555555556in"}

보안 부팅(Secure Boot)은 임베디드 시스템(Embedded System)이 진위성(Authenticity)과 무결성(Integrity)을 검증할 수 있는 소프트웨어만 실행하도록 하는 보안 메커니즘(Security Mechanism)이다. 단순히 펌웨어(Firmware)가 플래시 또는 저장장치에 존재한다는 이유로 신뢰하지 않고, 실행 가능한 각 단계를 암호학적으로 검증한 후에만 제어권을 전달한다. 로봇 시스템에서는 이를 통해 제어기, 임베디드 리눅스 컴퓨터(Embedded Linux Computer), 엣지 AI 플랫폼(Edge AI Platform), 안전 관련 프로세서(Safety-related Processor)를 비인가 또는 변조된 소프트웨어로부터 보호할 수 있다.

보안 부팅의 기반은 신뢰 루트(Root of Trust)이며, 이는 시스템이 본질적으로 신뢰해야 하는 가장 작은 구성요소 또는 정보 집합을 의미한다. 일반적으로 변경 불가능한 부트 ROM(Immutable Boot ROM), 일회성 프로그래밍 메모리(One-Time Programmable Memory), eFuse, 보호된 키 저장소(Protected Key Storage), 보안 요소(Secure Element), TPM 또는 하드웨어 보안 서브시스템(Hardware Security Subsystem)을 이용하여 구현한다. 이후의 모든 검증 결정이 궁극적으로 이 기반에 의존하기 때문에 신뢰 루트가 침해되면 전체 부팅 체인(Boot Chain)의 보안이 무효화될 수 있다.

일반적인 보안 부팅 순서(Secure Boot Sequence)는 프로세서가 리셋(Reset) 상태에서 벗어나 내부 부트 ROM에 저장된 변경 불가능한 코드를 실행하면서 시작된다. 이 첫 번째 단계의 코드는 공개 키 해시(Public-key Hash)와 같은 신뢰할 수 있는 암호화 정보(Cryptographic Material)를 포함하거나 참조한다. 이후 실행할 부트 구성요소를 인증한 다음 제어권을 전달하며, 검증된 구성요소가 다시 다음 단계를 인증한다. 이러한 방식으로 부트로더(Bootloader), 펌웨어, 운영체제 커널(Operating-system Kernel), 장치 구성(Device Configuration), 애플리케이션까지 확장되는 신뢰 체인(Chain of Trust)이 형성된다.

디지털 서명(Digital Signature)은 이러한 신뢰 체인의 핵심적인 진위성 검증 메커니즘이다. 소프트웨어 릴리스 과정(Software Release Process)에서 승인된 서명 시스템(Signing System)은 펌웨어의 암호학적 다이제스트(Cryptographic Digest)를 계산하고 보호된 개인 키(Private Key)를 사용하여 서명한다. 대상 장치는 이에 대응하는 신뢰된 공개 키(Public Key)를 이용하여 서명을 검증한다. 서명 이후 펌웨어 내용이 변경되면 계산된 다이제스트가 인증된 서명과 더 이상 일치하지 않으므로 검증에 실패한다.

보안 부팅을 설계할 때 무결성(Integrity)과 진위성(Authenticity)은 구분해야 한다. 체크섬(Checksum)이나 일반적인 해시(Hash)는 우발적인 변경을 탐지할 수 있지만, 펌웨어를 교체할 수 있는 공격자는 보호되지 않은 기준 해시(Reference Hash)까지 함께 변경할 수 있다. 디지털 서명은 소프트웨어의 무결성을 승인된 서명자(Authorized Signer)와 결합한다. 따라서 보안 부팅에는 암호학적 무결성 검사와 함께 어떤 서명 주체가 실행 가능한 소프트웨어를 제공하도록 승인되었는지를 결정하는 신뢰 메커니즘이 필요하다.

검증 키(Verification Key) 자체도 안전하게 고정되어야 한다. 펌웨어와 동일한 쓰기 가능한 플래시 영역(Writable Flash Region)에 제한 없이 공개 키를 저장하면 공격자가 펌웨어와 검증 키를 모두 교체할 수 있다. 따라서 시스템은 일반적으로 승인된 루트 공개 키(Root Public Key)의 해시를 ROM, OTP 메모리, eFuse 또는 기타 하드웨어 보호 영역(Hardware-protected Region)에 저장한다. 이후 부트 ROM은 서명 검증에 사용되는 키가 변경 불가능한 신뢰 앵커(Immutable Trust Anchor)와 일치하는지를 확인한다.

현대의 임베디드 플랫폼(Embedded Platform)은 하나의 펌웨어 이미지만 실행하는 경우가 드물기 때문에 보안 부팅은 일반적으로 여러 단계의 검증으로 구성된다. 프로세서는 1단계 펌웨어(First-stage Firmware), U-Boot와 같은 2차 부트로더(Secondary Bootloader), 신뢰 실행 펌웨어(Trusted Execution Firmware), 운영체제 커널, 장치 트리(Device Tree), initramfs, 루트 파일 시스템(Root Filesystem), 애플리케이션 구성요소를 순차적으로 로드할 수 있다. 보안 설계자는 실행 가능하거나 보안에 민감한 정보를 포함하는 단계를 식별하고, 검증되지 않은 경로를 통해 의도된 신뢰 체인을 우회할 수 없도록 해야 한다.

정상적으로 서명된 소프트웨어라고 해서 영구적으로 안전한 것은 아니므로 롤백 방지(Rollback Protection)가 필요하다. 공격자는 알려진 취약점이 존재하는 과거의 정상 서명 펌웨어를 의도적으로 설치할 수 있다. 안티 롤백(Anti-rollback) 메커니즘은 보호된 보안 버전(Security Version), 단조 증가 카운터(Monotonic Counter) 또는 이에 상응하는 상태를 유지하고 허용된 버전보다 낮은 이미지를 거부한다. 이를 통해 보안 부팅은 진위성뿐만 아니라 허용 가능한 최소 보안 세대(Minimum Security Generation)까지 강제할 수 있다.

키 계층 구조(Key Hierarchy)를 설계하면 개별 자격 증명(Credential)이 침해되었을 때 발생하는 영향을 줄일 수 있다. 릴리스 키(Release Key)를 하드웨어에 직접 영구 저장하는 대신 신뢰 루트가 중간 키(Intermediate Key)를 승인하고, 이 중간 키가 운영 펌웨어(Production Firmware)를 검증하도록 구성할 수 있다. 부트 구성요소, 애플리케이션, 복구 이미지(Recovery Image), 개발 환경(Development Environment), 제품 세대(Product Generation)에 서로 다른 키를 할당할 수도 있다. 신중하게 설계된 키 계층은 장기간의 장치 수명주기 동안 통제된 키 교체(Key Rotation)와 폐기(Revocation)도 가능하게 한다.

복구(Recovery)는 보안 부팅 아키텍처(Secure Boot Architecture)의 예외가 아니라 처음부터 포함되어야 하는 기능이다. 손상이나 중단된 OTA 업데이트로 검증이 실패하면 장치는 검증된 정상 복구 파티션(Known-good Recovery Partition) 또는 대체 A/B 이미지(Alternate A/B Image)를 통해 부팅할 수 있다. 복구 소프트웨어 자체도 동일하거나 동등한 신뢰 체인을 통해 인증되어야 한다. 그렇지 않으면 제한되지 않은 복구 인터페이스(Recovery Interface)가 보안 부팅 보호 기능을 의도적으로 우회하는 경로가 될 수 있다.

보안 부팅과 OTA 보안(OTA Security)은 밀접하게 연결되어 있다. OTA 시스템은 새로운 소프트웨어가 장치에 어떻게 전달되는지를 결정하고, 보안 부팅은 설치된 소프트웨어의 실행 허용 여부를 결정한다. 따라서 업데이트 패키지(Update Package)는 설치 전에 인증되어야 하며 이후 부팅 과정에서 다시 검증되어야 한다. 서명된 OTA 패키지(Signed OTA Package), A/B 파티션(A/B Partition), 롤백 제어(Rollback Control), 보호된 서명 키(Protected Signing Key), 보안 부팅을 결합하면 배포 채널(Distribution Channel)의 침해가 자동으로 지속적인 실행 코드 침해(Persistent Executable Compromise)로 이어지는 것을 방지할 수 있다.

측정 부팅(Measured Boot)은 보안 부팅을 보완하는 메커니즘을 제공한다. 보안 부팅은 비인가 소프트웨어의 실행을 거부하여 보안 정책(Security Policy)을 강제하는 반면, 측정 부팅은 시작 과정에 참여하는 구성요소의 암호학적 측정값(Cryptographic Measurement)을 기록한다. TPM의 플랫폼 구성 레지스터(Platform Configuration Register, PCR)는 이러한 측정값을 누적하여 이후 증명(Attestation)에 활용할 수 있다. 따라서 시스템은 허용할 수 없는 소프트웨어의 실행을 차단하는 동시에 실제 부팅 과정에 참여한 소프트웨어 상태에 대한 증거도 제공할 수 있다.

개발 모드(Development Mode)와 제조 모드(Manufacturing Mode)는 엔지니어링 과정에서 보안 제어가 완화되는 경우가 많기 때문에 특별한 주의가 필요하다. 디버그 인증서(Debug Certificate), 서명되지 않은 개발 이미지(Unsigned Development Image), JTAG 접근, 부팅 구성 핀(Boot Configuration Pin), 공장 프로그래밍 명령(Factory Programming Command), 임시 키(Temporary Key)는 양산 제품에 그대로 남으면 영구적인 취약점이 될 수 있다. 양산 프로비저닝(Production Provisioning)에서는 장치를 최종 보안 상태로 명확하게 전환하고 비인가 개발 부팅 경로와 디버그 메커니즘이 비활성화되거나 엄격하게 통제되는지 검증해야 한다.

실패 처리(Failure Handling)는 통제되지 않은 실행보다 명확하게 정의된 안전하고 복구 가능한 상태(Safe and Recoverable State)를 우선해야 한다. 인증에 실패하면 시스템은 부팅 과정을 중단하거나, 검증된 대체 이미지를 선택하거나, 제한된 복구 환경(Restricted Recovery Environment)에 진입하거나, 진단 상태(Diagnostic Condition)를 보고할 수 있다. 로봇에서는 메인 컴퓨터가 부팅할 수 없는 동안 액추에이터(Actuator)와 안전 기능(Safety Function)이 어떤 물리적 상태를 유지할지도 고려하여 사이버보안 실패(Cybersecurity Failure)가 통제되지 않은 물리적 동작으로 이어지지 않도록 해야 한다.

보안 부팅은 궁극적으로 전원이 인가된 하드웨어(Powered Hardware)에서 실행 가능한 소프트웨어(Executable Software)로 이어지는 최초의 신뢰 가능한 전환을 확립한다. 변경 불가능한 신뢰 앵커(Immutable Trust Anchor), 보호된 검증 키, 디지털 서명, 단계별 검증(Staged Verification), 안티 롤백 제어, 안전한 복구(Secure Recovery), 통제된 프로비저닝(Controlled Provisioning), 서명된 업데이트가 올바르게 통합되면 각각의 소프트웨어 계층은 하위 계층으로부터 신뢰를 상속받는다. 이러한 신뢰 체인은 임베디드 제어기, 로봇 컴퓨터, 엣지 AI 장치, 장기간 운용되는 연결형 기계(Connected Machine)를 보호하는 핵심 보안 경계(Security Boundary)가 된다.

## 11.04 Crypto Libraries: mbedTLS / wolfSSL [w/Code]

![](images/image4.png){width="7.268055555555556in" height="7.268055555555556in"}

암호화 라이브러리(Cryptographic Library)는 개발자가 복잡한 암호 기술을 처음부터 직접 구현하지 않고도 임베디드 시스템(Embedded System)에 필요한 보안 알고리즘(Security Algorithm)과 프로토콜(Protocol)을 사용할 수 있도록 재사용 가능한 구현을 제공한다. mbed TLS와 wolfSSL 같은 라이브러리는 메모리, 저장공간, 처리 성능, 에너지가 제한될 수 있는 임베디드 환경을 위해 설계되었다. 이를 통해 펌웨어(Firmware)와 임베디드 애플리케이션(Embedded Application)에서 보안 통신(Secure Communication), 인증(Authentication), 암호화(Encryption), 인증서 처리(Certificate Handling), 키 관리(Key Management), 암호학적 검증(Cryptographic Verification)을 지원할 수 있다.

mbed TLS는 마이크로컨트롤러(Microcontroller), RTOS 플랫폼, 임베디드 리눅스 장치(Embedded Linux Device), 게이트웨이(Gateway), 연결형 제어기(Connected Controller)에 맞게 구성할 수 있는 모듈식 암호화 및 TLS 소프트웨어 스택(Modular Cryptographic and TLS Software Stack)을 제공한다. 개발자는 제품에서 실제로 필요한 알고리즘과 프로토콜 기능만 포함하도록 구성할 수 있다. 이러한 구성 가능성(Configurable Architecture)은 소형 MCU에서는 최소한의 암호화 풋프린트(Cryptographic Footprint)가 필요하고, 임베디드 리눅스 기반 로봇 컴퓨터에서는 보다 완전한 TLS 및 인증서 인프라(Certificate Infrastructure)를 사용할 수 있다는 점에서 중요하다.

wolfSSL 역시 임베디드 및 자원 제약 시스템(Resource-constrained System)에 최적화되어 있으며 TLS 기능과 함께 암호화 기본 연산(Cryptographic Primitive) 및 플랫폼 통합 기능(Platform Integration Capability)을 제공한다. 베어메탈 펌웨어(Bare-metal Firmware), RTOS 애플리케이션, 임베디드 리눅스 시스템, 네트워크 연결 장치(Network-enabled Device)에 통합할 수 있다. 이러한 임베디드 중심 설계는 보안 기능과 메모리 사용량, 실행 성능, 이식성(Portability), 하드웨어 가속(Hardware Acceleration), 결정론적 시스템 동작(Deterministic System Behavior) 사이의 균형이 필요한 경우 유용하다.

두 라이브러리는 개념적으로 암호화 기본 연산(Cryptographic Primitive)을 상위 수준의 보안 통신 프로토콜(Secure Communication Protocol)과 분리한다. 기본 연산에는 해싱(Hashing), 메시지 인증(Message Authentication), 대칭키 암호화(Symmetric Encryption), 비대칭키 암호화(Asymmetric Cryptography), 디지털 서명(Digital Signature), 키 유도(Key Derivation), 난수 생성(Random-number Generation)이 포함된다. TLS와 관련 프로토콜 계층은 이러한 메커니즘을 결합하여 통신 상대를 인증하고 암호화 매개변수를 협상하며 세션 키(Session Key)를 설정하고 애플리케이션 데이터를 도청이나 비인가 변경으로부터 보호한다.

대칭키 암호화 알고리즘(Symmetric Encryption Algorithm)은 통신 장치 사이에 공유 세션 키가 설정된 이후 데이터를 효율적으로 보호한다. 비대칭키 암호화(Asymmetric Cryptography)는 신원 확인(Identity Verification), 디지털 서명, 안전한 키 설정(Secure Key Establishment) 등에 사용된다. 해시 함수(Hash Function)는 무결성 검증을 위한 고정 크기의 표현을 생성하며, 메시지 인증 코드(Message Authentication Code)는 통신 주체가 비밀 정보를 공유하는 경우 무결성과 진위성을 제공한다. 임베디드 애플리케이션은 암호화 알고리즘을 독자적으로 조합하기보다 검증된 라이브러리 API를 통해 이러한 기능을 사용해야 한다.

난수 생성(Random-number Generation)은 암호화 키, 논스(Nonce), 초기화 값(Initialization Value), 프로토콜 챌린지(Protocol Challenge)에 예측할 수 없는 값이 필요하기 때문에 암호 시스템의 기본적인 의존 요소이다. 따라서 강력한 라이브러리 구성에는 대상 플랫폼(Target Platform)의 신뢰할 수 있는 엔트로피 소스(Entropy Source)가 필요하다. 임베디드 시스템은 하드웨어 난수 생성기(Hardware Random-number Generator), 보안 요소(Secure Element), TPM, MCU 보안 주변장치(Security Peripheral) 또는 신중하게 설계된 플랫폼 소스로부터 엔트로피를 얻을 수 있다. 난수가 약하거나 예측 가능하면 올바르게 구현된 암호화 알고리즘도 무력화될 수 있다.

인증서 처리(Certificate Processing)는 임베디드 장치가 공개 키 기반 구조(Public Key Infrastructure, PKI)에 참여할 수 있도록 한다. 로봇이나 제어기는 자신의 신원을 나타내는 장치 인증서(Device Certificate)를 저장하고 서버 또는 다른 장치가 제공하는 인증서를 검증할 수 있다. 암호화 라이브러리는 인증서 체인(Certificate Chain)을 분석하고 서명을 검증하며 신뢰할 수 있는 인증 기관(Certificate Authority)을 평가하고 TLS 연결 과정의 인증을 지원한다. 공격자가 정상적인 통신 상대를 성공적으로 가장할 수 있다면 암호화된 통신 자체의 보호 효과가 크게 감소하기 때문에 정확한 인증서 검증은 필수적이다.

TLS 통합(TLS Integration)은 일반적으로 제품에 적합한 프로토콜 버전(Protocol Version), 암호 스위트(Cipher Suite), 인증 방식(Authentication Method), 신뢰 앵커(Trust Anchor), 엔트로피 소스, 인증서 정책(Certificate Policy)을 선택하는 것에서 시작한다. 이후 애플리케이션은 TCP와 같은 기본 전송 계층(Transport)을 기반으로 보안 세션(Secure Session)을 생성한다. 핸드셰이크(Handshake) 과정에서 필요한 경우 통신 주체를 인증하고 세션 키를 유도한다. 협상이 완료되면 애플리케이션 개발자가 개별 암호화 연산을 직접 구현하지 않아도 메시지가 암호화되고 무결성이 보호된다.

임베디드 시스템의 통합은 자원 제한(Resource Limitation)을 명시적으로 고려해야 한다는 점에서 데스크톱 환경과 다르다. 필요하지 않은 알고리즘, 레거시 프로토콜 버전(Legacy Protocol Version), 사용하지 않는 인증서 형식, 디버깅 기능(Debugging Function)은 플래시와 RAM 사용량뿐만 아니라 공격 표면(Attack Surface)도 증가시킨다. 따라서 제품에 필요하지 않은 기능은 제거하면서 현대적이고 안전한 알고리즘을 유지하도록 구성해야 한다. 최종 라이브러리 설정은 버전 관리(Version Control) 대상이 되어야 하며 단순한 빌드 옵션(Build Option)이 아니라 제품 보안 아키텍처(Security Architecture)의 일부로 관리해야 한다.

하드웨어 가속(Hardware Acceleration)은 CPU 부하를 줄이는 동시에 민감한 키 연산(Key Operation)을 보호할 수 있다. 많은 MCU와 임베디드 프로세서에는 AES 엔진, 해시 가속기(Hash Accelerator), 공개 키 가속기(Public-key Accelerator), 하드웨어 난수 생성기, 보안 요소 또는 격리된 보안 서브시스템(Isolated Security Subsystem)이 포함되어 있다. 암호화 라이브러리를 플랫폼별 하드웨어 인터페이스와 통합하면 선택된 연산을 하드웨어에서 실행할 수 있다. 이를 통해 성능을 향상시키고 키 노출을 줄일 수 있지만 하드웨어 드라이버와 통합 경계(Integration Boundary) 자체도 신중하게 검증해야 한다.

HSM, TPM 및 보안 요소(Secure Element)와의 통합은 키 보호를 더욱 강화한다. 개인 키(Private Key)를 애플리케이션이 접근할 수 있는 메모리에 직접 저장하는 대신 라이브러리 또는 플랫폼 추상화 계층(Platform Abstraction Layer)이 보호된 하드웨어에 서명이나 인증 연산을 요청할 수 있다. 개인 키는 보안 경계(Secure Boundary) 내부에 유지되고 암호화 연산 결과만 애플리케이션으로 반환된다. 이러한 방식은 소프트웨어 프로토콜의 유연성과 하드웨어 기반 자격 증명 보호(Hardware-backed Credential Protection)를 결합하며, 장치 신원(Device Identity)과 플릿 인증(Fleet Authentication)에 특히 유용하다.

메모리 관리(Memory Management)와 동시성(Concurrency)은 실시간 임베디드 소프트웨어(Real-time Embedded Software)에서 특별히 고려해야 한다. TLS 핸드셰이크, 인증서 분석, 비대칭 암호 연산은 일반적인 애플리케이션 메시지 처리보다 훨씬 많은 메모리와 실행 시간을 요구할 수 있다. 개발자는 실제 운영 조건에서 스택 사용량(Stack Usage), 힙 사용량(Heap Consumption), 최악 조건 지연시간(Worst-case Latency), 스케줄링 영향(Scheduling Effect)을 측정해야 한다. 보안 처리가 모터 제어 루프(Motor-control Loop), 안전 태스크(Safety Task), 워치독 서비스(Watchdog Servicing) 또는 다른 시간 임계 기능(Time-critical Function)을 예기치 않게 방해해서는 안 된다.

라이브러리 보안은 초기 구성뿐만 아니라 지속적인 유지보수(Maintenance)에 의존한다. 암호화 라이브러리에서는 제품 배포 이후에도 구현 취약점(Implementation Vulnerability), 프로토콜 결함(Protocol Defect), 파싱 오류(Parsing Error), 부채널 취약점(Side-channel Weakness)이 발견될 수 있다. 따라서 정확한 라이브러리 버전과 구성을 소프트웨어 자재 명세서(Software Bill of Materials, SBOM)에 기록하고, 보안 권고(Security Advisory)를 지속적으로 확인하며, 업그레이드 시 테스트하고 OTA 유지보수 계획에 포함해야 한다. 장기간 운용되는 로봇은 전체 플랫폼을 교체하지 않고도 암호화 구성요소를 업데이트할 수 있는 실질적인 메커니즘이 필요하다.

안전한 구현(Secure Implementation)을 위해서는 라이브러리를 둘러싼 애플리케이션 경계(Application Boundary)도 보호해야 한다. 올바르게 구현된 TLS라도 노출된 개인 키, 비활성화된 인증서 검증, 하드코딩된 공유 자격 증명(Hard-coded Shared Credential), 안전하지 않은 디버그 인터페이스(Debug Interface), 잘못된 신뢰 저장소(Trust Store), 부적절한 오류 처리(Error Handling)를 보완할 수는 없다. 민감한 버퍼(Sensitive Buffer)의 사용은 최소화하고 자격 증명에는 최소 권한(Least Privilege)을 적용해야 하며 인증 실패 시 안전하게 실패(Fail Securely)하도록 해야 한다. 로깅에서도 키, 비밀번호, 세션 정보(Session Material), 민감한 메타데이터가 포함된 인증서 또는 복호화된 애플리케이션 정보가 노출되지 않도록 해야 한다.

따라서 mbed TLS와 wolfSSL은 그 자체로 완전한 보안 솔루션(Security Solution)이 아니라 보다 큰 임베디드 신뢰 아키텍처(Embedded Trust Architecture)를 구성하는 보안 빌딩 블록(Security Building Block)으로 이해해야 한다. 적절한 구성과 함께 강력한 엔트로피(Entropy), 보호된 키(Protected Key), 보안 부팅(Secure Boot), 하드웨어 신뢰 루트(Hardware Root of Trust), 인증된 OTA 업데이트(Authenticated OTA Update), 인증서 수명주기 관리(Certificate Lifecycle Management), 체계적인 소프트웨어 유지보수를 결합하면 암호화 라이브러리는 안전한 통신과 신뢰할 수 있는 연결형 로봇 시스템(Connected Robotic System)을 구현하기 위한 프로토콜 및 알고리즘 기반을 제공한다.

## 11.05 Secure Communication Channels: TLS 1.3 / DTLS [w/Code]

![](images/image5.png){width="7.268055555555556in" height="7.268055555555556in"}

보안 통신 채널(Secure Communication Channel)은 신뢰할 수 있다고 가정할 수 없는 네트워크를 통해 임베디드 장치(Embedded Device), 로봇, 엣지 컴퓨터(Edge Computer), 플릿 서버(Fleet Server), 클라우드 서비스(Cloud Service) 사이에서 교환되는 데이터를 보호한다. 특히 전송 계층 보안(Transport Layer Security, TLS) 1.3은 신뢰성 있는 전송 연결에서 인증(Authentication), 기밀성(Confidentiality), 무결성(Integrity)을 제공하며, 데이터그램 전송 계층 보안(Datagram Transport Layer Security, DTLS)은 UDP 방식의 전송 동작이 필요한 데이터그램 기반 통신에 유사한 보안 원칙을 적용한다.

TLS는 전송 계층(Transport Layer)의 상위이자 애플리케이션 프로토콜(Application Protocol)의 하위에서 동작하므로 애플리케이션이 모든 메시지마다 암호화를 개별적으로 구현하지 않고도 보호된 정보를 교환할 수 있다. 임베디드 애플리케이션은 TCP 위에서 TLS 세션(Session)을 설정한 후 보호된 채널을 통해 일반적인 애플리케이션 데이터를 전송한다. TLS 계층은 송신 정보를 암호화하고 수신 레코드(Record)를 인증하며 암호화 상태(Cryptographic State)를 관리하고 무결성 검증에 실패한 메시지를 거부한다.

TLS 1.3은 오래된 암호화 옵션(Cryptographic Option)을 제거하고 불필요한 핸드셰이크(Handshake) 복잡성을 줄임으로써 이전 TLS 설계를 단순화하고 강화한다. 현대적인 암호 스위트(Cipher Suite)는 연관 데이터가 포함된 인증 암호화(Authenticated Encryption with Associated Data)를 사용하여 기밀성과 무결성 보호를 하나의 조정된 구조로 결합한다. 키 설정(Key Establishment)은 안전한 임시 메커니즘(Ephemeral Mechanism)을 사용하고 인증에는 일반적으로 인증서(Certificate)와 디지털 서명(Digital Signature)을 사용한다. 이러한 구조는 안전하지 않은 협상이나 레거시 구성 오류(Legacy Configuration Error)의 가능성을 줄인다.

TLS 핸드셰이크는 보호된 통신에 필요한 암호학적 컨텍스트(Cryptographic Context)를 설정한다. 클라이언트(Client)는 지원하는 프로토콜 매개변수(Protocol Parameter)와 키 설정 정보를 제공하면서 통신을 시작한다. 서버(Server)는 호환되는 매개변수를 선택하고 인증 정보를 제공하며 양측은 공유 세션 비밀 정보(Shared Session Secret)를 유도한다. 검증이 성공하면 이후의 애플리케이션 트래픽(Application Traffic)은 대칭키 암호화(Symmetric Cryptography)가 지속적인 데이터 교환에 훨씬 효율적이므로 대칭 세션 키(Symmetric Session Key)를 사용하여 보호된다.

서버 인증(Server Authentication)은 일반적으로 X.509 인증서와 신뢰할 수 있는 공개 키 기반 구조(Public Key Infrastructure, PKI)를 기반으로 한다. 임베디드 클라이언트는 인증서가 의도된 서버에 속하는지, 허용된 신뢰 체인(Trust Chain)을 통해 서명되었는지, 유효성 요구사항(Validity Requirement)을 만족하는지, 예상된 신원(Identity)과 일치하는지를 검증한다. 인증서 검증이 비활성화되거나 잘못 구현되어 있다면 TLS를 활성화하는 것만으로는 충분하지 않다. 공격자가 암호화된 연결을 유지하면서도 의도된 원격 엔드포인트(Remote Endpoint)를 가장할 수 있기 때문이다.

상호 TLS(Mutual TLS, mTLS)는 인증을 통신 양측으로 확장한다. 서버를 검증하는 것에 더하여 서버도 임베디드 장치 또는 로봇을 나타내는 클라이언트 인증서(Client Certificate)를 요청하고 검증한다. 따라서 각 로봇은 플릿 전체가 공유하는 비밀번호 대신 고유한 암호학적 신원(Cryptographic Identity)을 가질 수 있다. 개인 키(Private Key)를 TPM, HSM, 보안 요소(Secure Element), MCU 보안 서브시스템(MCU Security Subsystem)으로 보호하면 상호 TLS를 통해 강력한 하드웨어 기반 장치 인증(Hardware-backed Device Authentication)을 구현할 수 있다.

순방향 비밀성(Forward Secrecy)은 현대적인 TLS 키 설정의 중요한 특성이다. 임시 세션 비밀 정보(Ephemeral Session Secret)를 생성함으로써 장기간 사용하는 개인 인증 키(Long-term Private Authentication Key)가 나중에 침해되더라도 이전에 수집된 세션 트래픽(Session Traffic)이 자동으로 노출되지 않도록 한다. 이는 로봇과 산업 시스템이 수년 동안 운영될 수 있다는 점에서 특히 중요하다. 장치 자격 증명(Device Credential)이 나중에 침해되었다는 이유만으로 과거에 기록된 네트워크 통신까지 복호화될 수 있어서는 안 된다.

DTLS는 애플리케이션이 TCP가 제공하는 순서가 보장되고 신뢰성 있는 바이트 스트림(Ordered and Reliable Byte Stream)에 의존할 수 없는 데이터그램 통신(Datagram Communication)을 보호한다. UDP 기반 시스템에서는 패킷(Packet)이 손실되거나 중복되거나 순서가 변경될 수 있으므로 DTLS는 이러한 특성을 수용하면서도 인증, 암호화, 무결성 보호, 재전송 공격 방지(Replay Resistance)를 제공해야 한다. 따라서 낮은 통신 오버헤드(Communication Overhead), 데이터그램 의미 구조(Datagram Semantics), 지연시간 특성(Latency Characteristics) 때문에 UDP가 적합한 임베디드 애플리케이션에 DTLS를 사용할 수 있다.

따라서 TLS와 DTLS의 차이는 근본적으로 서로 다른 보안 목표가 아니라 기반 전송 모델(Underlying Transport Model)의 차이에서 발생한다. TLS는 일반적으로 신뢰성 있는 스트림 기반 통신(Stream-oriented Communication)을 보호하고, DTLS는 신뢰성이 보장되지 않는 데이터그램 기반 통신을 보호한다. 설계자는 UDP와 DTLS가 항상 더 빠르거나 TCP와 TLS가 항상 더 안전하다고 가정하지 말고 애플리케이션의 동작 특성에 따라 프로토콜을 선택해야 한다. 전체 시스템의 타이밍(Timing)과 신뢰성(Reliability)은 실제 환경에서 측정해야 한다.

재전송 공격 방지(Replay Protection)는 명령 중심의 로봇 통신(Command-oriented Robotic Communication)에서 특히 중요하다. 공격자는 정상적인 패킷을 복호화하거나 수정하지 않고 캡처한 후 나중에 다시 전송할 수 있다. 보안 채널 프로토콜은 순서 정보(Sequence Information), 인증된 레코드(Authenticated Record), 프로토콜 상태(Protocol State)를 이용하여 허용할 수 없는 반복을 식별한다. 안전 필수 명령(Safety-critical Command)의 의미적 유효성(Semantic Validity)이 타임스탬프(Timestamp), 운영 모드(Operational Mode), 임무 상태(Mission State), 빠르게 변화하는 로봇 상태에 의존한다면 애플리케이션 수준에서 추가적인 최신성 검증(Freshness Mechanism)이 필요할 수 있다.

세션 재개(Session Resumption)는 보안 연결을 반복적으로 설정하는 비용을 줄일 수 있다. 완전한 공개 키 핸드셰이크(Public-key Handshake)는 CPU 시간, 메모리, 대역폭(Bandwidth), 에너지를 소비하므로 자원이 제한된 임베디드 장치가 짧은 통신마다 이를 수행하기에는 부담이 될 수 있다. TLS 메커니즘은 적절한 경우 이전에 설정된 상태를 기반으로 새로운 세션을 안전하게 유도할 수 있다. 그러나 세션 재개 자격 증명(Resumption Credential)을 안전하게 보호해야 하며 오래되거나 침해된 세션 정보가 무기한 사용되지 않도록 수명주기 정책(Lifecycle Policy)을 적용해야 한다.

mbed TLS와 wolfSSL 같은 암호화 라이브러리(Cryptographic Library)는 임베디드 시스템을 위한 실용적인 TLS 및 DTLS 구현을 제공한다. 라이브러리 구성(Configuration)에 따라 지원되는 프로토콜 버전, 암호 스위트, 인증서 처리, 엔트로피 소스(Entropy Source), 메모리 요구량, 하드웨어 가속 인터페이스(Hardware Acceleration Interface)가 결정된다. 제품에서는 오래된 프로토콜과 불필요한 알고리즘을 비활성화하고 명시적인 신뢰 저장소(Trust Store)를 유지하며, 라이브러리의 기본 설정이 자동으로 시스템 보안 요구사항을 만족한다고 가정하지 말고 실제 양산 구성(Production Configuration)을 시험해야 한다.

특히 실시간 로봇 플랫폼(Real-time Robotic Platform)에서는 보안과 성능(Performance)을 함께 평가해야 한다. 핸드셰이크와 인증서 검증은 상당한 순간적 계산 부하(Computational Burst)를 발생시킬 수 있으며 암호화된 레코드는 운영 과정에서도 처리 및 통신 오버헤드를 추가한다. 보안 통신 처리는 하드 실시간 제어 루프(Hard Real-time Control Loop)와 적절하게 분리해야 한다. 하드웨어 암호화 가속기(Hardware Cryptographic Accelerator)를 사용하면 CPU 사용량을 줄일 수 있지만 최악 조건 지연시간(Worst-case Latency), 메모리 사용량, 연결 복구(Connection Recovery), 네트워크 혼잡(Network Congestion)은 여전히 시스템 수준에서 시험해야 한다.

보안 채널은 올바른 키 및 인증서 수명주기 관리(Key and Certificate Lifecycle Management)에도 의존한다. 장치 신원(Device Identity)은 제조 또는 배포 과정에서 안전하게 프로비저닝(Provisioning)되어야 하며 인증서에는 갱신(Renewal), 교체(Rotation), 폐기(Revocation), 대체(Replacement) 절차가 필요하다. 침해된 로봇은 전체 플릿의 자격 증명을 교체하지 않고도 신뢰된 통신 환경에서 제거할 수 있어야 한다. 배포 규모가 개별 장치에서 수백 또는 수천 대의 연결형 기계로 확대될수록 자동화된 인증서 관리(Automated Certificate Management)가 더욱 중요해진다.

네트워크 보안(Network Security)은 TLS 또는 DTLS 연결이 설정되었다고 해서 끝나는 것이 아니다. 애플리케이션은 여전히 명령을 인증하고 권한 부여 정책(Authorization Policy)을 적용하며 메시지 내용을 검증하고 자원 사용량을 제한하며 연결 실패에 안전하게 대응해야 한다. 네트워크 분할(Network Segmentation), 속도 제한(Rate Limiting), 모니터링(Monitoring), 보안 로깅(Secure Logging), 자격 증명 보호(Credential Protection), 침입 탐지(Intrusion Detection)는 암호화된 채널을 보완한다. TLS는 전송 중 통신을 보호하지만 침해된 엔드포인트(Compromised Endpoint)나 안전하지 않은 애플리케이션 동작까지 수정할 수는 없다.

따라서 TLS 1.3과 DTLS는 계층화된 임베디드 통신 아키텍처(Layered Embedded Communication Architecture)의 핵심 기반 구성요소로 이해해야 한다. 하드웨어로 보호된 장치 신원, 신뢰할 수 있는 인증서, 강력한 엔트로피, 보안 부팅(Secure Boot), 암호화 라이브러리, 인증된 OTA 업데이트(Authenticated OTA Update), 네트워크 분할, 지속적인 자격 증명 관리를 결합하면 로봇과 주변 인프라 사이에 신뢰할 수 있는 통신 채널을 구축할 수 있다. 그 결과 신뢰할 수 없는 네트워크 환경에서도 기밀성이 유지되고 통신 주체가 인증되며 데이터 무결성이 보호되고 다양한 공격과 장애에 대응할 수 있는 견고한 통신 체계를 형성할 수 있다.

## 11.06 Code Obfuscation and Tamper Resistance

![](images/image6.png){width="7.268055555555556in" height="7.268055555555556in"}

코드 난독화(Code Obfuscation)와 변조 저항성(Tamper Resistance)은 배포 이후 임베디드 소프트웨어(Embedded Software)가 비인가 분석, 변경, 추출, 조작을 당하지 않도록 보호한다. 특히 로봇과 연결형 제어기(Connected Controller)는 공격자가 장치에 물리적으로 접근하여 펌웨어 이미지(Firmware Image)를 추출하거나 바이너리(Binary)를 분석하고 저장된 코드를 변경할 가능성이 있기 때문에 이러한 보호가 중요하다. 이러한 기술은 배포된 펌웨어가 접근 불가능하다고 가정하는 대신 보호된 소프트웨어를 이해하거나 변경하는 데 필요한 공격 비용과 난이도를 증가시킨다.

난독화(Obfuscation)는 필요한 실행 동작은 유지하면서 소프트웨어 내부 논리를 이해하기 어렵게 만드는 형태로 프로그램을 변환한다. 심볼 이름(Symbol Name), 제어 구조(Control Structure), 데이터 표현(Data Representation), 프로그램 구성(Program Organization) 등을 변환하여 역공학(Reverse Engineering)에 더 많은 노력이 필요하도록 만들 수 있다. 난독화 자체가 암호학적 기밀성(Cryptographic Confidentiality)을 제공하는 것은 아니지만 독점 알고리즘, 보안 검사 로직(Security Check Logic), 프로토콜 로직(Protocol Logic), 중요한 지식재산(Intellectual Property)을 분석하는 비용을 증가시킬 수 있다.

심볼 및 메타데이터 축소(Symbol and Metadata Reduction)는 가장 단순한 보호 기법 중 하나이다. 양산 바이너리(Production Binary)에는 내부 아키텍처를 노출할 수 있는 불필요한 디버그 심볼(Debug Symbol), 설명적인 함수 이름, 소스 경로(Source Path), 개발 문자열(Development String), 진단 정보(Diagnostic Information)를 포함하지 않는 것이 좋다. 이러한 정보를 제거하면 역공학자가 이용할 수 있는 정보를 줄일 수 있다. 그러나 장애 분석, 유지보수, 사고 조사에 필요하므로 해당 심볼과 디버깅 정보는 조직 내부에서 안전하게 별도로 보관해야 한다.

제어 흐름 난독화(Control-flow Obfuscation)는 프로그램의 논리적인 실행 구조를 복원하기 어렵게 만든다. 변환 과정에서 분기(Branch)를 재구성하거나 간접 제어 전환(Indirect Control Transfer)을 도입하고 연산을 여러 부분으로 분할하거나 기본 블록(Basic Block) 사이의 관계를 불명확하게 만들면서 원래의 기능적 동작은 유지할 수 있다. 이러한 방법은 정적 분석(Static Analysis)을 어렵게 하지만 과도한 변환은 코드 크기, 실행 시간, 검증 복잡도(Verification Complexity)를 증가시킬 수 있으므로 메모리 제약 및 실시간 임베디드 시스템에서 특히 주의해야 한다.

데이터 난독화(Data Obfuscation)는 바이너리 이미지 내부의 민감한 상수(Constant), 테이블(Table), 구성 구조(Configuration Structure), 프로토콜 식별자(Protocol Identifier), 독점 매개변수(Proprietary Parameter)를 쉽게 식별하지 못하도록 하는 방법이다. 정적 데이터를 인코딩(Encoding)하거나 변환하면 단순한 문자열 검색만으로 구현 세부사항이 노출되는 것을 방지할 수 있다. 그러나 이를 안전한 키 저장(Secure Key Storage)과 혼동해서는 안 된다. 암호화 개인 키, 비밀번호, 장기 비밀 정보(Long-term Secret)는 가역적인 소프트웨어 난독화가 아니라 하드웨어 기반 또는 암호학적으로 보호된 저장소를 사용해야 한다.

안티 디버깅(Anti-debugging) 및 안티 분석(Anti-analysis) 메커니즘은 소프트웨어 실행에 대한 비인가 관찰을 탐지하거나 제한할 수 있다. 양산 시스템에서는 개발 과정에서 필요했던 JTAG, SWD, 트레이스 포트(Trace Port), 부트로더 콘솔(Bootloader Console), 진단 인터페이스(Diagnostic Interface)를 비활성화하거나 인증된 접근만 허용할 수 있다. 하드웨어 보안 메커니즘은 디버그 접근을 영구적으로 제한하거나 인증된 유지보수 세션(Authenticated Maintenance Session)을 요구할 수 있다. 배포 이후에도 제한되지 않은 엔지니어링 인터페이스를 남겨두면 공격자가 메모리와 프로세서 상태를 직접 관찰하여 정교한 소프트웨어 보호 기능을 우회할 수 있다.

변조 저항성(Tamper Resistance)은 소프트웨어 변환을 넘어 비인가 물리적 또는 논리적 변경을 어렵게 만들거나 이를 탐지하고 복구할 수 있도록 보호 범위를 확장한다. 하드웨어 메커니즘에는 보호된 플래시 영역(Protected Flash Region), 메모리 접근 제어(Memory Access Control), 보안 요소(Secure Element), 디버그 잠금(Debug Lockdown), 전압 및 클록 모니터링(Voltage and Clock Monitoring), 변조 센서(Tamper Sensor), 글리치 보호(Glitch Protection), 오류 탐지(Fault Detection), 보안 삭제(Secure Erase) 기능 등이 포함될 수 있다. 이러한 제어는 임베디드 장비가 물리적으로 접근 가능한 환경이나 잠재적으로 적대적인 환경에서 운영될 때 특히 중요하다.

펌웨어 무결성 검증(Firmware Integrity Verification)은 난독화만 사용하는 것보다 변경에 대한 더욱 강력한 방어를 제공한다. 코드 서명(Code Signing)과 보안 부팅(Secure Boot)을 사용하면 장치는 실행 가능한 소프트웨어가 승인된 출처에서 제공되었으며 변경되지 않았는지를 검증할 수 있다. 하드웨어 신뢰 루트(Hardware Root of Trust)는 부트로더(Bootloader)를 인증하고, 부트로더는 이후의 펌웨어와 운영체제 구성요소를 검증할 수 있다. 난독화가 역공학을 지연시키는 역할을 한다면 암호학적 검증(Cryptographic Verification)은 변경된 소프트웨어의 실행 허용 여부를 직접 결정한다.

런타임 무결성(Runtime Integrity) 메커니즘은 부팅 과정 이후 발생하는 조작을 탐지할 수 있다. 메모리 보호(Memory Protection), 권한 분리(Privilege Separation), 체크섬(Checksum), 보호된 제어 데이터(Protected Control Data), 워치독(Watchdog), 런타임 모니터링(Runtime Monitoring)을 통해 예상하지 못한 변경이나 비정상적인 실행 상태를 식별할 수 있다. 보안 모니터링은 메모리 위반, 인증 실패, 부팅 실패, 통신 이상(Communication Anomaly), 물리적 변조 이벤트를 감시하고 필요에 따라 로깅, 격리(Isolation), 리셋, 잠금(Lockdown), 보안 삭제 또는 안전 상태 전환(Safe-state Transition)을 수행할 수 있다.

결함 주입 저항성(Fault-injection Resistance)은 펌웨어를 직접 변경하는 대신 하드웨어 동작을 의도적으로 교란하는 공격에 대응한다. 전압 글리치(Voltage Glitch), 클록 조작(Clock Manipulation), 전자기적 교란(Electromagnetic Disturbance) 등의 결함 공격은 명령 실행을 건너뛰거나 비교 연산을 손상시키고 인증 절차를 우회하거나 보안 상태 전환을 변경하려 할 수 있다. 방어 하드웨어는 비정상적인 전압과 클록 상태를 감시하고 중복 검사(Redundant Check)를 수행하며 결함을 탐지하여 예기치 않은 실행이 발생하더라도 보안에 중요한 결정이 접근 권한을 허용하는 대신 안전하게 실패(Fail Safely)하도록 구성할 수 있다.

비밀 정보(Secret)는 일반적인 프로그램 로직보다 강력하게 보호해야 한다. 암호화 키(Encryption Key), 장치 신원(Device Identity), 서명 자격 증명(Signing Credential), 인증 비밀 정보(Authentication Secret)는 가능한 경우 보안 저장소(Secure Storage), 보안 요소, TPM, HSM 기반 인프라(HSM-backed Infrastructure), MCU 보안 하드웨어에 격리해야 한다. 애플리케이션은 일반 메모리에 개인 키를 노출하지 않고 암호화 연산만 요청하는 것이 이상적이다. 실행 코드 내부에 직접 포함된 키를 난독화하는 것은 단순히 표현을 숨기는 것이므로 안전한 키 보관(Secure Key Custody)과 동일하게 취급해서는 안 된다.

변조 대응(Tamper Response)은 임베디드 시스템의 운영 역할에 따라 설계되어야 한다. 의심스러운 물리적 접근이나 무결성 실패가 탐지되면 제한된 운영(Restricted Operation), 자격 증명 무효화(Credential Invalidation), 보안 로깅(Secure Logging), 네트워크 격리(Network Isolation), 리셋 또는 선택된 비밀 정보의 보안 삭제를 수행할 수 있다. 로봇 시스템에서는 이러한 대응이 기능 안전(Functional Safety) 요구사항도 만족해야 한다. 변조가 탐지되었다는 이유만으로 보안 메커니즘이 액추에이터(Actuator)의 위험한 동작을 갑작스럽게 발생시켜서는 안 된다.

보호 기능은 개발 단계에서 양산 단계로의 전환(Development-to-production Transition)까지 포함해야 한다. 엔지니어링 장치는 제한되지 않은 디버깅, 유연한 부팅 모드(Boot Mode), 임시 자격 증명(Temporary Credential), 서명되지 않은 소프트웨어를 허용하는 경우가 많지만 실제 배포 제품에는 강화된 구성(Hardened Configuration)이 필요하다. 보안 프로비저닝(Secure Provisioning)을 통해 장치를 양산 보안 상태로 전환하고 운영용 비밀 정보를 보호하며 필요한 경우 보안 부팅과 읽기 방지(Readout Protection)를 활성화하고 보안 수준이 낮은 수명주기 상태로 롤백되는 것을 방지해야 한다.

난독화는 무조건 적용하기보다 엔지니어링 트레이드오프(Engineering Tradeoff)를 평가해야 한다. 변환 과정에서 바이너리 크기, CPU 사용량, 시작 시간(Startup Time), 디버깅 난이도, 시험 복잡도가 증가할 수 있다. 또한 정적 분석, 코드 커버리지(Code Coverage), 안전 인증(Safety Certification), 충돌 진단(Crash Diagnostics), 결정론적 타이밍(Deterministic Timing) 가정에 영향을 줄 수 있다. 따라서 안전 필수 및 실시간 구성요소는 보안 강화 이후에도 요구되는 기능적 동작과 시간적 동작이 변경되지 않았음을 신중하게 검증해야 한다.

어떠한 난독화 기술도 대상 장치를 완전히 통제할 수 있는 충분한 능력의 공격자에게 영구적인 방어 수단으로 간주해서는 안 된다. 난독화의 목적은 역공학 비용을 높이고 선택된 구현 지식을 보호하는 것이며, 보다 강력한 보안 제어는 진위성(Authenticity), 기밀성(Confidentiality), 접근 제한(Access Restriction), 무결성을 직접적으로 보장해야 한다. 따라서 효과적인 임베디드 보호는 난독화를 보안 부팅, 코드 서명, 보호된 키, 디버그 잠금, 메모리 보호, 변조 탐지, 인증된 업데이트(Authenticated Update), 런타임 모니터링, 체계적인 수명주기 관리(Lifecycle Management)와 결합해야 한다.

결국 코드 난독화와 변조 저항성은 심층 방어(Defense in Depth)를 구성하는 상호 보완적인 계층이다. 난독화는 소프트웨어의 해석과 악용을 어렵게 하고, 변조 저항 하드웨어(Tamper-resistant Hardware)는 물리적 조작을 제한하며, 암호학적 검증은 비인가 펌웨어의 실행을 차단하고, 모니터링은 운영 중 발생하는 의심스러운 동작을 탐지한다. 이러한 메커니즘을 함께 적용하면 임베디드 제품의 전체 수명주기 동안 신뢰 가능하고 복구 가능한 동작을 유지하면서 펌웨어, 지식재산, 자격 증명, 로봇의 동작을 보호할 수 있다.

## 11.07 Vulnerability Scanning: Embedded SW SBOM Management

![](images/image7.png){width="7.268055555555556in" height="7.268055555555556in"}

취약점 스캐닝(Vulnerability Scanning)과 소프트웨어 자재 명세서(Software Bill of Materials, SBOM) 관리는 임베디드 제품(Embedded Product)에 포함된 소프트웨어 구성요소와 알려진 보안 취약점(Security Vulnerability)에 대한 지속적인 가시성(Visibility)을 제공한다. 현대의 로봇 펌웨어(Robot Firmware)는 내부에서 개발한 코드만으로 구성되는 경우가 드물며 운영체제 패키지, 미들웨어(Middleware), 통신 라이브러리, 드라이버(Driver), 부트 구성요소(Boot Component), 오픈소스 라이브러리(Open-source Library), 공급업체 바이너리(Vendor Binary), 개발 의존성(Development Dependency) 등을 포함할 수 있다. SBOM은 이러한 소프트웨어 공급망(Software Supply Chain)을 파악하기 위한 목록을 제공한다.

SBOM은 제품이나 펌웨어 릴리스(Firmware Release)에 포함된 소프트웨어 구성요소를 설명하는 구조화된 기록(Structured Record)이다. 유용한 항목에는 구성요소 이름, 버전, 공급업체(Supplier), 패키지 식별자(Package Identifier), 라이선스(License), 의존성 관계(Dependency Relationship), 출처 정보(Provenance Information) 등이 포함된다. 취약점이 발표된 이후 소프트웨어 구성을 다시 조사하는 대신 엔지니어링 팀은 SBOM을 조회하여 어떤 제품, 펌웨어 버전, 로봇 모델 또는 배포된 장치에 영향을 받는 구성요소가 포함되어 있는지 확인할 수 있다.

SBOM 생성(SBOM Generation)은 독립적인 스프레드시트로 수동 관리하기보다 소프트웨어 빌드 및 릴리스 프로세스(Software Build and Release Process)에 통합되어야 한다. 빌드 시스템(Build System), 패키지 관리자(Package Manager), 소스 저장소(Source Repository), 컨테이너 이미지(Container Image), 바이너리 분석 도구(Binary Analysis Tool)로부터 구성요소 정보를 수집할 수 있다. 생성된 SBOM은 실제로 릴리스된 산출물(Released Artifact)과 정확하게 대응해야 하며, 이를 통해 배포된 펌웨어 이미지를 해당 이미지가 생성된 정확한 의존성 집합(Dependency Set)까지 추적할 수 있어야 한다.

SPDX와 CycloneDX 같은 일반적인 SBOM 표현 형식은 엔지니어링 조직, 공급업체, 고객, 보안 도구(Security Tool) 사이에서 구성요소 정보를 교환하기 위한 기계 판독 가능 형식(Machine-readable Format)을 제공한다. 표준화된 식별자를 사용하면 취약점 스캐너(Vulnerability Scanner)가 소프트웨어 패키지와 취약점 데이터베이스(Vulnerability Database)를 연계할 수 있어 자동화된 분석이 쉬워진다. 어떤 형식을 선택하는가보다 구성요소 신원(Component Identity), 버전 정보, 의존성 관계, 릴리스 추적성(Release Traceability)을 전체 수명주기 동안 정확하게 유지하는 것이 더욱 중요하다.

취약점 스캐닝(Vulnerability Scanning)은 알려진 구성요소와 소프트웨어 버전을 공개된 취약점 정보와 비교한다. 스캐너는 공통 취약점 및 노출(Common Vulnerabilities and Exposures, CVE) 기록, 보안 권고(Security Advisory), 공급업체 공지(Vendor Notice), 기타 취약점 인텔리전스(Vulnerability Intelligence)와의 연관성을 식별할 수 있다. 소스 코드 및 바이너리 분석(Source-code and Binary Analysis)은 단순한 패키지 버전 정보만으로 표현되지 않는 구현 취약점(Implementation Weakness)을 식별하여 의존성 스캐닝(Dependency Scanning)을 보완할 수 있다. 이러한 방법은 서로 다른 형태의 증거를 제공하므로 동일한 방법으로 간주해서는 안 된다.

취약점이 일치한다고 해서 해당 임베디드 제품에서 반드시 악용 가능(Exploitable)하다는 의미는 아니다. 취약한 라이브러리에 포함된 기능이 실제 빌드에 포함되지 않았거나 접근할 수 없거나 활성화되지 않았거나 배포된 구성에서 외부에 노출되지 않았을 수 있다. 반대로 일반적인 심각도(Severity)가 중간 수준인 취약점이라도 로봇 통신, 펌웨어 업데이트, 인증, 안전 관련 동작(Safety-relevant Behavior)을 제어한다면 치명적인 위험이 될 수 있다. 따라서 분류 및 판단(Triage)에는 취약점 데이터베이스 정보뿐만 아니라 실제 제품 컨텍스트(Product Context)가 필요하다.

위험 우선순위 결정(Risk Prioritization)은 기술적인 심각도와 악용 가능성(Exploitability), 노출도(Exposure), 운영 역할(Operational Role), 물리적 결과(Physical Consequence)를 함께 고려해야 한다. 공격자가 취약한 인터페이스에 접근할 수 있는지, 인증이 필요한지, 취약점 악용을 통해 코드 실행(Code Execution)이나 권한 상승(Privilege Escalation)이 가능한지, 침해가 액추에이터(Actuator)나 안전 기능(Safety Function)에 영향을 미칠 수 있는지를 평가해야 한다. 임베디드 보안 팀은 단순히 수치화된 심각도 순서로 취약점을 처리하기보다 현실적인 시스템 영향(System Impact)을 기준으로 수정 우선순위를 결정해야 한다.

오탐(False Positive)과 불완전한 구성요소 식별(Incomplete Component Identification)은 취약점 관리(Vulnerability Management)의 실질적인 과제이다. 패키지 이름이 배포판이나 공급업체에 따라 다를 수 있으며, 공급업체가 업스트림 버전(Upstream Version)의 일반적인 패턴을 변경하지 않고 보안 수정 사항을 백포트(Backport)할 수도 있고, 독점 바이너리(Proprietary Binary)는 제한된 메타데이터만 제공할 수 있다. 따라서 모든 자동화된 일치 결과에는 엔지니어링 검토를 지원할 수 있는 충분한 출처 및 구성 정보가 필요하다. 탐지 결과를 예외 처리하거나 억제(Suppression)하는 경우에도 그 근거를 기록하고 소프트웨어 또는 배포 조건이 변경될 때 다시 검토해야 한다.

의존성 그래프(Dependency Graph)는 취약한 소프트웨어가 제품에 간접적으로 포함될 수 있기 때문에 특히 중요하다. 애플리케이션이 미들웨어에 의존하고 해당 미들웨어가 다시 다른 라이브러리에 의존하면 개발자가 직접 선택하지 않은 전이적 의존성(Transitive Dependency)이 발생한다. 효과적인 SBOM은 가능한 경우 이러한 관계를 나타내어 특정 구성요소가 왜 존재하는지, 어떤 상위 기능이 해당 구성요소를 필요로 하는지, 그리고 구성요소를 업그레이드하거나 교체 또는 제거했을 때 어떤 기능이 영향을 받을 수 있는지를 파악할 수 있도록 해야 한다.

임베디드 리눅스 플랫폼(Embedded Linux Platform)은 일반적으로 MCU 펌웨어보다 훨씬 많은 소프트웨어 구성요소를 포함한다. 커널 구성요소(Kernel Component), 시스템 라이브러리(System Library), 네트워크 서비스(Network Service), 패키지 관리자 콘텐츠, 컨테이너(Container), 드라이버, 암호화 라이브러리(Cryptographic Library), 로보틱스 미들웨어(Robotics Middleware), 애플리케이션 패키지를 모두 추적해야 할 수 있다. MCU 시스템은 구성요소 수가 상대적으로 적지만 RTOS 커널, 공급업체 SDK, 프로토콜 스택(Protocol Stack), 부트로더(Bootloader), 암호화 라이브러리, 사전 컴파일된 하드웨어 추상화 계층(Hardware Abstraction Layer)에 의존할 수 있으며 이러한 구성요소의 취약점도 배포된 제품에 영향을 줄 수 있다.

취약점 관리는 제품 출시 이전부터 시작해야 한다. 지속적 통합(Continuous Integration, CI) 파이프라인은 소프트웨어 개발 과정에서 의존성, 소스 코드, 컨테이너 이미지, 생성된 펌웨어 산출물을 스캔할 수 있다. 정의된 위험 임계값(Risk Threshold)을 초과하는 취약점이 포함된 릴리스는 승인된 예외가 존재하지 않는 한 정책을 통해 차단할 수 있다. 이러한 시프트 레프트(Shift-left) 접근법은 취약한 의존성이 서명된 펌웨어에 포함되어 로봇 플릿(Robot Fleet) 전체에 배포되기 전에 교체할 수 있으므로 수정 비용을 줄인다.

새로운 취약점은 소프트웨어가 출시된 이후 오랜 시간이 지나서도 발견될 수 있으므로 이 과정은 배포 이후에도 계속되어야 한다. 이전에는 허용 가능했던 펌웨어 이미지도 포함된 구성요소에서 새로운 취약점이 공개되면 보안 문제가 될 수 있다. 따라서 조직은 새로운 취약점 인텔리전스를 저장된 SBOM과 연계하고 영향을 받는 제품, 소프트웨어 릴리스, 배포 장치 집단(Deployment Population), 운영 환경(Operational Environment)을 식별하는 지속적인 취약점 모니터링(Continuous Vulnerability Monitoring) 체계를 구축해야 한다.

취약점 수정(Remediation)이 항상 최신 패키지를 즉시 설치하는 것을 의미하지는 않는다. 임베디드 제품에는 엄격한 호환성(Compatibility), 메모리, 타이밍(Timing), 인증(Certification), 하드웨어 제약이 존재할 수 있다. 보안 패치는 통신 타이밍, 드라이버 동작, CPU 부하, 실시간 스케줄링(Real-time Scheduling)에 영향을 줄 수 있다. 따라서 업데이트는 플릿에 배포하기 전에 회귀 시험(Regression Testing), 보안 시험(Security Testing), 기능 시험(Functional Testing), 안전 시험(Safety Testing)을 통과해야 한다. 검증된 패치를 준비하는 동안에는 서비스 비활성화, 네트워크 접근 제한, 인터페이스 격리와 같은 임시 완화 조치(Temporary Mitigation)가 필요할 수 있다.

SBOM 관리는 OTA 및 플릿 구성 관리(Fleet Configuration Management)와 직접 연결되어야 한다. 취약한 구성요소가 발견되면 조직은 어떤 펌웨어 릴리스에 해당 구성요소가 포함되어 있는지, 어떤 장치가 해당 릴리스를 실행하고 있는지, 어떤 대체 버전이 문제를 해결하는지, 업데이트를 어떤 방식으로 배포할 것인지를 확인할 수 있어야 한다. 단계적 배포(Staged Rollout), 모니터링, 롤백(Rollback), 업데이트 후 검증(Post-update Verification)을 적용하면 취약점 인텔리전스를 단순한 보안 보고서가 아니라 통제된 운영 수정(Controlled Operational Remediation)으로 전환할 수 있다.

공급망 관리(Supply-chain Management)는 동일한 가시성을 외부 소프트웨어 공급업체까지 확장한다. 공급업체는 제공하는 라이브러리, SDK, 드라이버, 바이너리 모듈(Binary Module)에 대해 정확한 구성요소 정보, 보안 권고, 업데이트 메커니즘, 취약점 대응 프로세스(Vulnerability-response Process)를 제공해야 한다. 내부적으로는 출처 기록(Provenance Record)을 통해 소스 구성요소, 빌드 환경(Build Environment), 서명된 산출물(Signed Artifact), SBOM, 시험 증거(Test Evidence), 릴리스된 펌웨어를 연결하여 소프트웨어 구성과 보안 의사결정에 대한 추적성을 유지해야 한다.

취약점 스캐닝과 SBOM 관리는 궁극적으로 소프트웨어 구성을 식별하고, 새롭게 알려진 취약점을 탐지하고, 실제 시스템에 미치는 영향을 평가하고, 취약점을 수정하거나 완화하고, 수정된 소프트웨어를 검증하고, 배포 기록을 갱신하는 지속적인 수명주기 루프(Continuous Lifecycle Loop)를 형성한다. 이를 보안 개발(Secure Development), 서명된 OTA 업데이트(Signed OTA Update), 구성 관리(Configuration Management), 모니터링, 사고 대응(Incident Response)과 통합하면 장기간 운영되는 임베디드 및 로봇 시스템의 신뢰성을 지속적으로 유지하는 데 필요한 소프트웨어 가시성과 보안 대응 능력을 확보할 수 있다.

## 11.08 Penetration Testing: Embedded Firmware Analysis

![](images/image8.png){width="7.268055555555556in" height="7.268055555555556in"}

임베디드 시스템(Embedded System)의 침투 테스트(Penetration Testing)는 공격자의 관점에서 제품을 조사하여 보안 취약점(Security Weakness)을 식별하려는 통제된 보안 평가(Controlled Security Assessment)이다. 일반적인 소프트웨어 테스트와 달리 펌웨어(Firmware), 하드웨어 인터페이스(Hardware Interface), 부팅 메커니즘(Boot Mechanism), 통신 채널(Communication Channel), 자격 증명(Credential), 물리적 접근(Physical Access)을 함께 고려한다. 로봇에서는 모션(Motion), 액추에이터(Actuator), 센싱(Sensing), 안전 기능(Safety Function), 운영 가용성(Operational Availability)에 미칠 수 있는 영향도 함께 고려해야 한다.

효과적인 침투 테스트는 명확한 범위(Scope)와 승인(Authorization)을 정의하는 것에서 시작한다. 엔지니어는 테스트를 시작하기 전에 대상 하드웨어 리비전(Hardware Revision), 펌웨어 버전, 인터페이스, 네트워크 환경, 테스트 계정(Test Account), 허용된 기법, 안전 제한사항(Safety Restriction)을 정의한다. 실제 운영 중인 로봇에는 하드웨어를 손상시키거나 위험한 움직임을 발생시킬 수 있는 통제되지 않은 실험을 수행해서는 안 된다. 전용 실험실 장치, 격리된 네트워크(Isolated Network), 시뮬레이션(Simulation), 통제된 테스트 픽스처(Test Fixture)는 공격적인 보안 테스트를 보다 안전하게 수행할 수 있는 환경을 제공한다.

정찰(Reconnaissance)은 임베디드 공격 표면(Embedded Attack Surface)의 초기 지도를 구축하는 과정이다. 테스터(Tester)는 프로세서, 메모리 장치, 통신 제어기, 외부 커넥터(External Connector), 이동식 저장장치(Removable Storage), 무선 인터페이스(Wireless Interface), 네트워크 서비스(Network Service), 부팅 모드(Boot Mode), 유지보수 경로(Maintenance Path)를 식별한다. 문서, 펌웨어 메타데이터(Firmware Metadata), 노출된 포트, 서비스 배너(Service Banner), 업데이트 패키지(Update Package), 하드웨어 검사를 통해 시스템의 구성과 신뢰 경계(Trust Boundary)를 파악할 수 있다. 이러한 정보는 이후의 심층적인 펌웨어 및 인터페이스 분석을 위한 방향을 제공한다.

펌웨어 획득(Firmware Acquisition)은 펌웨어 이미지가 장치 내부의 소프트웨어 아키텍처(Software Architecture)를 드러내는 경우가 많기 때문에 기본적인 단계이다. 이미지는 공식 업데이트 패키지, 개발 산출물(Development Artifact), 이동식 저장장치, 접근 가능한 플래시 장치(Flash Device), 승인된 디버그 인터페이스(Debug Interface) 등을 통해 확보할 수 있다. 테스터는 분석의 재현성(Reproducibility)을 유지하기 위해 해시(Hash)와 출처 정보(Provenance)를 기록한다. 보호 메커니즘을 우회하면 독점 코드와 민감한 자격 증명이 노출될 수 있으므로 펌웨어 추출은 승인된 대상에 대해서만 수행해야 한다.

정적 펌웨어 분석(Static Firmware Analysis)은 펌웨어 이미지를 실행하지 않은 상태에서 검사하는 방법이다. 분석자는 파일 형식(File Format), 프로세서 아키텍처(Processor Architecture), 부트 구성요소(Boot Component), 파티션 구조(Partition Layout), 파일 시스템(Filesystem), 라이브러리, 구성 파일(Configuration File), 인증서(Certificate), 스크립트(Script), 실행 바이너리(Executable Binary)를 식별한다. 임베디드 리눅스 펌웨어에서는 커널 이미지(Kernel Image), 장치 트리(Device Tree), 루트 파일 시스템(Root Filesystem), 서비스, 패키지 정보를 확인할 수 있으며, MCU 펌웨어에서는 원시 바이너리 영역(Raw Binary Region), 벡터 테이블(Vector Table), 상수(Constant), 문자열(String), 메모리 맵(Memory Mapping)을 분석해야 할 수 있다.

소스 코드를 사용할 수 없는 경우 바이너리 역공학(Binary Reverse Engineering)은 더욱 깊은 가시성을 제공한다. 디스어셈블러(Disassembler)와 디컴파일러(Decompiler)는 기계어 코드(Machine Code)의 일부를 함수, 제어 흐름(Control Flow), 데이터 참조(Data Reference), 프로토콜 처리기(Protocol Handler), 보안 검사(Security Check)를 이해할 수 있는 형태로 복원할 수 있다. 주요 분석 대상에는 인증 루틴(Authentication Routine), 업데이트 검증(Update Verification), 명령 파서(Command Parser), 디버그 기능, 자격 증명 처리(Credential Handling), 특권 연산(Privileged Operation) 등이 포함된다. 역공학은 프로세서와 펌웨어 아키텍처에 대한 이해와 결합될 때 가장 효과적이다.

문자열 및 구성 분석(String and Configuration Analysis)은 심층적인 역공학을 시작하기 전에 보안과 관련된 정보를 빠르게 발견할 수 있다. 펌웨어에는 URL, IP 주소, 사용자 이름, 인증서, 파일 시스템 경로, 진단 명령(Diagnostic Command), API 엔드포인트(API Endpoint), 개발 메시지, 암호화 함수(Cryptographic Function)에 대한 참조가 포함될 수 있다. 하드코딩된 비밀번호(Hard-coded Password), 개인 키(Private Key), 공유 자격 증명(Shared Credential), 문서화되지 않은 유지보수 명령(Undocumented Maintenance Command)은 펌웨어를 추출하는 것만으로 재사용 가능한 접근 수단이 노출될 수 있으므로 특히 심각한 취약점이 된다.

동적 분석(Dynamic Analysis)은 펌웨어가 실행되는 동안 동작을 관찰함으로써 정적 분석을 보완한다. 테스터는 통제된 조건에서 프로세스(Process), 메모리 동작, 파일, 네트워크 연결(Network Connection), 시스템 호출(System Call), 로그, 프로토콜 상호작용(Protocol Interaction)을 모니터링한다. 임베디드 리눅스 시스템에서는 직접 계측(Instrumentation)이 가능할 수 있지만 MCU 펌웨어에서는 실제 대상 하드웨어, 에뮬레이션(Emulation), 디버깅(Debugging), 하드웨어 인 더 루프(Hardware-in-the-Loop) 환경이 필요할 수 있다. 동적 테스트는 정적 바이너리만으로 정확하게 추론하기 어려운 동작을 발견하는 데 유용하다.

하드웨어 인터페이스(Hardware Interface)는 일반적인 네트워크 접근으로는 사용할 수 없는 기능을 제공하는 경우가 많기 때문에 침투 테스트의 중요한 대상이다. UART 콘솔, JTAG, SWD, SPI, I2C, USB, 부팅 구성 핀(Boot Configuration Pin), 테스트 패드(Test Pad), 이동식 저장장치를 검사하여 이러한 인터페이스가 양산 제품에서도 활성화되어 있는지, 인증이 필요한지, 메모리 읽기, 펌웨어 변경, 특권 콘솔(Privileged Console), 보안 구성 변경(Security Configuration Change)을 허용하는지를 확인한다.

부팅 보안 테스트(Boot Security Testing)는 장치가 의도된 신뢰 체인(Chain of Trust)을 실제로 강제하는지 검증한다. 테스터는 변경되거나 서명되지 않은 펌웨어(Unsigned Firmware)가 거부되는지, 모든 실행 가능한 부팅 단계가 검증되는지, 복구 이미지(Recovery Image)가 인증되는지, 취약한 이전 릴리스가 다시 설치될 수 있는지를 조사한다. 보안 부팅(Secure Boot)을 우회하려는 테스트는 통제되고 승인된 범위에서 수행해야 한다. 목적은 보안 아키텍처가 비인가 소프트웨어가 지속적인 실행 코드(Persistent Executable Code)로 자리 잡는 것을 실제로 방지하는지 확인하는 것이다.

통신 테스트(Communication Testing)는 Ethernet, Wi-Fi, Bluetooth, CAN, UART 및 애플리케이션 전용 프로토콜(Application-specific Protocol)과 같은 인터페이스를 평가한다. 분석자는 인증(Authentication), 암호화(Encryption), 인증서 검증(Certificate Validation), 메시지 무결성(Message Integrity), 재전송 공격 저항성(Replay Resistance), 권한 부여(Authorization), 비정상 메시지 처리(Malformed-message Handling), 자원 제한(Resource Limit)을 조사한다. 내부 로봇 네트워크는 신뢰된 필드버스(Fieldbus)가 강력한 송신자 인증 없이 구조적으로 유효한 명령을 받아들일 수 있으므로 특히 주의해야 하며, 침해된 노드가 센서, 액추에이터 또는 다른 제어기에 영향을 미칠 가능성을 평가해야 한다.

퍼즈 테스트(Fuzz Testing)는 파서(Parser)와 통신 인터페이스에 비정상적이거나 예상하지 못한 값, 경계값(Boundary Value), 무작위 입력(Randomized Input)을 체계적으로 제공하는 방법이다. 프로토콜 디코더(Protocol Decoder), 업데이트 처리기(Update Handler), 파일 파서(File Parser), 진단 서비스(Diagnostic Service), 명령 처리기(Command Processor)는 입력 처리 오류가 충돌(Crash), 메모리 손상(Memory Corruption), 서비스 거부(Denial of Service), 의도하지 않은 실행 경로로 이어질 수 있기 때문에 중요한 테스트 대상이다. 효과적인 퍼징(Fuzzing)은 비정상 동작을 발생시킨 정확한 입력을 기록하여 개발자가 결함을 재현하고 분석하고 수정한 후 회귀 시험(Regression Testing)을 수행할 수 있도록 한다.

자격 증명 및 암호화 테스트(Credential and Cryptographic Testing)는 비밀 정보가 어떻게 생성되고 저장되며 접근되고 사용되는지를 검사한다. 테스터는 플릿 공유 자격 증명(Fleet-wide Shared Credential), 취약한 비밀번호, 노출된 키, 예측 가능한 난수(Predictable Randomness), 잘못된 인증서 검증, 오래된 암호화 구성(Obsolete Cryptographic Configuration), 일반 플래시나 파일 시스템에 저장된 비밀 정보를 확인한다. TPM, 보안 요소(Secure Element), 하드웨어 보안 모듈(Hardware Security Module, HSM)이 사용되는 경우 개인 키가 실제로 보호되고 애플리케이션이 의도된 하드웨어 기반 보안 경계(Hardware-backed Security Boundary)를 우회할 수 없는지도 검증해야 한다.

런타임 익스플로잇 테스트(Runtime Exploitation Testing)는 발견된 취약점이 의미 있는 보안 경계(Security Boundary)를 넘어설 수 있는지를 평가한다. 초기에는 애플리케이션 수준의 접근만 제공하는 결함이라도 권한 상승(Privilege Escalation), 지속적인 펌웨어 변경(Persistent Firmware Modification), 자격 증명 추출(Credential Extraction), 다른 로봇 구성요소로의 측면 이동(Lateral Movement)을 가능하게 한다면 훨씬 심각한 문제가 된다. 특히 통제되지 않은 개념 증명(Proof of Concept) 동작이 물리적 장비나 안전 기능에 영향을 줄 수 있는 경우에는 실제 영향을 입증하는 데 필요한 최소 수준의 익스플로잇(Exploitation)만 수행해야 한다.

테스트 결과는 관찰 사항(Observation), 취약점(Vulnerability), 악용 가능성(Exploitability), 입증된 영향(Demonstrated Impact)을 명확하게 구분해야 한다. 각각의 발견 사항에는 영향을 받는 하드웨어 및 펌웨어 버전, 진입 지점(Entry Point), 필요한 공격자 접근 수준, 재현 조건(Reproduction Condition), 보안 영향(Security Consequence), 근거 자료(Supporting Evidence), 권장 완화 방법(Recommended Mitigation)이 포함되어야 한다. 액추에이터 명령을 변경하거나 안전 기능을 비활성화하는 취약점은 일반적인 정보보안 손실보다 더 큰 결과를 초래할 수 있으므로 심각도는 일반적인 점수만이 아니라 실제 임베디드 시스템의 맥락을 반영해야 한다.

침투 테스트 결과는 취약점 수정(Remediation)과 회귀 시험으로 직접 연결되어야 한다. 개발자는 취약한 코드를 수정하고 불필요한 인터페이스를 비활성화하며 인증을 강화하고 자격 증명을 보호하고 의존성(Dependency)을 업데이트하거나 부팅 및 네트워크 정책을 변경한다. 수정된 펌웨어는 다시 빌드하고 서명한 후 재시험하여 취약점이 제거되었으며 기능, 타이밍(Timing), 안전 측면에서 새로운 회귀 문제가 발생하지 않았는지를 검증해야 한다. 발견 사항은 위협 모델(Threat Model), 보안 요구사항(Security Requirement), 테스트 케이스(Test Case), 향후 설계 지침(Design Guideline)에도 반영되어야 한다.

따라서 임베디드 침투 테스트와 펌웨어 분석(Firmware Analysis)은 제품 출시 전에 한 번 수행하는 공격 시험이 아니라 지속적인 보안 보증 활동(Continuous Assurance Activity)으로 이해해야 한다. 하드웨어 리비전, 펌웨어, 통신 프로토콜, OTA 메커니즘, 서드파티 구성요소(Third-party Component), 배포 환경이 변경될 때마다 재평가가 필요하다. 이를 위협 모델링(Threat Modeling), 보안 부팅, 취약점 스캐닝(Vulnerability Scanning), SBOM 관리, 런타임 모니터링(Runtime Monitoring), 사고 대응(Incident Response)과 결합하면 로봇의 전체 수명주기에 걸쳐 계층화된 보안 제어가 현실적인 공격 경로를 견딜 수 있다는 실질적인 증거를 확보할 수 있다.

## 11.09 Security Certification: IEC 62443 / FIPS 140-2

![](images/image9.png){width="7.268055555555556in" height="7.268055555555556in"}

보안 인증(Security Certification)은 임베디드 및 산업 시스템(Embedded and Industrial System)이 정의된 사이버보안 요구사항(Cybersecurity Requirement)에 따라 설계, 구현, 평가되었음을 보여주는 체계적인 증거를 제공한다. 표준(Standard)과 검증 프로그램(Validation Program)은 반복 가능한 엔지니어링 프로세스, 기술적 통제(Technical Control), 문서화, 독립적인 평가(Independent Assessment)를 확립함으로써 조직이 비공식적인 보안 주장에만 의존하지 않도록 한다. 로봇 플랫폼(Robotic Platform)의 인증은 사이버보안 통제를 하드웨어, 펌웨어(Firmware), 네트워크, 업데이트 메커니즘(Update Mechanism), 운영 수명주기 관리(Operational Lifecycle Management)와 연결해야 한다.

IEC 62443은 산업 자동화 및 제어 시스템(Industrial Automation and Control System)을 위해 개발된 사이버보안 표준군(Cybersecurity Standards Family)이다. 산업 보안은 자산 소유자(Asset Owner), 시스템 통합업체(System Integrator), 제품 공급업체(Product Supplier), 네트워크, 제어기, 운영 프로세스 사이의 상호작용에 의존하기 때문에 적용 범위는 개별 장치를 넘어선다. 따라서 공장, 물류 시설, 인프라 및 기타 운영 기술(Operational Technology, OT) 환경에 배치되는 임베디드 제어기와 로봇 시스템은 보다 광범위한 산업 사이버보안 아키텍처(Industrial Cybersecurity Architecture)의 일부로 고려할 수 있다.

IEC 62443의 핵심 개념 중 하나는 심층 방어(Defense in Depth)이며, 하나의 보안 경계에만 의존하는 대신 여러 보안 통제 계층(Security Control Layer)을 통해 시스템의 서로 다른 영역을 보호한다. 네트워크 분할(Network Segmentation), 인증(Authentication), 권한 부여(Authorization), 보안 통신(Secure Communication), 시스템 무결성(System Integrity), 제한된 인터페이스(Restricted Interface), 모니터링(Monitoring), 사고 대응(Incident Response)이 상호 보완적인 보호를 제공한다. 하나의 계층이 침해되더라도 추가적인 통제가 공격의 확산을 제한하고 공격자가 전체 로봇 또는 산업 환경에 무제한으로 접근하는 것을 방지해야 한다.

존과 컨듀잇(Zones and Conduits)은 보안 요구사항과 통신 관계에 따라 산업 시스템을 구성하기 위한 아키텍처 방법을 제공한다. 유사한 보안 요구를 가진 구성요소를 존(Zone)으로 그룹화하고, 존 사이의 통제된 통신 경로를 컨듀잇(Conduit)으로 표현할 수 있다. 따라서 로봇 제어기(Robot Controller), 플릿 서버(Fleet Server), 엔지니어링 워크스테이션(Engineering Workstation), 안전 제어기(Safety Controller), 기업 네트워크(Enterprise Network)를 서로 다른 보안 존에 배치하고 이들 사이의 통신을 명시적으로 통제할 수 있다.

IEC 62443은 식별 및 인증 제어(Identification and Authentication Control), 사용 제어(Use Control), 시스템 무결성, 데이터 기밀성(Data Confidentiality), 제한된 데이터 흐름(Restricted Data Flow), 이벤트에 대한 적시 대응(Timely Response to Events), 자원 가용성(Resource Availability)을 다루는 기본 보안 요구사항(Foundational Security Requirements)을 정의한다. 이러한 요구사항은 광범위한 사이버보안 목표를 산업 구성요소와 시스템에 적용할 수 있는 엔지니어링 관점으로 변환한다. 임베디드 제품 설계자는 규정 준수(Compliance)를 개발 완료 후 수행하는 문서 작업으로 취급하기보다 관련 기술적 메커니즘을 이러한 목표에 대응시켜야 한다.

보안 수준(Security Level)은 서로 다른 유형의 위협에 대해 기대되는 보호 정도를 표현하기 위한 프레임워크를 제공한다. 높은 보안 요구 수준에서는 더욱 높은 능력과 동기를 가진 공격자에 대한 강력한 저항성이 요구된다. 적절한 목표 수준은 시스템 위험(System Risk), 노출도(Exposure), 운영상의 결과(Operational Consequence), 배포 환경(Deployment Environment)에 따라 결정된다. 따라서 격리된 연구실에서 운영되는 로봇과 중요 산업 인프라(Critical Industrial Infrastructure)에서 운영되는 연결형 자율 기계(Connected Autonomous Machine)는 서로 다른 보안 수준을 요구할 수 있다.

IEC 62443은 또한 안전한 제품 개발 수명주기(Secure Product Development Lifecycle)를 강조한다. 아키텍처 및 설계 단계에서 보안 요구사항을 정의하고 이후 위협 모델링(Threat Modeling), 안전한 구현(Secure Implementation), 검증(Verification), 취약점 관리(Vulnerability Management), 업데이트 계획(Update Planning), 통제된 릴리스(Controlled Release)를 수행해야 한다. 배포 이후 발견된 보안 결함(Security Defect)에 대해서도 문서화된 처리 및 수정 프로세스(Remediation Process)가 필요하다. 이러한 수명주기 접근법은 펌웨어, 운영체제, 미들웨어(Middleware), 통신 소프트웨어가 수년간 배포되어 운영될 수 있는 로봇에서 특히 중요하다.

구성요소 수준 보안 평가(Component-level Security Assessment)는 임베디드 제품이 보다 큰 시스템에 안전하게 통합되기 위해 필요한 기능을 제공하는지 검토한다. 인증, 역할 기반 접근 제어(Role-based Access Control), 소프트웨어 무결성 검증(Software Integrity Verification), 보호된 통신(Protected Communication), 안전한 구성(Secure Configuration), 감사 기능(Audit Capability), 업데이트 메커니즘 등이 구성요소 보안 아키텍처에 포함될 수 있다. 제품 인증만으로 전체 배포 시스템이 자동으로 안전해지는 것은 아니며 시스템 구성, 네트워크 아키텍처, 통합 방식, 운영 절차 역시 중요하다.

FIPS 140-2는 보다 좁지만 중요한 보안 영역인 암호화 모듈(Cryptographic Module)의 검증을 다룬다. 전체 로봇이나 산업 시스템의 사이버보안을 인증하는 것이 아니라 암호화 기능(Cryptographic Function)을 구현하는 모듈에 대한 보안 요구사항을 규정한다. 암호화 모듈은 소프트웨어, 펌웨어, 하드웨어 또는 이들의 조합으로 구현될 수 있다. 검증(Validation)은 정의된 모듈이 지정된 운영 조건(Operational Condition)에서 해당 요구사항을 만족한다는 증거를 제공한다.

따라서 FIPS 140-2를 고려할 때 암호화 모듈 경계(Cryptographic Module Boundary)를 정의하는 것이 매우 중요하다. 설계자는 어떤 소프트웨어와 하드웨어가 검증 대상 암호화 모듈에 포함되는지, 그리고 어떤 인터페이스가 해당 경계를 통과하는지를 식별해야 한다. 키(Key), 암호화 매개변수(Cryptographic Parameter), 입력 데이터(Input Data), 출력 데이터(Output Data), 제어 정보(Control Information), 상태 정보(Status Information)는 정의된 아키텍처에 따라 처리되어야 한다. 보안 주장은 검증된 모듈과 구성(Configuration)에 적용되며 해당 모듈을 사용하는 모든 애플리케이션으로 자동 확장되는 것은 아니다.

FIPS 140-2는 암호화 모듈 보안의 여러 측면에 대해 점진적으로 강화되는 요구사항을 가진 복수의 보안 수준(Security Level)을 정의한다. 수준과 구현 방식에 따라 모듈 사양(Module Specification), 포트와 인터페이스(Ports and Interfaces), 역할과 인증(Roles and Authentication), 유한 상태 동작(Finite-state Behavior), 물리적 보안(Physical Security), 암호화 키 관리(Cryptographic Key Management), 전자기적 고려사항(Electromagnetic Considerations), 자체 시험(Self-test), 설계 보증(Design Assurance), 특정 공격에 대한 완화(Mitigation of Specified Attacks) 등이 요구될 수 있다. 선택되는 수준은 조달 요구사항(Procurement Requirement)과 목표 보안 환경을 반영해야 한다.

승인된 암호화 알고리즘(Approved Cryptographic Algorithm) 역시 중요한 고려사항이다. 해당 FIPS 검증을 요구하는 시스템에서는 기술적으로 강력하다는 이유만으로 임의의 암호화 또는 해싱 알고리즘(Hashing Algorithm)을 사용할 수 있는 것은 아니다. 암호화 구현은 관련 검증 프레임워크에서 허용되는 알고리즘과 운영 구성을 사용해야 한다. 따라서 알고리즘 구현, 키 크기(Key Size), 운영 모드(Mode of Operation), 난수 생성(Random-number Generation), 키 설정(Key Establishment), 모듈 구성을 검증된 보안 경계의 일부로 관리해야 한다.

자체 시험(Self-test)은 암호화 기능이 모듈 초기화 또는 정의된 동작을 수행할 때 올바르게 작동한다는 신뢰를 확보하는 데 도움을 준다. 모듈은 무결성 검사(Integrity Check), 알려진 정답 시험(Known-answer Test) 또는 기타 요구되는 진단 절차(Diagnostic Procedure)를 수행하고 중요한 시험에 실패하면 오류 상태(Error State)로 진입할 수 있다. 임베디드 설계자는 이러한 동작을 부팅(Boot), 복구(Recovery), 워치독 처리(Watchdog Handling), 시스템 초기화(System Initialization)에 반영하여 암호화 기능의 실패가 감지되지 않은 불안전한 동작으로 이어지지 않도록 해야 한다.

키 관리(Key Management)는 암호화 인증을 보다 광범위한 임베디드 보안 아키텍처와 연결한다. 키는 전체 수명주기에 걸쳐 안전하게 생성(Generation), 설정(Establishment), 저장(Storage), 사용(Use), 교체(Rotation), 폐기(Destruction)되어야 한다. TPM, HSM, 보안 요소(Secure Element), 보호된 MCU 저장소(Protected MCU Storage), 검증된 암호화 모듈을 통해 민감한 정보를 격리할 수 있다. 그러나 주변 애플리케이션 역시 권한 부여, 인증서 정책(Certificate Policy), 보안 프로비저닝(Secure Provisioning), 운영 절차를 적용하여 정상적인 암호화 메커니즘이 잘못 사용되지 않도록 해야 한다.

인증 계획(Certification Planning)은 개발 후반의 변경 비용이 커질 수 있기 때문에 아키텍처 개발 단계에서 시작해야 한다. 하드웨어 선택, 암호화 라이브러리(Cryptographic Library), 운영체제 구성, 디버그 인터페이스(Debug Interface), 키 저장소(Key Storage), 업데이트 메커니즘, 네트워크 아키텍처, 개발 프로세스가 모두 평가 범위(Assessment Scope)에 영향을 줄 수 있다. 개발 과정 전체에서 요구사항 추적성(Requirements Traceability)과 시험 증거(Test Evidence)를 유지하면 구현된 보안 통제가 목표 인증 요구사항과 대응한다는 것을 입증하는 데 필요한 노력을 줄일 수 있다.

따라서 IEC 62443과 FIPS 140-2는 임베디드 보안(Embedded Security)의 서로 다르지만 상호 보완적인 측면을 다룬다. IEC 62443은 시스템, 구성요소, 아키텍처, 개발 프로세스, 수명주기 보안(Lifecycle Security)을 포괄하는 보다 광범위한 산업 사이버보안 프레임워크를 제공하는 반면, FIPS 140-2는 암호화 모듈 보안과 검증에 집중한다. 이를 보안 부팅(Secure Boot), 보호된 키(Protected Key), TLS, 취약점 관리, 침투 테스트(Penetration Testing), 서명된 OTA 업데이트(Signed OTA Update), 지속적인 모니터링(Continuous Monitoring)과 결합하면 인증은 독립적인 규정 준수 활동이 아니라 통합된 보안 보증 전략(Integrated Assurance Strategy)의 일부가 된다.

## 11.10 Robot Embedded Security Incident Response Procedure

![](images/image10.png){width="7.268055555555556in" height="7.268055555555556in"}

로봇 임베디드 보안 사고 대응 절차(Robot Embedded Security Incident Response Procedure)는 로봇 제어기(Robotic Controller), 펌웨어(Firmware), 통신 네트워크(Communication Network), 센서(Sensor), 액추에이터(Actuator), 지원 인프라(Supporting Infrastructure)에 영향을 미치는 사이버보안 사고(Cybersecurity Incident)를 조직이 어떻게 탐지하고 평가하며 격리하고 제거한 후 복구할 것인지를 정의한다. 일반적인 IT 보안 사고와 달리 침해된 로봇은 물리적 결과(Physical Consequence)를 발생시킬 수 있으므로 사이버보안 대응은 사고 수명주기(Incident Lifecycle) 전체에서 기능 안전(Functional Safety), 운영 연속성(Operational Continuity), 인명 보호(Human Protection)와 조정되어야 한다.

사고 대비(Incident Preparation)는 공격이 발생하기 전에 시작된다. 각각의 로봇 플랫폼에는 명확한 보안 책임(Security Ownership), 에스컬레이션 경로(Escalation Path), 비상 연락망(Emergency Contact), 자산 목록(Asset Inventory), 펌웨어 기준선(Firmware Baseline), 네트워크 구성도(Network Diagram), 장치 신원(Device Identity), 로깅 정책(Logging Policy), 복구 이미지(Recovery Image), 자격 증명 관리 절차(Credential-management Procedure)가 정의되어 있어야 한다. 또한 실제 사고가 발생했을 때 중요한 격리 결정을 신속하고 명확하게 실행할 수 있도록 안전한 통신 채널과 사전에 정의된 대응 권한(Response Authority)을 확립해야 한다.

보안 모니터링(Security Monitoring)은 의심스러운 동작을 식별하는 데 필요한 증거를 제공한다. 로봇 제어기, 게이트웨이(Gateway), 플릿 서버(Fleet Server), 무선 네트워크(Wireless Network), 인증 서비스(Authentication Service), 업데이트 시스템(Update System)은 유용한 보안 텔레메트리(Security Telemetry)를 생성해야 한다. 반복적인 인증 실패, 예상하지 못한 네트워크 연결, 펌웨어 무결성 오류(Firmware Integrity Error), 비정상적인 권한 변경, 이상 명령 시퀀스(Command Sequence), 비인가 구성 변경, 디버그 인터페이스(Debug Interface) 활동, 설명되지 않는 리셋, 정상 운영 동작에서의 이탈 등이 사고 징후가 될 수 있다.

탐지 메커니즘(Detection Mechanism)은 사이버보안 지표(Cybersecurity Indicator)와 로봇 특화 운영 신호(Robot-specific Operational Signal)를 결합해야 한다. 예상하지 못한 액추에이터 명령, 일관되지 않은 센서 데이터, 설명되지 않는 경로 변경, 비정상적인 통신 타이밍(Communication Timing), 반복적인 안전 상태 전환(Safety-state Transition), 비인가 노드에서 발생한 명령 등은 시스템 침해를 나타낼 수 있다. 보안 로그(Security Log)를 로봇 상태와 물리적 동작(Physical Behavior)에 연계하면 일반적인 장비 고장과 사이버 사고를 구분하고 잠재적인 악의적 활동을 보다 완전하게 이해할 수 있다.

이벤트가 탐지되면 초기 분류(Initial Triage)를 통해 해당 이벤트가 확인된 보안 사고, 의심되는 침해(Suspected Compromise), 소프트웨어 결함, 하드웨어 고장 또는 정상적인 이상 현상(Benign Anomaly)인지를 판단한다. 분석자는 영향을 받은 장치, 펌웨어 버전, 네트워크 인터페이스, 자격 증명, 관찰된 지표, 운영상의 결과를 식별한다. 심각도(Severity)는 기밀성(Confidentiality)이나 데이터 손실뿐만 아니라 해당 이벤트가 모션(Motion), 액추에이터, 내비게이션(Navigation), 안전 기능(Safety Function), 생산 프로세스 또는 주변 작업자에게 영향을 줄 수 있는지도 고려해야 한다.

사이버 침해가 위험한 물리적 동작을 발생시킬 수 있다면 안전(Safety)이 최우선되어야 한다. 영향을 받은 로봇은 사전에 정의된 안전 상태(Safe State)로 전환하거나 특정 자율 기능(Autonomous Function)을 중단하고 운영 능력을 제한하거나 승인된 대체 제어 메커니즘(Authorized Fallback Mechanism)으로 제어를 전환해야 할 수 있다. 전원, 통신 또는 제어기를 갑작스럽게 차단하는 행위 자체가 모바일 로봇(Mobile Robot), 매니퓰레이터(Manipulator), 산업 기계(Industrial Machinery), 협업 플릿(Coordinated Fleet)에 위험을 발생시킬 수 있으므로 보안 격리(Security Containment)는 안전 엔지니어링(Safety Engineering)과 조정되어야 한다.

격리(Containment)는 공격자가 계속 활동하거나 다른 시스템으로 확산하는 능력을 제한한다. 사고 유형에 따라 로봇을 플릿 네트워크에서 격리하거나 침해된 계정(Compromised Account)을 비활성화하고 인증서를 폐기(Revocation)하며 의심스러운 엔드포인트(Endpoint)를 차단하고 원격 접근을 제한하거나 취약한 서비스를 비활성화하고 영향을 받은 네트워크 존(Network Zone)을 분리할 수 있다. 격리는 가능한 경우 필수적인 안전 통신을 유지하면서 침해된 구성요소가 신뢰된 로봇이나 인프라에 영향을 미치지 못하도록 해야 한다.

플릿 환경(Fleet Environment)은 하나의 자격 증명이나 소프트웨어 취약점이 동시에 많은 로봇에 영향을 미칠 수 있으므로 특별한 주의가 필요하다. 사고 대응팀(Incident Response Team)은 문제가 단일 장치에 한정되는지, 특정 펌웨어 릴리스, 하드웨어 리비전(Hardware Revision), 특정 사이트 또는 전체 배포 집단(Deployed Population)에 영향을 미치는지를 판단해야 한다. 자산 목록, 플릿 구성 데이터베이스(Fleet Configuration Database), 소프트웨어 자재 명세서(Software Bill of Materials, SBOM)를 활용하면 취약한 구성요소, 인증서, 소프트웨어 버전 또는 노출된 서비스를 공유하는 시스템을 신속하게 식별할 수 있다.

안전과 운영 조건이 허용하는 경우 파괴적인 수정 작업을 수행하기 전에 증거 보존(Evidence Preservation)을 실시해야 한다. 관련 증거에는 펌웨어 해시(Firmware Hash), 메모리 캡처(Memory Capture), 파일 시스템 이미지(Filesystem Image), 보안 로그, 네트워크 트레이스(Network Trace), 인증 기록(Authentication Record), 업데이트 이력(Update History), 구성 상태(Configuration State), 타임스탬프(Timestamp), 물리적 관찰 기록(Physical Observation)이 포함될 수 있다. 증거의 출처(Provenance)와 적절한 증거 관리 연속성(Chain of Custody)을 유지하면 근본 원인 분석(Root-cause Analysis), 규제 조사(Regulatory Investigation), 공급업체 협력, 이후의 사고 타임라인 검증을 지원할 수 있다.

조사(Investigation)는 사고가 어떻게 발생했으며 공격자가 무엇을 수행했는지를 재구성하는 과정이다. 분석자는 초기 접근 경로(Initial Access Path), 악용된 취약점, 침해된 자격 증명, 영향을 받은 권한 수준(Privilege Level), 지속성 메커니즘(Persistence Mechanism), 측면 이동(Lateral Movement), 영향을 받은 자산을 파악한다. 펌웨어 분석(Firmware Analysis), 네트워크 분석(Network Analysis), 취약점 스캐닝(Vulnerability Scanning), 침투 테스트(Penetration Testing) 기법, 정상 소프트웨어 기준선(Known-good Software Baseline)과의 비교를 통해 실행 코드, 부팅 구성요소, 설정 또는 저장된 비밀 정보가 변경되었는지를 판단할 수 있다.

근절(Eradication)은 단순히 영향을 받은 로봇의 연결을 차단하는 것이 아니라 침해의 원인과 지속성을 제거하는 과정이다. 취약한 소프트웨어를 교체하고 비인가 코드를 제거하며 신뢰할 수 있는 소스로부터 펌웨어를 다시 빌드하고 자격 증명을 교체(Rotation)하며 인증서를 폐기하고 불필요한 인터페이스를 비활성화하거나 안전하지 않은 구성을 수정하고 악용된 네트워크 경로를 차단할 수 있다. 장치 상태의 신뢰성을 확인할 수 없다면 부분적인 수리를 시도하는 것보다 검증된 클린 이미지(Verified Clean Image)를 이용하여 복구하는 것이 더 안전할 수 있다.

자격 증명 침해(Credential Compromise)는 임베디드 장치가 장기간 유지되는 신원(Long-lived Identity)을 사용하는 경우가 많기 때문에 체계적인 대응이 필요하다. 사고와 관련된 장치 인증서(Device Certificate), API 자격 증명, 플릿 인증 키(Fleet Authentication Key), 유지보수 계정(Maintenance Account), 서비스 토큰(Service Token)을 평가하고 필요한 경우 교체해야 한다. 하드웨어로 보호된 개인 키(Hardware-protected Private Key)가 추출될 수 없었다는 증거가 확인되면 해당 키 자체는 여전히 신뢰할 수 있지만 관련 인증서, 권한 부여 정책(Authorization Policy), 서버 측 신뢰 관계(Server-side Trust Relationship)는 폐기 또는 재프로비저닝(Re-provisioning)이 필요할 수 있다.

복구(Recovery)는 검증된 소프트웨어와 구성을 사용하여 로봇 시스템을 통제된 운영 상태로 되돌리는 과정이다. 펌웨어는 인증(Authentication)과 무결성 검사(Integrity Check)를 거친 후 신뢰할 수 있는 업데이트 메커니즘을 통해 설치되어야 한다. 네트워크 정책, 자격 증명, 인증서, 보안 설정은 승인된 기준선(Approved Baseline)으로 복원해야 한다. 특히 액추에이터나 자율 동작이 영향을 받았을 가능성이 있다면 로봇이 제한 없는 정상 운영으로 복귀하기 전에 기능 시험(Functional Test), 안전 시험(Safety Test), 통신 시험(Communication Test), 보안 검증(Security Verification)을 완료해야 한다.

플릿 복구(Fleet Recovery)는 일반적으로 모든 장치를 즉시 업데이트하는 대신 단계적 배포(Staged Deployment)를 사용해야 한다. 수정된 펌웨어를 먼저 실험실 시스템이나 제한된 운영 그룹에 배포하고 예상하지 못한 동작이 없는지 모니터링한 후 나머지 플릿으로 확대할 수 있다. 기술적으로 적절한 경우 롤백 기능(Rollback Capability)을 제공해야 하지만, 동시에 안티 롤백 보안(Anti-rollback Security)을 통해 이미 취약하다고 알려진 버전으로 복원되는 것을 방지해야 한다. 영향을 받은 모든 로봇의 배포 상태(Deployment Status)는 추적 가능해야 한다.

사고 후 분석(Post-incident Analysis)은 발생한 사건을 전체 보안 아키텍처(Security Architecture)의 개선으로 연결한다. 조직은 근본 원인, 대응 효과(Response Effectiveness), 탐지 공백(Detection Gap), 격리 지연(Containment Delay), 영향을 받은 자산, 복구 결과, 교훈(Lessons Learned)을 문서화해야 한다. 이후 동일한 공격 경로가 다시 악용되기 어렵도록 위협 모델(Threat Model), 보안 요구사항(Security Requirement), 모니터링 규칙(Monitoring Rule), 침투 테스트, SBOM 기록, 취약점 관리 정책(Vulnerability-management Policy), 프로비저닝 절차(Provisioning Procedure), 개발 관행(Development Practice)을 갱신해야 한다.

따라서 성숙한 로봇 임베디드 보안 사고 대응 절차는 대비(Preparation), 탐지(Detection), 초기 분류(Triage), 안전 평가(Safety Assessment), 격리(Containment), 증거 보존(Evidence Preservation), 조사(Investigation), 근절(Eradication), 복구(Recovery), 개선(Improvement)으로 이어지는 지속적인 순환 구조를 형성한다. 이를 보안 부팅(Secure Boot), 보호된 장치 신원(Protected Identity), 서명된 OTA 업데이트(Signed OTA Update), 취약점 관리(Vulnerability Management), 플릿 모니터링(Fleet Monitoring), 네트워크 분할(Network Segmentation), 기능 안전과 통합하면 로봇 시스템은 침해의 영향을 제한하고 신뢰할 수 있는 운영 상태를 복원하며 전체 운영 수명주기 동안 보안 회복탄력성(Security Resilience)을 지속적으로 강화할 수 있다.
