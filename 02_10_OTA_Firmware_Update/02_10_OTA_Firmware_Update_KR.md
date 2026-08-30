**Volume 02 Embedded Software Engineering**

# 10. OTA Firmware Update

## 10.01 OTA Update Architecture Overview: System Perspective

![](images/image1.png){width="7.268055555555556in" height="7.268055555555556in"}

무선 업데이트(OTA, Over-the-Air) 아키텍처(architecture)는 임베디드 로봇 시스템(embedded robotic system)에서 단순히 새로운 펌웨어 이미지(firmware image)를 장치로 전송하는 메커니즘이 아니다. 이는 소프트웨어 개발(software development), 릴리스 관리(release management), 업데이트 인프라(update infrastructure), 통신 네트워크(communication network), 임베디드 업데이트 에이전트(embedded update agent), 부트로더(bootloader), 영구 저장장치(persistent storage), 런타임 감시(runtime supervision), 복구 메커니즘(recovery mechanism)을 연결하는 통합 시스템이다. 이러한 아키텍처는 로봇의 전체 운용 수명 동안 소프트웨어를 지속적으로 발전시키면서도 시스템 가용성(system availability)과 안전성(safety)을 유지해야 한다.

시스템 관점(system perspective)에서 OTA 수명주기(lifecycle)는 개발 및 릴리스 환경(development and release environment)에서 시작된다. 펌웨어(firmware), 운영체제 구성요소(operating-system component), 장치 드라이버(device driver), 구성 데이터(configuration data), 미들웨어(middleware), 응용 소프트웨어(application software), 경우에 따라 인공지능 관련 런타임 패키지(AI-related runtime package)가 통제된 파이프라인(controlled pipeline)을 통해 빌드된다. 각 릴리스(release)에는 명확한 버전 정보(version information)와 호환성 메타데이터(compatibility metadata)가 부여되어 설치 가능한 하드웨어 및 소프트웨어 환경을 판단할 수 있어야 한다.

배포(deployment) 전에 업데이트 산출물(update artifact)은 임의의 바이너리 파일(binary file)이 아니라 신뢰할 수 있는 릴리스 패키지(trusted release package)로 변환되어야 한다. 패키지는 일반적으로 페이로드(payload)와 함께 버전, 대상 플랫폼(target platform), 종속성(dependency), 필요한 저장공간, 설치 절차, 호환성 제약조건(compatibility constraint)을 설명하는 메타데이터를 포함한다. 암호학적 해시(cryptographic hash)와 디지털 서명(digital signature)은 무결성(integrity)과 진위성(authenticity)을 보장하며, 로봇은 손상되거나 변조되었거나 승인되지 않은 패키지를 거부할 수 있다.

서버 측(server side)은 OTA 아키텍처의 관리 평면(management plane)을 구성한다. 서버는 소프트웨어 산출물(software artifact), 장치 식별정보(device identity), 릴리스 정책(release policy), 배포 상태(deployment status), 업데이트 이력(update history)을 관리한다. 로봇은 자신의 하드웨어 식별정보, 설치된 소프트웨어 버전, 운용 상태(operational state), 업데이트 기능(update capability)을 주기적으로 전달한다. 관리 시스템은 이러한 정보를 배포 정책과 비교하여 특정 릴리스가 적용 가능한지 판단하고 로봇 모델, 하드웨어 세대, 소프트웨어 기준선(software baseline), 운용 그룹 또는 플릿 세그먼트(fleet segment)에 따라 업데이트를 제어한다.

업데이트 인프라와 로봇 사이의 통신은 신뢰할 수 없는 분산 시스템 경계(unreliable distributed-system boundary)로 취급해야 한다. 로봇 이동 중 무선 연결이 끊길 수 있고 패킷(packet)이 지연될 수 있으며 대용량 이미지 다운로드가 여러 통신 세션에 걸쳐 수행될 수도 있다. 따라서 견고한 OTA 구현은 재개 가능한 전송(resumable transfer), 로컬 캐싱(local caching), 무결성 검사(integrity check), 재시도 정책(retry policy), 대역폭 제어(bandwidth control), 타임아웃 처리(timeout handling)를 지원해야 한다. 다운로드 실패(download failure)와 설치 실패(installation failure)를 구분하여 일시적인 통신 장애가 실행 중인 시스템에 불필요한 영향을 주지 않도록 해야 한다.

로봇 내부에서 OTA 업데이트 에이전트(OTA update agent)는 원격 관리(remote management)와 로컬 임베디드 소프트웨어(local embedded software)를 연결하는 핵심 인터페이스(interface)를 제공한다. 에이전트는 배포 명령을 수신하고 선행 조건(prerequisite)을 평가하며 산출물을 다운로드하고 메타데이터와 서명을 검증한 뒤 패키지를 영구 저장장치에 스테이징(staging)한다. 또한 배터리 수준, 사용 가능한 디스크 공간, 네트워크 상태, 현재 임무 상태, 온도 및 플랫폼별 제약조건을 확인한 후 중요 펌웨어의 설치를 허용해야 한다.

설치(installation)는 가능한 범위에서 정상 실행(normal execution)과 격리되어야 한다. 모터, 센서, 통신 버스(communication bus), 안전 관련 기능을 제어하는 실행 이미지를 동작 중에 변경하면 정의되지 않은 동작(undefined behavior)이 발생할 수 있다. 따라서 OTA 아키텍처는 응용 프로그램 종료(application shutdown), 서비스 전환(service transition), 저장장치 동기화(storage synchronization), 재부팅 순서(reboot sequencing)를 조정한다. 로봇에서는 중요 펌웨어 변경 전에 도킹(docking), 충전(charging), 임무 완료(mission completion), 유지보수 상태(maintenance state) 진입을 요구할 수도 있다.

A/B 파티셔닝(A/B partitioning)은 고신뢰성 업데이트(high-reliability update)를 위한 중요한 아키텍처 패턴(architectural pattern)을 제공한다. 현재 활성화된 소프트웨어는 한 파티션(partition)에서 실행되고 새로운 이미지는 비활성 파티션(inactive partition)에 기록된다. 설치 과정에서 기존 이미지는 변경되지 않는다. 검증이 완료되면 부트 설정(boot configuration)이 다음 시작 시 후보 파티션(candidate partition)을 선택함으로써 기존의 정상 소프트웨어와 새로운 소프트웨어 사이에 트랜잭션 경계(transactional boundary)를 형성한다.

따라서 부트로더(bootloader)는 단순한 초기화 구성요소(initialization component)가 아니라 OTA 신뢰 및 복구 체인(trust and recovery chain)의 일부이다. 앞선 내용에서 다룬 보안 부팅(secure boot), 이중 파티션 부팅 설계(dual-partition boot design), 부트로더 기반 복구(bootloader-based recovery)는 이러한 OTA 아키텍처의 기반을 구성한다. 시스템 시작 시 부트 과정은 선택된 이미지를 식별하고 필요한 보안 속성(security property)을 검증하며 후보 소프트웨어를 로드하고 새로운 버전이 허용 가능한 운용 상태에 도달했는지를 판단하는 과정에 참여한다.

단순히 부팅에 성공했다는 사실만으로 업데이트 성공(update success)을 선언해서는 안 된다. 새로 설치된 이미지가 커널(kernel)을 초기화하더라도 모터 컨트롤러(motor controller)와 통신하지 못하거나 구성 데이터를 로드하지 못하고 ROS 2 서비스(service)를 시작하지 못하거나 센서 또는 네트워크 인터페이스(network interface)에 접근하지 못할 수 있다. 따라서 상태 감시 메커니즘(health-monitoring mechanism)이 업데이트 이후의 준비 상태(readiness)를 평가해야 한다. 필수 서비스와 플랫폼 기능이 정의된 승인 기준(acceptance criteria)을 만족한 이후에만 후보 이미지를 새로운 정상 소프트웨어 기준선(known-good software baseline)으로 확정해야 한다.

업데이트 이후 검증(post-update validation)에 실패하면 복구 아키텍처(recovery architecture)는 이전에 검증된 구성(configuration)을 복원해야 한다. 롤백(rollback)은 반복적인 부팅 실패, 워치독 리셋(watchdog reset), 응용 프로그램 충돌(application crash), 상태 검사 실패(health-check failure), 확인 타이머(confirmation timer) 만료 등에 의해 시작될 수 있다. 아키텍처 수준에서 롤백은 OTA 트랜잭션(transaction)을 완결하는 역할을 한다. 즉, 새로운 소프트웨어를 정상 상태로 확정하거나 안정적으로 이전 상태로 복귀할 수 있을 때까지 배포는 완료된 것으로 간주해서는 안 된다.

보안(security)은 전체 업데이트 체인(update chain)에 걸쳐 적용되어야 한다. 전송 암호화(transport encryption)는 통신을 보호하지만 보안 전송만으로 펌웨어 자체의 신뢰성을 보장할 수는 없다. 릴리스 자격증명(release credential), 서명 키(signing key), 업데이트 서버(update server), 장치 식별정보, 부트 검증(boot verification), 로컬 저장장치는 하나의 신뢰 체인(chain of trust)을 구성한다. 따라서 OTA 보안은 단일 암호학적 검사(cryptographic check)가 아니라 펌웨어 서명, 무결성 검증, 공급망 공격 방어(supply-chain attack defense), 하드웨어 보안 메커니즘(hardware security mechanism)을 포함하는 수명주기 속성(lifecycle property)으로 이해해야 한다.

OTA 아키텍처는 또한 이기종 소프트웨어 경계(heterogeneous software boundary)를 고려해야 한다. 하나의 로봇에는 임베디드 리눅스 컴퓨터(embedded Linux computer), GPU 엣지 프로세서(GPU edge processor), 안전 MCU(safety MCU), 모터 컨트롤러, 센서 펌웨어(sensor firmware), 주변장치 마이크로컨트롤러(peripheral microcontroller)가 포함될 수 있으며 각각 서로 다른 업데이트 메커니즘을 사용한다. 이들을 독립적으로 업데이트하면 호환되지 않는 중간 상태가 발생할 수 있으므로 시스템 수준 업데이트 관리자는 종속성 규칙, 호환성 매트릭스(compatibility matrix), 설치 순서, 재부팅 동작을 통합적으로 관리해야 한다.

로봇이 즉시 작업을 중단할 수 없는 경우에는 운용 정책(operational policy)이 특히 중요하다. 중요도가 낮은 패키지는 임무 수행 중에도 다운로드하거나 스테이징할 수 있지만 안전 관련 펌웨어(safety-related firmware)는 통제된 유지보수 시간(controlled maintenance window)을 요구할 수 있다. 다운로드(download), 검증(verification), 스테이징(staging), 활성화(activation), 확인(confirmation)을 서로 분리하면 정상 소프트웨어 상태로의 결정론적 전환(deterministic transition)을 유지하면서 시스템 중단시간(downtime)을 최소화할 수 있다.

플릿 규모(fleet scale)에서 OTA는 장치 수준 설치 문제가 아니라 오케스트레이션 문제(orchestration problem)가 된다. 릴리스는 일반적으로 개발 장치(development unit), 실험실 로봇(laboratory robot), 검증 플릿(validation fleet), 카나리아 장치(canary device), 그리고 점차 확대되는 운영 플릿(production fleet)의 순서로 통제된 배포가 이루어져야 한다. 각 단계에서 수집되는 텔레메트리(telemetry)는 부팅 성공 여부, 충돌, 자원 소비(resource consumption), 통신 안정성, 임무 수행 동작 등에 대한 증거를 제공하며 다음 배포 단계로 진행할지를 결정하는 근거가 된다.

결과적으로 완전한 OTA 아키텍처는 빌드(build), 패키징(packaging), 서명(signing), 게시(publish), 할당(assign), 다운로드(download), 검증(verify), 스테이징(stage), 활성화(activate), 재부팅(reboot), 검증(validate), 확정(commit), 감시(monitor), 필요 시 복구(recover)로 이어지는 폐쇄형 수명주기(closed lifecycle)로 이해할 수 있다. 핵심 목적은 단순한 원격 소프트웨어 전달(remote software delivery)이 아니라 하나의 알려진 시스템 구성(known system configuration)에서 다른 검증된 구성으로 안전하고 통제된 전환을 수행하는 것이다. 물리적 움직임과 안전 메커니즘에 직접 연결되는 로봇에서는 이러한 특성으로 인해 OTA가 신뢰성(reliability), 보안(security), 유지보수성(maintainability), 수명주기 엔지니어링(lifecycle engineering)의 핵심 요소가 된다.

## 10.02 Safe Update Design with A/B Partition

![](images/image2.png){width="7.268055555555556in" height="7.268055555555556in"}

A/B 파티셔닝(A/B partitioning)은 소프트웨어 설치 실패로 인해 임베디드 시스템(embedded system)이 영구적으로 사용할 수 없는 상태가 되는 것을 방지하기 위한 결함 허용 업데이트 아키텍처(fault-tolerant update architecture)이다. 현재 실행 중인 시스템 이미지(system image)를 직접 교체하는 대신 저장장치를 일반적으로 슬롯 A(Slot A)와 슬롯 B(Slot B)라고 부르는 두 개의 부팅 가능한 영역으로 분할한다. 하나의 슬롯은 활성 상태를 유지하고 다른 슬롯은 업데이트 대상이 되어 OTA 과정 대부분에서 정상 동작이 확인된 소프트웨어 환경(known-good software environment)을 보존한다.

정상 운용(normal operation) 중에는 부트로더(bootloader)가 하나의 파티션을 활성 슬롯(active slot)으로 선택하고 운영체제(operating system), 미들웨어(middleware), 드라이버(driver), 로봇 애플리케이션(robot application)이 해당 이미지에서 실행된다. 비활성 슬롯(inactive slot)은 현재 운용에 필요하지 않으므로 실행 중인 이미지를 변경하지 않고 새로운 소프트웨어 릴리스(software release)를 저장할 수 있다. 이러한 분리는 OTA 업데이트를 실행 중 이미지의 직접 덮어쓰기(in-place overwrite)가 아니라 독립적으로 부팅 가능한 두 시스템 상태 사이의 통제된 전환으로 바꾼다.

일반적인 업데이트는 OTA 에이전트(OTA agent)가 승인된 릴리스(authorized release)를 수신하고 장치가 설치 선행 조건(installation prerequisite)을 만족하는지 판단하면서 시작된다. 다운로드와 암호학적 검증(cryptographic verification)이 완료되면 새로운 이미지는 비활성 파티션에 기록된다. 슬롯 A가 현재 활성 상태라면 슬롯 B가 설치 대상이 되고 그 반대의 경우도 동일하다. 기록 중 중단이 발생하더라도 현재 로봇을 운용하는 소프트웨어 구성이 손상되지 않도록 활성 파티션은 변경하지 않아야 한다.

비활성 이미지(inactive image)는 부팅 후보(boot candidate)가 되기 전에 완전히 검증되어야 한다. 검증에는 패키지 메타데이터(package metadata), 이미지 크기, 암호학적 해시(cryptographic hash), 디지털 서명(digital signature), 파일시스템 일관성(filesystem consistency), 버전 호환성(version compatibility), 하드웨어 호환성(hardware compatibility) 확인 등이 포함될 수 있다. 또한 업데이트 프로그램은 필요한 모든 데이터 블록이 정상적으로 기록되었는지 확인해야 하며, 이러한 검사가 통과된 이후에만 다음 재부팅에서 새 슬롯을 사용하도록 부트 메타데이터(boot metadata)를 변경해야 한다.

부트 메타데이터(boot metadata)는 어떤 슬롯을 시작해야 하는지, 어떤 이미지가 정상 상태로 확인되었는지, 후보 이미지가 아직 검증 중인지 등을 기록하기 때문에 설계에서 매우 중요한 요소이다. 구현 방식에 따라 부팅 시도 횟수(boot-attempt counter), 업데이트 상태(update state), 롤백 플래그(rollback flag), 버전 정보(version information)를 추가로 유지할 수 있다. 두 파티션 이미지가 물리적으로 정상이어도 잘못된 부트 메타데이터로 인해 복구가 불가능해질 수 있으므로 이러한 정보 자체도 손상으로부터 보호해야 한다.

새로운 파티션으로의 전환은 업데이트를 즉시 성공으로 확정하기보다 시험 부팅 상태(trial-boot state)를 사용하는 것이 바람직하다. 비활성 슬롯에 설치가 완료되면 부트로더는 해당 슬롯을 다음 부팅 후보로 지정하고 장치를 재부팅한다. 이후 후보 이미지(candidate image)는 시험 상태에서 시작된다. 런타임 검증(runtime validation)이 정상 동작을 확인하기 전까지 이전 활성 파티션은 정상 확인 이미지(known-good image)로 유지되어 후보 이미지가 실패할 경우 복구 경로를 제공한다.

따라서 견고한 부트로더는 부팅 가능(bootable), 후보(candidate), 확인 완료(confirmed), 실패(failed)와 같은 소프트웨어 상태를 구분해야 한다. 후보 파티션을 시작할 때 허용된 부팅 시도 횟수를 감소시키거나 확인 메커니즘(confirmation mechanism)을 시작할 수 있다. 시스템이 성공 확인 전에 반복적으로 리셋된다면 부트로더는 이를 업데이트 실패의 증거로 판단할 수 있다. 이후 결함이 있는 소프트웨어를 계속 부팅하는 대신 이전의 정상 확인 파티션을 자동으로 선택한다.

부팅 후 검증(post-boot validation)은 단순한 커널 시작(kernel startup)을 넘어 안전 메커니즘을 확장한다. 후보 시스템은 중요 서비스가 정상적으로 초기화되고 필요한 하드웨어 인터페이스(hardware interface)가 사용 가능한 상태임을 입증해야 한다. 로봇에서는 모터 컨트롤러(motor controller), 안전 컨트롤러(safety controller), 센서(sensor), 네트워크 인터페이스(network interface), 저장장치(storage), 미들웨어, 핵심 애플리케이션과의 통신 상태를 검사할 수 있다. 정의된 상태 기준(health criteria)을 만족하여 안정적인 운용 상태에 도달한 경우에만 업데이트를 최종 확인한다.

확인 동작(confirmation operation)은 A/B 업데이트 트랜잭션(A/B update transaction)의 커밋 지점(commit point)을 의미한다. 후보 이미지가 필요한 검사를 통과하면 시스템은 새로운 파티션을 정상 확인 상태(known-good)로 기록하고 임시 시험 정보를 제거한다. 이전 활성 슬롯은 이후 업데이트 주기에서 교체될 때까지 백업(backup)으로 유지할 수 있다. 결과적으로 연속적인 릴리스 과정에서 두 슬롯은 활성 이미지와 다음 후보 이미지를 저장하는 역할을 번갈아 수행한다.

실패 처리(failure handling)는 결정론적(deterministic)이어야 한다. 비활성 슬롯에 이미지를 기록하는 동안 전원이 차단되더라도 기존 활성 슬롯은 여전히 부팅할 수 있어야 한다. 검증이 실패하면 활성화(activation)가 수행되어서는 안 된다. 후보 이미지가 부팅 또는 런타임 검증 중 실패하면 부트 제어(boot control)는 이전의 정상 확인 이미지로 돌아가야 한다. 이러한 동작이 A/B 파티셔닝을 사용하는 핵심 이유이며, 업데이트 과정의 여러 단계에서 발생하는 실패가 장치 사용 불능 상태가 아니라 정의된 복구 상태(recovery state)로 이어지도록 한다.

전원 손실 복원력(power-loss resilience)을 확보하려면 메타데이터와 영구 구성(persistent configuration)도 신중하게 처리해야 한다. 이미지 업데이트와 부트 선택 상태 변경을 하나의 통제되지 않은 작업으로 취급해서는 안 된다. 중요한 메타데이터는 원자적(atomic) 또는 복구 가능한 방식으로 기록되어 예기치 않은 리셋이 발생하더라도 모호한 슬롯 선택 상태가 남지 않아야 한다. 설계는 부트로더가 어떤 이미지를 신뢰해야 하며 어떤 복구 동작을 수행해야 하는지 항상 판단할 수 있는 충분한 영구 정보를 제공해야 한다.

모든 형태의 영구 데이터(persistent data)를 시스템 파티션과 함께 복제할 필요는 없다. 장치 식별정보(device identity), 보정 매개변수(calibration parameter), 보안 자격증명(security credential), 로봇 구성(robot configuration), 운용 로그(operational log), 사용자 생성 데이터(user-generated data)는 슬롯 A와 슬롯 B 사이를 전환해도 유지되는 별도의 영구 파티션(persistent partition)에 저장해야 할 수 있다. 자동 롤백 이후에도 이전 버전이 데이터를 사용할 수 있도록 전환 과정에서 기존 버전과 후보 버전 모두가 공유 데이터의 스키마(schema)를 처리할 수 있어야 한다.

파티션 크기(partition sizing)는 현재 이미지 크기만이 아니라 전체 업데이트 수명주기(update lifecycle)를 고려해야 한다. 두 슬롯 모두 향후 운영체제와 애플리케이션 크기 증가를 수용할 수 있는 충분한 공간이 필요하며, 다운로드 패키지(downloaded package), 임시 파일(temporary file), 로그(log), 복구 정보(recovery information)를 위한 추가 저장공간도 필요할 수 있다. 따라서 플래시 용량이 제한된 임베디드 플랫폼에서는 완전한 A/B 이미지가 제공하는 강력한 복구 능력과 두 개의 부팅 가능한 환경을 유지하기 위한 저장공간 오버헤드(storage overhead) 사이의 절충이 필요하다.

보안 제어(security control)는 전체 슬롯 전환 과정에서 계속 적용되어야 한다. 부트로더는 단순히 이미지가 선택된 파티션에 존재한다는 이유만으로 해당 이미지를 부팅해서는 안 된다. 후보 이미지는 플랫폼의 보안 부팅(secure boot) 및 서명 검증(signature verification) 요구사항을 충족해야 하며, 롤백 방지(rollback protection)는 취약점이 존재하는 과거 버전의 무단 설치를 방지할 수 있다. 따라서 부트 선택 메커니즘, 버전 메타데이터, 서명 키(signing key), 복구 로직(recovery logic)은 펌웨어 이미지 자체와 동일한 신뢰 체인(chain of trust)에 속한다.

로봇 시스템(robotic system)에서는 파티션 전환을 물리적 운용 상태(physical operating state)와도 조정해야 한다. 로봇이 이동 중이거나 물체를 조작하거나 안전 필수 장비(safety-critical equipment)를 제어하거나 배터리 용량이 부족한 경우 활성화를 금지할 수 있다. 이미지는 사전에 다운로드하고 준비할 수 있지만 재부팅과 슬롯 전환은 임무 완료(mission completion), 도킹(docking), 충전(charging), 명시적으로 승인된 서비스 시간(service window)과 같은 안전한 유지보수 조건에 도달할 때까지 연기할 수 있다.

결과적으로 A/B 파티셔닝은 임베디드 소프트웨어 진화(embedded software evolution)를 위한 트랜잭션 모델(transactional model)을 제공한다. 기존 이미지는 커밋된 상태(committed state)를 나타내고 비활성 파티션은 후보 상태(candidate state)를 저장하며, 시험 부팅(trial boot)은 통제된 실행을 수행하고 상태 검증(health validation)은 승인 여부를 판단하며 확인 과정은 새로운 상태를 최종 커밋한다. 중요한 단계에서 문제가 발생하면 롤백이 이전 구성을 복원한다. 이러한 아키텍처는 단순히 소프트웨어를 빠르게 설치하는 것보다 복구 가능성(recoverability)이 중요한 고신뢰성 OTA 배포의 기반을 제공한다.

## 10.03 Delta Update Algorithms: bsdiff / xdelta [w/Code]

![](images/image3.png){width="7.268055555555556in" height="7.268055555555556in"}

델타 업데이트(delta update)는 설치된 소프트웨어 이미지(installed software image)를 새로운 버전으로 변환하는 데 필요한 정보만 배포함으로써 OTA 전송 크기를 줄이는 방식이다. 모든 릴리스마다 전체 펌웨어 또는 파일시스템 이미지(filesystem image)를 다운로드하는 대신, 업데이트 시스템은 이전 버전과 새로운 버전을 비교하여 작은 델타 패키지(delta package)를 생성한다. 대상 장치에서는 이 패키지를 검증된 기준 이미지(base image)에 적용하여 활성화 전에 정확한 새로운 소프트웨어 이미지를 재구성한다.

기본적인 델타 업데이트 작업 흐름(delta-update workflow)은 계산 특성이 서로 다른 두 단계로 구성된다. 릴리스 준비 과정에서 서버 측 도구(server-side tool)는 이전 이미지와 새로운 이미지를 입력받아 차이를 계산한다. 생성된 패치(patch)는 완전한 대상 이미지 대신 변경 명령과 변경 데이터를 포함한다. 설치 과정에서는 임베디드 장치가 기존 이미지와 다운로드된 패치를 결합하여 후보 이미지(candidate image)를 재구성하고, 이후 일반적인 OTA 검증 및 활성화 과정으로 전달한다.

델타 업데이트는 통신 대역폭(communication bandwidth), 다운로드 시간(download time), 데이터 비용(data cost)이 제한된 환경에서 특히 유용하다. 작은 펌웨어 수정은 논리적으로 프로그램의 제한된 부분만 변경할 수 있지만 전체 이미지 업데이트(full-image update)는 수백 메가바이트 또는 수 기가바이트를 전송해야 할 수 있다. 효율적인 차분 알고리즘(differencing algorithm)이 이전 릴리스에서 재사용 가능한 콘텐츠를 찾아내면 전송 산출물의 크기를 크게 줄여 네트워크 사용량과 OTA 패키지 획득 시간을 감소시킬 수 있다.

델타 생성(delta generation)의 효율성은 빌드 사이에서 소프트웨어 이미지가 어떻게 변경되는지에 크게 좌우된다. 작은 소스 코드 변경이라도 컴파일러 출력(compiler output), 링크 주소(link address), 압축 파일시스템(compressed filesystem), 타임스탬프(timestamp), 메타데이터(metadata), 블록 재배치(reordered block) 등에 의해 바이너리 이미지 전체로 변경이 확산될 수 있다. 따라서 OTA 엔지니어링에서는 작은 소스 변경이 반드시 작은 델타 패키지를 만든다고 가정하지 말고 실제 생성된 패치 크기를 평가해야 한다.

bsdiff는 이전 바이너리(old binary)와 새로운 바이너리(new binary) 사이의 관계를 찾아 작은 패치를 생성하도록 설계된 바이너리 차분 방식(binary differencing approach)이다. 개념적으로 일치하는 영역과 서로 다른 영역을 분석한 다음, 이전 파일의 바이트를 어떻게 재사용하거나 수정하여 새로운 파일을 생성할 것인지를 인코딩(encoding)한다. 생성된 패치는 변환 정보(transformation information)와 추가 데이터를 분리하여 기존 이미지의 상당 부분을 다시 전송하지 않고 재구성 과정에 활용할 수 있도록 한다.

일반적인 bsdiff 작업 흐름은 개념적으로 \`bsdiff old.bin new.bin update.patch\`로 표현할 수 있으며, 대상 장치에서는 \`bspatch old.bin reconstructed.bin update.patch\`를 수행한다. 첫 번째 작업은 일반적으로 상대적으로 풍부한 계산 자원을 사용할 수 있는 빌드 환경(build environment) 또는 OTA 백엔드(OTA backend)에서 수행된다. 두 번째 작업은 임베디드 플랫폼에서 대상 이미지를 재구성한다. 실제 제품에서는 패치 적용을 독립적인 명령으로 처리하기보다 버전 검사, 서명 검증, 저장장치 보호, 실패 처리와 함께 통합해야 한다.

bsdiff는 높은 유사성을 가진 바이너리에 대해 매우 작은 패치 크기를 제공할 수 있지만, 자원이 제한된 임베디드 장치를 대상으로 할 경우 자원 요구사항(resource requirement)을 고려해야 한다. 패치 생성에는 상당한 연산과 메모리가 필요할 수 있으며 재구성 과정에서도 기준 데이터, 패치, 결과 이미지를 위한 충분한 작업공간과 저장공간이 필요하다. 따라서 델타 효율성은 네트워크 절감량만으로 평가해서는 안 되며 RAM 사용량, 플래시 용량, 실행 시간, 전원 손실 동작, 대상 하드웨어 성능을 함께 고려해야 한다.

xdelta는 또 다른 실용적인 바이너리 델타 기법(binary delta technique)의 계열이며 일반적으로 VCDIFF 방식의 인코딩(VCDIFF-style encoding)과 관련된다. 전체 대상 소프트웨어를 전송하는 대신 소스 데이터의 어떤 부분을 복사하고 어떤 새로운 데이터를 삽입해야 하는지를 설명하는 명령을 인코딩한다. 이러한 복사 및 추가 모델(copy-and-add model)은 일반적인 바이너리 콘텐츠에 적합하며, 관련된 소프트웨어 산출물 사이의 차이만 배포하면서 대상 버전을 정확하게 재구성하기 위한 기반을 제공한다.

개념적으로 xdelta 작업 흐름은 기존 릴리스를 소스 사전(source dictionary)으로 사용하여 새로운 릴리스에 대한 델타를 생성한다. 이후 장치는 정확히 예상된 소스 버전을 사용하여 해당 델타를 디코딩(decoding)한다. 이러한 종속성은 매우 중요하다. 버전 A에서 버전 B로 변환하도록 생성된 패치를 일반적으로 버전 C에 적용해서는 안 된다. 따라서 OTA 메타데이터는 재구성을 시작하기 전에 필요한 기준 버전(base version), 대상 버전(target version), 패치 형식(patch format), 알고리즘 버전(algorithm version), 무결성 정보(integrity information)를 식별해야 한다.

bsdiff, xdelta, 전체 이미지 업데이트 중 어떤 방식을 선택할지는 패치 크기만이 아니라 시스템 제약조건(system constraint)을 기준으로 결정해야 한다. 바이너리 차이 크기의 최소화가 중요하다면 bsdiff가 유리할 수 있으며, xdelta 계열 방식은 구현 방법에 따라 실용적인 스트리밍 중심 동작(streaming-oriented behavior)과 통제된 자원 사용을 제공할 수 있다. 버전 간 차이가 매우 크거나 재구성 자원이 제한적이거나 여러 버전별 델타 경로를 관리하는 것이 비효율적이라면 전체 이미지가 더 적합할 수 있다.

따라서 실제 OTA 백엔드(production OTA backend)는 하나의 릴리스에 대해 여러 업데이트 산출물을 생성할 수 있다. 예를 들어 바로 이전 버전을 실행하는 장치에는 작은 델타를 제공하고, 더 오래된 버전에는 별도의 호환 델타를 제공하거나 전체 이미지로 폴백(fallback)할 수 있다. 배포 서비스(deployment service)는 장치가 보고한 상태에 따라 적절한 산출물을 선택한다. 이를 위해 소프트웨어 버전을 노드(node)로, 유효한 델타 변환을 지원되는 소스 버전과 대상 버전을 연결하는 방향성 에지(directed edge)로 표현하는 명시적인 업데이트 그래프(update graph)가 필요하다.

대상 장치는 델타를 적용하기 전에 기준 이미지(base image)를 검증해야 한다. 설치된 소스가 패치 생성 시 가정한 정확한 이미지와 다르면 재구성이 실패하거나 잘못된 결과를 생성할 수 있다. 업데이트 프로그램은 신뢰할 수 있는 버전 메타데이터를 비교하고 가능하면 소스 이미지의 암호학적 다이제스트(cryptographic digest)도 확인해야 한다. 재구성이 완료된 후에는 전체 후보 이미지를 다시 해싱(hashing)하고 예상 대상 다이제스트와 비교하여 빌드 시스템에서 생성한 릴리스와 정확히 동일한 이미지가 만들어졌는지 확인해야 한다.

암호학적 서명(cryptographic signing)은 전체 OTA 산출물과 동일한 방식으로 델타 패키지와 메타데이터를 보호해야 한다. 작은 패치도 실행 가능한 소프트웨어 변환 데이터(executable software transformation data)이므로 소프트웨어 공급망 신뢰 경계(software supply-chain trust boundary)에 포함된다. 패치 적용 전에는 인증(authentication)이 이루어져야 하며, 재구성 후에는 최종 이미지 무결성 검증(final-image integrity verification)을 수행해야 한다. 이후 보안 부팅(secure boot)이 시작 과정에서 후보 이미지를 독립적으로 검증함으로써 다운로드, 재구성, 저장, 실행에 걸쳐 신뢰성을 유지할 수 있다.

델타 재구성(delta reconstruction)은 A/B 파티션 아키텍처(A/B partition architecture)와 자연스럽게 통합될 수 있다. 로봇은 활성 슬롯(active slot)에서 계속 동작하는 동안 업데이트 프로그램이 필요한 기준 콘텐츠를 읽고 비활성 슬롯(inactive slot)에 새로운 이미지를 생성할 수 있다. 변환 과정에서도 기존의 정상 확인 이미지(known-good image)는 보호된다. 재구성과 무결성 검증이 성공하면 비활성 슬롯이 부팅 후보가 되고 안전한 A/B 업데이트에서 사용하는 시험 부팅(trial boot), 상태 검증(health validation), 확인(confirmation), 롤백(rollback) 과정을 따른다.

패치 적용 과정에서는 전원 중단(power interruption)이 발생할 수 있다고 가정해야 한다. 업데이트 프로그램은 부분적으로 적용된 변환 결과에 유일한 부팅 가능 이미지가 의존하도록 만들어서는 안 된다. 임시 출력(temporary output), 트랜잭션 메타데이터(transactional metadata), 필요한 경우 체크포인트(checkpoint), 비활성 슬롯 재구성을 활용하면 중단된 패치 작업으로 정상 확인 상태가 손상되는 것을 방지할 수 있다. 재시작 후 업데이트 에이전트는 재구성을 안전하게 재개할지, 처음부터 다시 시작할지, 불완전한 결과를 폐기하고 기존 릴리스로 계속 운용할지를 판단할 수 있어야 한다.

델타 업데이트는 지원되는 각 소스 버전마다 서로 다른 패치가 필요할 수 있기 때문에 수명주기 복잡성(lifecycle complexity)도 증가시킨다. 모든 과거 릴리스에서 현재 릴리스로 연결되는 델타를 유지하면 저장공간, 테스트, 배포 로직의 비용이 빠르게 증가한다. 실제 시스템에서는 일반적으로 최근 몇 개 버전에 대해서만 델타를 지원하고 오래되었거나 예상하지 못한 설치 버전에는 전체 이미지 폴백(full-image fallback)을 사용한다. 이를 통해 정기적으로 유지관리되는 대부분의 장치에서 대역폭 절감 효과를 유지하면서 업데이트 그래프의 복잡성을 관리할 수 있다.

로봇 플릿(robotic fleet)에서는 델타 알고리즘을 단순한 압축 기술로 취급하기보다 릴리스 오케스트레이션(release orchestration)과 결합하는 것이 가장 강력한 설계가 된다. 백엔드는 올바른 소스-대상 경로(source-to-target path)를 결정하고, 로봇은 현재 상태를 검증하며, 델타는 정확한 후보 이미지를 재구성하고, A/B 보호 구조는 복구 가능성(recoverability)을 유지한다. 따라서 bsdiff와 xdelta는 OTA의 전송 단계를 최적화하며, 서명, 무결성 검증, 파티션 안전성, 롤백, 플릿 정책(fleet policy)이 신뢰할 수 있는 임베디드 배포를 위한 전체 아키텍처를 완성한다.

## 10.04 Firmware Signing and Integrity Verification [w/Code]

![](images/image4.png){width="7.268055555555556in" height="7.268055555555556in"}

펌웨어 서명(firmware signing)은 임베디드 소프트웨어(embedded software)를 생산하는 조직과 해당 소프트웨어를 실행하는 장치 사이에 암호학적 신뢰 관계(cryptographic trust relationship)를 설정한다. OTA 시스템은 펌웨어 이미지(firmware image)가 예상된 서버에서 전달되었다는 이유만으로 이를 신뢰해서는 안 된다. 장치는 해당 소프트웨어가 신뢰할 수 있는 릴리스 권한 기관(trusted release authority)에 의해 승인되었으며 승인된 패키지가 생성된 이후 내용이 변경되지 않았는지를 독립적으로 검증해야 한다.

무결성 검증(integrity verification)과 진위성 검증(authenticity verification)은 서로 다른 보안 문제를 다룬다. SHA-256과 같은 암호학적 해시(cryptographic hash)는 펌웨어 바이트가 변경되었는지를 탐지할 수 있지만, 공격자가 펌웨어를 교체하면서 보호되지 않은 해시까지 변경할 수도 있다. 디지털 서명(digital signature)은 예상 다이제스트(digest)를 신뢰할 수 있는 개인 서명 키(private signing key)에 결합한다. 따라서 대응하는 공개 키(public key)를 이용한 검증을 통해 이미지의 무결성과 승인된 서명 과정에서 생성되었는지를 함께 판단할 수 있다.

서명 과정(signing process)은 일반적으로 펌웨어 이미지가 필요한 빌드(build), 통합(integration), 릴리스 테스트(release test)를 통과한 이후 수행된다. 정의된 펌웨어 내용에 대해 암호학적 다이제스트(cryptographic digest)를 계산하고, 해당 다이제스트 또는 이를 포함하는 구조화된 표현을 보호된 개인 키(private key)로 서명한다. 생성된 서명과 버전 정보, 하드웨어 호환성 데이터(hardware compatibility data), 이미지 길이, 알고리즘 식별자(algorithm identifier), 기타 릴리스 속성은 업데이트 산출물과 함께 인증된 펌웨어 메타데이터(authenticated firmware metadata)에 저장할 수 있다.

공개 키 암호화(public-key cryptography)는 서명 권한(signing authority)을 배포된 장치와 분리할 수 있게 한다. 운영 펌웨어(production firmware)를 승인하는 개인 키는 통제된 서명 인프라(controlled signing infrastructure) 내부에 보관되어야 하며 로봇에 배포해서는 안 된다. 장치에는 검증에 필요한 대응 공개 키 또는 신뢰할 수 있는 인증서 체인(certificate chain)만 필요하다. 따라서 하나의 로봇이 침해되더라도 공격자가 다른 로봇에서 정상 펌웨어로 인정되는 이미지를 생성할 수 있는 능력을 자동으로 획득하지는 않는다.

RSA 및 타원곡선 디지털 서명 방식(elliptic-curve digital signature scheme)과 같은 알고리즘을 임베디드 펌웨어 서명에 사용할 수 있으며, 구체적인 선택은 플랫폼 요구사항, 암호화 라이브러리(cryptographic library), 하드웨어 가속(hardware acceleration), 인증 제약조건(certification constraint), 수명주기 요구사항(lifecycle expectation)에 따라 결정된다. 보안 아키텍처는 지원되는 알고리즘, 키 크기(key size), 서명 형식(signature format), 해시 알고리즘을 명확하게 정의해야 하며 장기간 유지보수가 필요한 장치의 수명도 고려해야 한다.

개인 서명 키(private signing key)는 OTA 공급망(OTA supply chain)에서 가장 보안에 민감한 자산 중 하나이다. 이를 개발자 워크스테이션(developer workstation)이나 CI 서버의 일반 파일로 저장하면 불필요한 침해 경로가 만들어진다. 운영 시스템에서는 엄격하게 통제된 인프라를 사용하여 서명 작업을 격리하고 필요한 경우 하드웨어 보안 모듈(HSM, Hardware Security Module)을 사용해야 한다. 접근 제어(access control), 감사 로깅(audit logging), 업무 분리(separation of duties), 키 백업, 교체(rotation), 폐기(revocation), 사고 대응 절차(incident-response procedure)도 키 수명주기 관리(key lifecycle management)에 포함되어야 한다.

펌웨어 검증(firmware verification)은 설치 전에 시작된다. OTA 에이전트(OTA agent)가 패키지를 수신하면 패키지 구조와 인증된 메타데이터를 검증하고 예상되는 서명 권한을 확인한 다음 필요한 암호학적 다이제스트를 계산하여 디지털 서명을 검증해야 한다. 검사 중 하나라도 실패하면 부팅 가능한 소프트웨어 상태를 변경하기 전에 업데이트를 거부해야 한다. 따라서 다운로드 완료는 단순히 산출물을 확보했다는 의미이며 해당 산출물을 실행하거나 활성화할 권한이 부여되었다는 의미는 아니다.

검증은 무결성을 보호해야 하는 정확한 콘텐츠 범위에 적용되어야 한다. 어떤 바이트가 서명 대상인지 모호하면 심각한 구현 취약점(implementation weakness)이 발생할 수 있다. 펌웨어 형식(firmware format), 헤더 필드(header field), 실행 영역(executable region), 구성 요소(configuration element), 버전 메타데이터, 선택적 구성요소(optional component)는 명확한 인증 경계(authentication boundary)를 가져야 한다. 서명 환경과 임베디드 검증기가 정확히 동일한 논리적 표현에 대해 서명을 계산할 수 있도록 정규화된 메타데이터 인코딩(canonical metadata encoding)을 사용하는 것이 유용하다.

펌웨어가 기록되거나 재구성된 이후에는 두 번째 무결성 검사(integrity check)가 중요하다. 플래시 프로그래밍 오류(flash programming error), 중단된 쓰기 작업, 저장장치 손상(storage corruption), 델타 재구성(delta reconstruction) 과정의 결함으로 인해 다운로드 패키지가 정상이어도 저장된 이미지가 서명된 대상과 달라질 수 있다. 활성화 전에 최종 이미지 다이제스트(final image digest)를 계산하고 비교하면 대상 파티션에 저장된 후보 이미지가 배포 시스템이 기대하는 정확한 소프트웨어 릴리스인지 확인할 수 있다.

델타 업데이트(delta update)는 다운로드된 패치와 최종 실행 이미지가 서로 다른 산출물이므로 특히 세심한 무결성 처리가 필요하다. 패치는 적용 전에 인증되어야 하고 설치된 기준 이미지(base image)는 요구되는 소스 버전(source version)과 일치해야 하며 재구성된 이미지는 예상되는 대상 다이제스트(target digest)와 일치해야 한다. 이를 통해 패치 실행 성공만으로 펌웨어가 정상이라고 가정하는 대신 신뢰할 수 있는 기준 이미지에서 인증된 변환을 거쳐 정확한 대상 이미지에 이르는 검증 체인(verification chain)을 구성한다.

A/B 파티션 아키텍처(A/B partition architecture)는 보안 검증을 수행하기 위한 자연스러운 구조를 제공한다. 활성 슬롯(active slot)은 정상 확인 시스템(known-good system)으로 계속 실행되고 인증된 업데이트는 비활성 슬롯(inactive slot)에 기록된다. 해당 슬롯이 부팅 후보(boot candidate)가 되기 전에 서명 및 무결성 검사가 완료된다. 검증이 실패하면 부트 구성(boot configuration)은 변경되지 않는다. 이를 통해 현재 신뢰되는 소프트웨어를 보호하면서 유효하지 않은 후보 이미지가 활성화 경계(activation boundary)를 넘어가는 것을 방지한다.

보안 부팅(secure boot)은 펌웨어 인증(firmware authentication)을 시스템 시작 과정까지 확장한다. OTA 검증은 활성화 전에 업데이트를 확인하고 부트로더(bootloader)는 장치가 시작될 때 실행 소프트웨어를 독립적으로 다시 검증한다. 루트 공개 키(root public key) 또는 신뢰 키 자료(trusted key material)의 해시는 보호된 하드웨어, 변경 불가능한 부트 ROM(immutable boot ROM), 일회성 프로그래밍 메모리(OTP memory), 신뢰 플랫폼 모듈(TPM, Trusted Platform Module) 또는 다른 신뢰 루트(root-of-trust) 메커니즘에 고정할 수 있다. 이를 통해 공격자가 OTA 에이전트를 우회하여 승인되지 않은 실행 콘텐츠를 직접 설치하는 것을 방지한다.

버전 제어(version control) 역시 펌웨어 인증 과정에 참여해야 한다. 올바르게 서명된 이미지라도 오래된 취약점(vulnerability)을 포함하고 있다면 안전하지 않을 수 있기 때문이다. 롤백 방지(anti-rollback protection)는 신뢰할 수 있는 펌웨어를 단조 증가하는 보안 버전(monotonic security version) 또는 이에 상응하는 정책과 연결한다. 부트로더 또는 보안 업데이트 메커니즘은 서명이 수학적으로 유효하더라도 허용된 보안 수준보다 낮은 이미지를 거부하여 정상적인 과거 펌웨어가 다운그레이드 공격(downgrade attack)에 악용되는 것을 방지한다.

키 교체(key rotation)는 장기간 운용되는 임베디드 제품에 필수적이다. 서명 키는 만료되거나 정책에 따라 교체될 수 있으며 침해가 의심되는 경우 긴급하게 폐기해야 할 수도 있다. 따라서 장치는 임의적인 신뢰 변경을 허용하지 않으면서 신뢰할 수 있는 검증 키(trusted verification key)를 갱신할 수 있는 통제된 메커니즘을 가져야 한다. 루트 키(root key)가 중간 릴리스 키(intermediate release key)를 승인하도록 구성하면 강하게 보호된 신뢰 루트는 장치 수명주기 동안 안정적으로 유지하면서 운영 키를 교체할 수 있다.

인증서 기반 아키텍처(certificate-based architecture)는 펌웨어 서명을 신뢰 체인(chain of trust)을 통해 검증함으로써 이러한 계층 구조를 확장할 수 있다. 장치는 제조사 루트(manufacturer root)를 신뢰하고 개별 제품, 환경 또는 릴리스 조직은 하위 인증서(subordinate certificate)를 사용할 수 있다. 이러한 분리는 하나의 운영 키가 침해되었을 때의 영향을 줄이고 개발, 시험, 운영 환경에 서로 다른 권한을 부여할 수 있게 한다. 그러나 신뢰 정책(trust policy)은 자원이 제한된 임베디드 소프트웨어에서도 안정적으로 구현할 수 있을 정도로 단순하게 유지해야 한다.

실패 동작(failure behavior)은 항상 정상 확인 상태(known-good state)를 보존하는 방향으로 설계해야 한다. 유효하지 않은 서명, 일치하지 않는 해시, 지원되지 않는 알고리즘, 잘못 구성된 메타데이터, 만료된 자격증명(expired credential), 호환되지 않는 하드웨어 식별자 또는 금지된 버전이 발견되면 활성화를 중지하고 진단 정보(diagnostic evidence)를 생성해야 한다. 로봇은 현재의 신뢰할 수 있는 이미지를 사용하여 계속 동작하면서 OTA 관리 시스템에 실패를 보고할 수 있으며, 신뢰할 수 없는 업데이트를 거부했다는 이유로 불필요한 서비스 중단이 발생하는 것을 방지할 수 있다.

로봇 플릿(robotic fleet)에서 서명과 무결성 검증은 궁극적으로 소프트웨어 전달과 물리적 실행(physical execution) 사이의 보안 게이트(security gate)를 형성한다. 릴리스 파이프라인(release pipeline)은 어떤 소프트웨어가 승인되었는지를 결정하고 암호학적 서명은 신뢰할 수 없는 네트워크와 저장장치를 거치는 동안 그 승인을 보존한다. 임베디드 검증은 설치 전에 이를 확인하고 보안 부팅은 시작 시 다시 강제한다. A/B 파티션, 롤백 제어, 보호된 키, 감사 가능한 릴리스 프로세스와 결합하면 펌웨어 생성부터 로봇 운용까지 이어지는 연속적인 신뢰 체인을 구축할 수 있다.

## 10.05 RAUC Framework Configuration and Bundle Creation [w/Code]

![](images/image5.png){width="7.268055555555556in" height="7.268055555555556in"}

RAUC는 복구 가능한 시스템 상태(recoverable system state)를 유지하면서 인증된 소프트웨어 번들(authenticated software bundle)을 설치하도록 설계된 임베디드 리눅스 업데이트 프레임워크(embedded Linux update framework)이다. OTA 업데이트를 단순한 파일 교체로 처리하는 대신, RAUC는 장치를 관리되는 설치 슬롯(managed installation slot)의 집합으로 모델링하고 미리 정의된 시스템 구성(system configuration)에 따라 서명된 번들(signed bundle)을 적용한다. 따라서 통제된 A/B 업데이트, 부트로더 연동, 결정론적 복구 동작(deterministic recovery behavior)이 필요한 로봇과 산업용 장치에 특히 적합하다.

일반적인 RAUC 배포(deployment)는 현재 실행 중인 루트 파일시스템(root filesystem)과 비활성 업데이트 대상(inactive update target)을 분리한다. 일반적으로 rootfs.0과 rootfs.1로 표현되는 두 개의 루트 파일시스템 슬롯은 연속적인 릴리스 과정에서 활성(active)과 비활성(inactive) 역할을 번갈아 수행한다. 추가 슬롯은 부트 파티션(boot partition), 애플리케이션 데이터(application data), 기타 업데이트 가능한 구성요소를 나타낼 수 있다. RAUC는 시스템 구성을 통해 이러한 관계를 이해하고 번들 이미지를 설치할 위치를 결정한다.

RAUC의 핵심 구성은 일반적으로 \`system.conf\`에 정의된다. 이 파일은 호환 장치 식별정보(compatible device identity), 부트로더 연동(bootloader integration), 키링 위치(keyring location), 사용 가능한 슬롯을 기술한다. 각 슬롯 정의(slot definition)는 장치 경로(device path), 슬롯 유형(slot type), 부트 이름(boot name), 필요한 경우 상위 관계(parent relationship)와 같은 속성을 식별한다. 따라서 이 구성은 임베디드 대상의 물리적 저장장치 구조를 RAUC가 설치 중 안전하게 관리할 수 있는 논리적 업데이트 모델(logical update model)로 변환한다.

\`compatible\` 값은 업데이트 번들과 대상 하드웨어 사이에 중요한 호환성 경계(compatibility boundary)를 제공한다. 번들은 자신이 제작된 대상 시스템의 호환성을 선언하고, 대상 장치 구성 역시 자체 호환 식별정보(compatible identity)를 선언한다. RAUC는 설치 전에 이 값들을 비교한다. 이를 통해 올바르게 서명되었더라도 다른 보드, 제품 세대(product generation), 로봇 컨트롤러용으로 제작되어 기술적으로 호환되지 않는 번들이 실수로 설치되는 것을 방지한다.

부트로더 연동은 RAUC가 소프트웨어 설치와 실제 부팅 결정(boot decision)을 연결할 수 있게 한다. 플랫폼에 따라 RAUC는 지원되는 부팅 메커니즘과 연동하여 새로 기록된 슬롯이 다음 부팅 후보(boot candidate)가 되도록 할 수 있다. 실제 시작 전환(startup transition)은 여전히 부트로더가 수행하며, RAUC는 슬롯 상태(slot state)와 업데이트 결과에 관한 정보를 관리한다. 이러한 역할 분리를 통해 설치 로직과 저수준 부트 제어(low-level boot control)가 공통의 A/B 복구 전략에 참여할 수 있다.

슬롯 정의는 대상 장치의 실제 파티션 구조(partition layout)와 정확하게 일치해야 한다. 예를 들어 rootfs.0은 하나의 블록 장치 파티션(block device partition)을 참조하고 rootfs.1은 동일하거나 충분한 용량을 가진 다른 파티션을 참조할 수 있다. 잘못된 장치 매핑(device mapping)은 엉뚱한 저장 영역을 덮어써 A/B 업데이트의 안전 특성을 무력화할 수 있다. 따라서 파티션 테이블(partition table), 부트로더 구성, 파일시스템 생성, RAUC 구성은 하나의 통합된 저장장치 아키텍처(storage architecture)로 설계하고 검증해야 한다.

RAUC는 번들(bundle)을 사용하여 업데이트 콘텐츠를 배포한다. 번들은 하나 이상의 소프트웨어 이미지와 매니페스트(manifest), 암호학적 인증 정보(cryptographic authentication information)를 포함하는 구조화된 업데이트 산출물(structured update artifact)이다. 임의의 루트 파일시스템 이미지와 메타데이터를 각각 독립적으로 전송하는 대신 릴리스 파이프라인(release pipeline)은 이를 RAUC가 이해할 수 있는 하나의 단위로 패키징한다. 이 번들은 소프트웨어 생산 인프라와 임베디드 대상에서 실행되는 업데이트 메커니즘 사이의 통제된 인터페이스가 된다.

번들 매니페스트(bundle manifest)는 패키지가 무엇을 포함하고 있으며 해당 이미지가 논리적으로 어디에 속하는지를 기술한다. 번들 호환성(bundle compatibility), 버전 정보(version information), 설명 메타데이터(descriptive metadata), 구성된 슬롯 클래스(slot class)에 대응하는 이미지 항목(image entry)을 식별할 수 있다. 예를 들어 루트 파일시스템용 이미지는 특정 활성 파티션에 직접 고정되는 대신 적절한 rootfs 슬롯 클래스와 연결된다. 이후 RAUC가 현재 장치의 슬롯 상태를 기준으로 올바른 비활성 설치 대상을 결정한다.

번들 생성(bundle creation)은 임베디드 리눅스 빌드 과정에서 생성된 재현 가능한 소프트웨어 산출물(reproducible software artifact)로 시작한다. 루트 파일시스템 이미지는 Yocto, Buildroot 또는 다른 통제된 이미지 생성 파이프라인(image-generation pipeline)에서 만들어질 수 있다. 릴리스 과정에서는 필요한 이미지와 매니페스트를 번들 입력 디렉터리(bundle input directory)에 배치한 후 RAUC 번들 생성 메커니즘을 실행한다. 생성된 \`.raucb\` 산출물은 호환 장치에 인증된 방식으로 배포할 수 있는 형태로 릴리스를 패키징한다.

단순화된 릴리스 환경에서는 매니페스트, 파일시스템 이미지, 인증서(certificate), 서명 키(signing key) 구성이 준비된 후 개념적으로 \`rauc bundle bundle-input update.raucb\`를 실행할 수 있다. 실제 운영 환경에서는 서명 키를 일반적인 개발 자산(development asset)으로 취급해서는 안 된다. 번들 생성은 서명 권한(signing authority), 접근 권한(access permission), 감사 기록(audit record), 키 관리 절차(key-management procedure)가 명확하게 통제되는 보호된 릴리스 파이프라인에 통합되어야 한다.

RAUC는 암호학적 서명(cryptographic signing)을 사용하여 번들이 승인된 릴리스 권한 기관(accepted release authority)에서 생성되었으며 서명 이후 변경되지 않았음을 보장한다. 대상 장치는 키링(keyring)을 통해 신뢰할 수 있는 검증 자료(trusted verification material)를 구성하고, 개인 서명 키(private signing key)는 릴리스 환경에 유지한다. 설치가 요청되면 RAUC는 관리되는 슬롯에 내용을 기록하기 전에 번들을 검증한다. 따라서 신뢰할 수 없거나 손상된 번들은 설치 경계(installation boundary)를 통과하기 전에 거부된다.

설치는 개념적으로 \`rauc install update.raucb\`와 같은 명령으로 시작할 수 있다. RAUC는 먼저 번들을 검사하고 호환성과 인증(authentication)을 확인한 후 현재 부팅된 슬롯을 식별하고 적절한 비활성 대상을 선택하여 해당 이미지를 설치한다. 이 과정에서 활성 루트 파일시스템(active root filesystem)은 보호된 상태로 유지된다. 그러나 기록에 성공했다는 사실만으로 새로운 소프트웨어가 영구적으로 승인된 것은 아니며, 활성화(activation)와 부팅 후 확인(post-boot confirmation)은 별도의 수명주기 단계로 남는다.

슬롯 상태(slot status)는 장치의 업데이트 상태를 이해하는 데 필요한 운용 관점(operational view)을 제공한다. \`rauc status\`와 같은 명령을 통해 구성된 슬롯, 현재 부팅된 시스템, 관련 슬롯 상태 정보를 확인할 수 있다. 이러한 정보는 로컬 진단(local diagnostics)뿐 아니라 원격 OTA 관리에도 중요하다. 플릿 백엔드(fleet backend)는 단순히 업데이트를 다운로드한 장치와 실제로 설치, 부팅, 확인, 실패 또는 롤백한 장치를 구분할 수 있어야 한다.

설치 이후 부트로더는 새롭게 선택된 슬롯을 후보 시스템(candidate system)으로 시작할 수 있다. 이후 업데이트된 소프트웨어는 신뢰할 수 있는 상태로 판단되기 전에 애플리케이션별 상태 검사(application-specific health check)를 수행해야 한다. 로봇에서는 미들웨어 시작, 네트워크 연결, 모터 컨트롤러 통신, 안전 인터페이스(safety interface), 센서 가용성(sensor availability), 필수 서비스 등을 검증할 수 있다. 이러한 검사가 성공하면 플랫폼에서 사용하는 부트 및 RAUC 연동 전략에 따라 후보 시스템을 정상 상태(good)로 표시할 수 있다.

후보 시스템이 반복적으로 부팅에 실패하거나 필요한 상태 기준(health criteria)을 만족하지 못하면 시스템은 이전의 정상 확인 슬롯(known-good slot)을 보존하거나 해당 슬롯으로 복귀해야 한다. RAUC 자체가 신중하게 설계된 부트로더 복구 의미론(bootloader recovery semantics)의 필요성을 제거하는 것은 아니다. 대신 A/B 파티션, 부팅 시도 제어(boot-attempt control), 워치독 감시(watchdog supervision), 애플리케이션 상태 검증, 롤백 정책을 포함하는 더 넓은 아키텍처에 참여한다. 따라서 신뢰할 수 있는 OTA 동작은 업데이트 프레임워크만이 아니라 전체 플랫폼 통합에 의존한다.

로봇 제품에서 RAUC는 단순히 장치 측 유틸리티(device-side utility)로 추가하기보다 빌드 및 배포 수명주기(build and deployment lifecycle)에 통합할 때 가장 효과적이다. 빌드 시스템은 결정론적 이미지(deterministic image)를 생성하고, 릴리스 파이프라인은 번들을 생성하고 서명하며, OTA 서비스는 이를 배포한다. RAUC는 번들을 검증하고 통제된 슬롯에 설치하며 부트로더가 실제 전환을 수행한다. 이후 런타임 검증(runtime validation)을 통해 새로운 소프트웨어를 최종 확정할지 이전 시스템을 복원할지를 결정한다.

결과적으로 잘 설계된 RAUC 구현은 저장장치 아키텍처, 릴리스 패키징(release packaging), 암호학적 신뢰(cryptographic trust), 부트 제어(boot control), 운용 복구(operational recovery)를 하나의 업데이트 작업 흐름으로 연결한다. 번들 모델(bundle model)은 반복 가능하고 일관된 배포 단위(deployment unit)를 제공하며 슬롯 관리는 실행 중인 시스템이 직접 교체되는 것을 방지한다. A/B 파티셔닝, 보안 서명(secure signing), 호환성 검사, 상태 감시(health monitoring), 롤백 로직과 결합된 RAUC는 유지보수 가능하고 복구 가능한 임베디드 리눅스 OTA 업데이트를 위한 실용적인 기반을 제공한다.

## 10.06 Mender Platform Integration: Server/Client Setup [w/Code]

![](images/image6.png){width="7.268055555555556in" height="7.268055555555556in"}

Mender는 임베디드 리눅스 시스템(embedded Linux system) 전반에서 산출물 생성(artifact creation), 장치 관리(device management), 배포 제어(deployment control), 신뢰할 수 있는 설치(reliable installation)를 조정하도록 설계된 OTA 소프트웨어 업데이트 플랫폼(OTA software update platform)이다. 이 아키텍처는 서버 측 플릿 오케스트레이션(server-side fleet orchestration)과 클라이언트 측 업데이트 실행(client-side update execution)을 분리한다. 이를 통해 중앙 서비스가 여러 로봇의 소프트웨어 릴리스를 관리하면서 각 임베디드 장치는 인증, 다운로드, 설치, 재부팅, 업데이트 상태 보고를 독립적으로 수행할 수 있다.

서버 측(server side)은 OTA 수명주기(OTA lifecycle)의 관리 평면(management plane) 역할을 수행한다. 서버는 장치 식별정보(device identity), 소프트웨어 산출물(software artifact), 배포 정의(deployment definition), 인벤토리 정보(inventory information), 업데이트 상태(update status)를 관리한다. 운영자는 승인된 소프트웨어 릴리스를 업로드하고 이를 수신할 장치 또는 그룹을 선택한다. 서버는 배포를 조정하면서 클라이언트의 진행 상태를 수집하여 서로 다른 소프트웨어 버전과 업데이트 이력을 가진 다양한 로봇을 중앙에서 파악할 수 있도록 한다.

Mender 클라이언트(Mender client)는 관리되는 각 임베디드 리눅스 장치에서 실행되며 인증된 네트워크 연결(authenticated network connection)을 통해 서버와 통신한다. 클라이언트는 장치에 적용되는 배포가 있는지 주기적으로 확인하고 인벤토리와 소프트웨어 상태를 보고하며 할당된 산출물을 다운로드하고 해당 설치 작업 흐름을 실행한다. 일반적인 업데이트 통신은 장치에서 시작되므로 인바운드 관리 연결(inbound management connection)이 적합하지 않은 방화벽이나 네트워크 주소 변환(NAT, Network Address Translation) 환경의 로봇에도 효과적으로 적용할 수 있다.

장치 온보딩(device onboarding)은 새로운 클라이언트와 관리 서비스 사이의 신뢰 관계(trust relationship)를 설정한다. 클라이언트가 처음 서버에 연결하면 구성된 배포 모델에 따라 장치를 식별하고 승인할 수 있는 식별정보(identity information)를 제공한다. 승인 이후 서버는 해당 장치를 인벤토리 속성(inventory attribute)과 배포 그룹(deployment group)에 연결할 수 있다. 이를 통해 물리적 로봇과 해당 소프트웨어 상태 및 OTA 수명주기 기록을 연결하는 관리형 식별체계(managed identity)가 만들어진다.

Mender 아티팩트(Mender Artifact)는 플랫폼에서 사용하는 통제된 소프트웨어 전달 단위(controlled software delivery unit)이다. 임의의 파일시스템 이미지를 배포하는 대신 릴리스 파이프라인은 업데이트 콘텐츠를 아티팩트 식별정보(artifact identity), 호환 장치 유형(compatible device type), 페이로드 정보(payload information)를 설명하는 메타데이터와 함께 패키징한다. 아티팩트 형식을 통해 서버와 클라이언트는 설치 전에 특정 릴리스가 해당 장치 종류에 적합한지를 판단하여 잘못된 하드웨어에 기술적으로 호환되지 않는 소프트웨어가 배포될 위험을 줄인다.

아티팩트 생성(artifact creation)은 임베디드 리눅스 빌드 및 릴리스 파이프라인(build and release pipeline)과 통합되어야 한다. 루트 파일시스템 이미지(root filesystem image)는 통제된 빌드 환경에서 생성되고 릴리스 자동화(release automation)의 일부로 Mender 아티팩트에 패키징될 수 있다. 각 아티팩트에는 테스트된 빌드와 연결할 수 있는 의미 있는 소프트웨어 식별정보(software identity)가 포함되어야 한다. 생성된 파일은 서버에 업로드되어 개발, 검증 또는 운영 장치 그룹에 할당할 수 있는 변경되지 않는 릴리스 후보(immutable release candidate)가 된다.

클라이언트 구성(client configuration)은 임베디드 대상이 Mender 서비스에 접근하고 이를 신뢰하는 방법을 정의한다. 장치는 적절한 서버 엔드포인트(server endpoint), 인증 구성(authentication configuration), 영구 식별정보(persistent identity information), 대상 플랫폼에 필요한 업데이트 통합 구성을 가져야 한다. 네트워크 접근은 보안 전송 경로(secure transport path)를 통해 제공되어야 하며 신뢰할 수 있는 인증서와 장치 자격증명(device credential)을 보호해야 한다. 또한 운영 환경에서는 임시 개발 설정에 의존하지 않고 실제 운영 구성과 일관성을 유지해야 한다.

견고한 운영체제 업데이트를 위해 Mender는 A/B 루트 파일시스템 아키텍처(A/B root filesystem architecture)와 통합할 수 있다. 현재 활성화된 루트 파일시스템은 계속 실행되는 동안 클라이언트가 새로운 이미지를 비활성 파티션(inactive partition)에 설치한다. 따라서 정상적인 업데이트 설치 과정에서 활성 시스템(active system)을 직접 덮어쓰지 않는다. 이미지 기록이 성공적으로 완료되면 부트 구성(boot configuration)을 변경하여 다음 통제된 재부팅에서 비활성 파티션이 후보 시스템(candidate system)이 되도록 한다.

부트로더 연동(bootloader integration)은 A/B 트랜잭션(A/B transaction)을 완성하는 데 필수적이다. 클라이언트는 새로운 후보 이미지를 준비할 수 있지만 리셋 이후 어떤 루트 파일시스템이 실제로 시작되는지는 부트로더가 결정한다. 따라서 플랫폼은 파티션 정의(partition definition), 부트 변수(boot variable), 업데이트 상태를 일관되게 유지해야 한다. 저장장치 구조와 부트 구성이 서로 일치하지 않으면 아티팩트 다운로드와 설치가 성공하더라도 롤백 동작이 실패할 수 있으므로 부트 연동은 클라이언트 설정의 핵심 요소가 된다.

재부팅 이후 새롭게 설치된 소프트웨어는 영구적으로 승인되기 전에 검증 단계(validation stage)에 진입한다. 임베디드 시스템은 필요한 서비스, 네트워크, 미들웨어(middleware), 센서, 컨트롤러(controller), 애플리케이션 프로세스(application process)가 정상적으로 동작하는지를 평가할 수 있다. 로봇의 경우 단순히 리눅스 로그인 상태에 도달했다는 사실만으로 업데이트 성공을 판단하기에는 충분하지 않다. 운용 상태 검사(operational health check)를 통해 전체 로봇 소프트웨어 스택이 안전하고 사용 가능한 상태로 복귀했는지를 확인해야 한다.

후보 소프트웨어(candidate software)가 필요한 상태 기준(health criteria)을 충족하면 업데이트를 승인된 시스템 상태(accepted system state)로 커밋(commit)할 수 있다. 시작 과정이 실패하거나 애플리케이션 수준 검증(application-level validation)에서 심각한 문제가 발견되면 플랫폼은 이전의 정상 확인 파티션(known-good partition)으로 복귀할 수 있는 능력을 유지해야 한다. 이러한 커밋 또는 롤백(commit-or-rollback) 동작은 업데이트를 통제된 트랜잭션으로 전환하고 실패한 릴리스가 마지막 정상 소프트웨어 이미지를 영구적으로 교체하는 것을 방지한다.

서버 측 배포(server-side deployment)는 개별 업데이트 트랜잭션 위에 오케스트레이션 계층(orchestration layer)을 제공한다. 운영자는 아티팩트를 선택하고 대상 장치 집단(target device population)을 정의한 후 배포를 시작한다. 서비스는 사용 가능한 업데이트 상태에 따라 장치가 대기(pending), 다운로드(downloading), 설치(installing), 재부팅(rebooting), 성공(successful), 실패(failed) 상태인지를 기록한다. 이를 통해 플릿 관리자는 OTA를 관찰할 수 없는 원격 복사 작업으로 처리하지 않고 네트워크 문제, 장치별 실패, 릴리스 전체의 결함을 구분할 수 있다.

장치 그룹화(device grouping)는 여러 하드웨어 리비전(hardware revision), 애플리케이션 역할(application role), 고객 환경(customer environment), 운용 사이트(operational site)를 포함하는 로봇 플릿에서 특히 중요하다. 인벤토리 속성을 이용하여 제품 유형(product type), 하드웨어 세대(hardware generation), 소프트웨어 기준선(software baseline), 기타 배포 관련 특성에 따라 장치를 그룹화할 수 있다. 이후 모든 로봇에 동시에 릴리스를 전송하는 대신 단계적으로 도입함으로써 예상하지 못한 소프트웨어 결함의 운용 영향을 줄일 수 있다.

실용적인 롤아웃 전략(rollout strategy)은 개발 및 실험실 장치에서 시작하여 검증 로봇(validation robot)과 소규모 운영 그룹으로 점진적으로 확대된다. 업데이트 성공 여부와 운용 텔레메트리(operational telemetry)를 검토한 후 동일한 아티팩트를 점점 더 큰 장치 집단으로 승격(promote)할 수 있다. 카나리아 배포(canary deployment)는 위험이 가장 높은 초기 단계에서 새로운 릴리스에 노출되는 로봇 수를 제한한다. 이를 통해 플릿 전체 배포는 단일한 비가역적 작업이 아니라 통제된 승격 과정(controlled promotion process)이 된다.

보안(security)은 통신과 소프트웨어 승인(software authorization)을 모두 포함해야 한다. 전송 암호화(transport encryption)는 클라이언트와 서버 사이의 통신을 보호하고, 아티팩트 인증(artifact authentication)과 통제된 릴리스 프로세스는 소프트웨어 콘텐츠를 보호한다. 장치 자격증명은 정상 클라이언트를 식별해야 하며 서명 자료(signing material)는 보호된 릴리스 인프라 내부에 유지되어야 한다. OTA 플랫폼은 소프트웨어 공급망(software supply chain)의 일부이므로 서버 접근, 아티팩트 생성, 배포 권한, 자격증명, 감사 정보(audit information)에 명확한 보안 제어가 필요하다.

네트워크 중단(network interruption)이 자동으로 업데이트 손상으로 이어져서는 안 된다. 로봇은 액세스 포인트(access point) 사이를 이동하거나 연결이 끊어지고 무선 신호가 약한 영역에 진입하거나 장시간 오프라인 상태가 될 수 있다. 클라이언트-서버 아키텍처는 명확하게 정의된 로컬 상태(local state)를 유지하면서 지연된 업데이트 확인과 중단된 통신을 허용해야 한다. 필요한 아티팩트를 완전히 확보하고 검증한 이후에만 설치를 시작함으로써 신뢰하기 어려운 네트워크 전송과 안전 필수 소프트웨어 활성화(safety-critical software activation)를 분리해야 한다.

운용 통합(operational integration)은 안전하지 않은 시점에 업데이트가 시스템을 방해하지 않도록 해야 한다. 로봇은 정상 작업을 수행하는 동안 아티팩트를 다운로드할 수 있지만 설치 또는 재부팅은 임무 완료(mission completion), 도킹(docking), 충전(charging), 유지보수 시간(maintenance window) 진입까지 연기할 수 있다. OTA 서비스는 어떤 소프트웨어를 배포해야 하는지를 결정하고, 로봇 측 정책(robot-side policy)은 물리적 시스템이 업데이트의 중단성 작업(disruptive operation)을 수행하기에 적합한 상태인지를 결정한다.

결과적으로 Mender 통합(Mender integration)은 릴리스 엔지니어링(release engineering), 중앙 집중식 플릿 관리(centralized fleet management), 보안 장치 통신(secure device communication), 임베디드 설치(embedded installation), A/B 저장장치(A/B storage), 부트 제어(boot control), 검증(validation), 롤백(rollback)을 하나의 통합된 OTA 작업 흐름으로 연결한다. 서버는 가시성(visibility)과 오케스트레이션을 제공하고 클라이언트는 로컬 업데이트 트랜잭션을 보호한다. 단계적 배포(staged deployment)와 로봇 상태를 고려한 활성화 정책(robot-aware activation policy)을 결합하면 운용 중인 로봇 플릿 전체의 임베디드 리눅스 소프트웨어를 유지관리하기 위한 확장 가능한 기반을 제공한다.

## 10.07 OTA Rollback Mechanism Design

![](images/image7.png){width="7.268055555555556in" height="7.268055555555556in"}

OTA 롤백 메커니즘(OTA rollback mechanism)은 새롭게 설치된 릴리스가 정상적으로 동작하지 못할 때 임베디드 시스템(embedded system)을 이전에 검증된 소프트웨어 상태(previously verified software state)로 되돌리는 복구 경로(recovery path)이다. 롤백은 장애 발생 이후 추가하는 기능이 아니라 배포 전에 설계되어야 한다. 로봇 시스템에서는 실패한 펌웨어 업데이트가 마지막 정상 확인 구성(known-good configuration)을 영구적으로 대체하지 않도록 하여 시스템 가용성(availability)과 물리적 안전성(physical safety)을 보호한다.

롤백은 업데이트 아키텍처(update architecture)가 현재 소프트웨어와 후보 릴리스(candidate release)를 동시에 보존할 때 가장 신뢰성 있게 동작한다. A/B 파티션 설계(A/B partition design)에서는 활성 슬롯(active slot)이 정상 확인 이미지(known-good image)를 보유하고 비활성 슬롯(inactive slot)이 업데이트를 수신한다. 후보 이미지가 설치, 부팅, 런타임 검증(runtime validation)을 완료할 때까지 기존 이미지는 변경되지 않는다. 이러한 분리는 부트로더가 원격으로 소프트웨어를 다시 설치하지 않고 이전 슬롯으로 복귀할 수 있는 복구 경계(recovery boundary)를 형성한다.

롤백 수명주기(rollback lifecycle)는 새로운 이미지가 활성화되기 전에 시작된다. 업데이트 프로그램은 현재 신뢰되는 슬롯, 후보 이미지가 포함된 슬롯, 예상 소프트웨어 버전, 업데이트 트랜잭션(update transaction)의 상태를 기록한다. 영구 메타데이터(persistent metadata)는 재부팅과 예기치 않은 전원 손실 이후에도 유지되어야 한다. 신뢰할 수 있는 상태 정보가 없다면 부트로더는 유효한 후보와 불완전한 설치를 구분하거나 복구를 위해 어떤 파티션을 선택해야 하는지 판단하지 못할 수 있다.

설치 이후 후보 파티션(candidate partition)은 즉시 영구적으로 승인되는 대신 시험 상태(trial state)에 진입해야 한다. 부트로더는 다음 시작 시 새로운 슬롯을 선택하지만 이전 슬롯은 정상 확인 폴백(known-good fallback)으로 유지한다. 시험 부팅(trial boot)은 새로운 펌웨어가 정상적으로 초기화되는지를 판단할 수 있는 통제된 관찰 기간(controlled observation period)을 제공한다. 명시적인 확인(explicit confirmation)이 이루어진 경우에만 후보 이미지를 커밋된 소프트웨어 상태(committed software state)로 전환해야 한다.

부팅 시도 횟수 계산(boot-attempt counting)은 시작 과정을 완료하지 못하는 릴리스를 탐지하기 위한 기본적인 메커니즘을 제공한다. 확인되지 않은 이미지를 부팅하기 전에 부트로더는 영구 재시도 카운터(persistent retry counter)를 기록하거나 감소시킨다. 운영체제가 필요한 정상 상태에 도달하면 부팅을 확인하고 시험 조건을 해제한다. 확인 전에 반복적인 리셋이 발생하면 카운터가 최종적으로 실패 임계값(failure threshold)에 도달하고 부트로더는 이전의 정상 확인 파티션을 자동으로 선택한다.

워치독(watchdog)은 시작은 되었지만 응답하지 않는 소프트웨어를 탐지함으로써 부팅 시도 로직(boot-attempt logic)을 보완한다. 새로운 이미지는 커널(kernel)을 성공적으로 로드하더라도 서비스 초기화 중 교착 상태(deadlock)에 빠지거나 반복적으로 충돌하거나 중요한 제어 작업을 유지하지 못할 수 있다. 하드웨어 또는 소프트웨어 워치독은 예상되는 감시 신호(supervision signal)가 사라지면 플랫폼을 리셋할 수 있다. 시험 기간 중 반복되는 워치독 리셋은 롤백을 시작해야 한다는 근거로 사용할 수 있다.

롤백 결정(rollback decision)은 단순히 리눅스 또는 운영체제가 부팅되었는지만을 기준으로 해서는 안 된다. 로봇은 사용자 공간(user space)에 도달하더라도 필수 기능이 사용 불가능한 상태일 수 있다. 따라서 업데이트 후 상태 검증(post-update health validation)은 애플리케이션 서비스, 미들웨어(middleware), 센서 인터페이스(sensor interface), 모터 컨트롤러(motor controller), 안전 컨트롤러(safety controller), 네트워크, 저장장치 및 기타 필수 하위 시스템을 검사해야 한다. 전체 운용 스택(operational stack)이 정의된 상태 기준(health criteria)을 적절한 검증 시간 동안 만족한 이후에만 후보를 승인해야 한다.

애플리케이션 계층(application layer)은 확인 프로토콜(confirmation protocol)에 직접 참여할 수 있다. 시작 이후 상태 관리자(health manager)는 필요한 구성요소를 평가하고 후보 시스템이 허용 가능한 운용 상태에 도달했는지를 보고한다. 검증이 성공하면 시스템은 현재 슬롯이 정상임을 나타내는 영구 확인 정보(persistent confirmation)를 기록한다. 검증이 실패하거나 허용된 타임아웃(timeout) 내에 확인이 이루어지지 않으면 다음 부팅에서 후보를 실패한 것으로 처리하고 폴백 슬롯(fallback slot)으로 복귀할 수 있다.

롤백 메타데이터(rollback metadata)는 부분 쓰기(partial write)와 손상(corruption)으로부터 보호되어야 한다. 슬롯 상태(slot state), 재시도 카운터(retry counter), 확인 플래그(confirmation flag), 선택된 부팅 대상은 전원 중단 가능성이 있는 리셋 직전에 갱신될 수 있다. 원자적 저장 기법(atomic storage technique), 중복 메타데이터(redundant metadata), 체크섬(checksum), 부트로더별 트랜잭션 메커니즘(transactional mechanism)을 사용하여 일관된 상태를 유지할 수 있다. 이전 업데이트가 예기치 않게 종료되었더라도 복구 로직은 항상 안전한 부팅 대상을 결정할 수 있어야 한다.

공유 영구 데이터(shared persistent data)는 추가적인 롤백 문제를 발생시킨다. 이전 시스템 이미지가 그대로 보존되어 있더라도 후보 소프트웨어가 데이터베이스(database), 구성 스키마(configuration schema), 보정 구조(calibration structure), 애플리케이션 데이터를 이전 릴리스가 이해할 수 없는 방식으로 변경할 수 있다. 따라서 견고한 설계에는 하위 호환 데이터 마이그레이션(backward-compatible data migration), 버전 관리 스키마(versioned schema), 가역적 변환(reversible transformation), 중요 데이터의 보호된 복사본이 필요하다. 영구 데이터 때문에 이전 버전을 사용할 수 없다면 소프트웨어 롤백은 완전한 복구가 아니다.

롤백 정책(rollback policy)은 기능적 롤백(functional rollback)과 보안 다운그레이드(security downgrade)를 구분해야 한다. 업데이트 실패 후 바로 이전의 정상 확인 릴리스로 복귀하는 것은 일반적으로 바람직하지만, 임의의 과거 펌웨어 설치를 허용하면 알려진 취약점(vulnerability)이 다시 도입될 수 있다. 따라서 시스템은 복구 규칙과 롤백 방지 보안 정책(anti-rollback security policy)을 결합해야 한다. 승인된 복구 버전은 허용하되 승인된 보안 기준선(security baseline)보다 낮은 이미지는 유효한 과거 서명을 가지고 있더라도 금지할 수 있어야 한다.

롤백 과정에서는 가능하면 실패한 후보를 포기하기 전에 진단 증거(diagnostic evidence)를 보존해야 한다. 부팅 원인(boot reason), 워치독 이벤트(watchdog event), 커널 오류(kernel error), 서비스 실패(service failure), 상태 검사 결과, 소프트웨어 버전, 슬롯 식별정보(slot identity), 업데이트 트랜잭션 식별자를 영구 로그(persistent log)에 저장할 수 있다. 이전 이미지가 다시 동작한 후 이러한 기록을 OTA 백엔드(OTA backend)에 보고하면 장애가 이미지, 하드웨어 호환성, 구성 또는 로컬 장치 상태에서 발생했는지 분석할 수 있다.

플릿 규모(fleet scale)에서는 자동 롤백(automatic rollback)을 배포 오케스트레이션(deployment orchestration)과 연결해야 한다. 하나의 로봇에서 발생한 롤백은 개별 하드웨어 또는 저장장치 문제일 수 있지만 카나리아 그룹(canary group) 전체에서 반복적으로 실패한다면 릴리스 자체의 결함을 의미할 수 있다. 관리 시스템은 업데이트 및 롤백 결과를 집계하고 실패율이 임계값을 초과하면 추가 롤아웃(rollout)을 중지하거나 일시 정지해야 한다. 이를 통해 조사 과정에서 결함이 있는 후보 릴리스가 플릿 전체로 확산되는 것을 방지한다.

운용 상태(operational state)는 롤백과 재부팅 작업을 수행하는 시점에도 영향을 주어야 한다. 중요하지 않은 애플리케이션 문제가 발생한 로봇은 재시작 전에 안전한 위치까지 이동해야 할 수 있지만 안전 필수 구성요소(safety-critical component)가 실패한 경우에는 즉시 안전 상태(safe state)로 전환해야 할 수 있다. 따라서 롤백 컨트롤러(rollback controller)는 재부팅을 독립적인 소프트웨어 작업으로 취급하지 않고 임무 관리(mission management), 전원 관리(power management), 도킹 로직(docking logic), 안전 감시(safety supervision)와 연동해야 한다.

롤백 자체도 설치 과정만큼 적극적으로 테스트해야 한다. 검증에는 손상된 후보 이미지(corrupted candidate image), 중단된 쓰기, 메타데이터 갱신 중 전원 손실, 반복적인 부팅 실패, 워치독 리셋, 상태 검사 타임아웃(health-check timeout), 호환되지 않는 영구 데이터, 네트워크 손실 등을 포함해야 한다. 테스트에서는 단순히 이전 파티션으로 돌아갈 수 있는지만 확인하는 것이 아니라 복구 이후 필요한 로봇 기능을 다시 수행하고 실패한 업데이트를 올바르게 보고할 수 있는지도 검증해야 한다.

RAUC 또는 Mender와 같은 프레임워크(framework)는 업데이트 상태, 슬롯, 아티팩트(artifact), 부트 메커니즘과의 통합을 관리함으로써 이러한 아키텍처에 참여할 수 있지만 프레임워크만으로 로봇별 복구 정확성(robot-specific recovery correctness)을 정의할 수는 없다. 플랫폼 설계자는 어떤 릴리스를 실제로 승인하거나 거부할 것인지 결정하기 위해 상태 기준, 안전한 재부팅 조건, 영구 데이터 호환성, 부트로더 동작 의미(bootloader semantics), 워치독 동작, 플릿 정책을 정의해야 한다.

결과적으로 신뢰할 수 있는 롤백 설계는 정상 확인 이미지(known-good image)를 보존하고 후보 이미지를 별도로 설치한 뒤 시험 부팅을 수행하고 시작 과정을 감시하며 런타임 상태(runtime health)를 검증하고 성공한 경우에만 명시적으로 커밋하는 폐쇄형 복구 루프(closed recovery loop)를 구성한다. 확인 전에 중요한 실패가 발생하면 제어권을 신뢰할 수 있는 슬롯(trusted slot)으로 되돌리고 실패 원인을 기록한다. 이러한 접근 방식은 OTA 배포를 비가역적인 교체 작업에서 복구 가능한 트랜잭션(recoverable transaction)으로 전환하여 장기간 운용되는 임베디드 로봇 시스템에 적합한 업데이트 구조를 제공한다.

## 10.08 OTA During Robot Operation: Partial and Deferred Update

![](images/image8.png){width="7.268055555555556in" height="7.268055555555556in"}

로봇이 운용 상태를 유지하는 동안 OTA 업데이트를 수행하려면 임무 실행 중에도 안전하게 수행할 수 있는 작업과 제어 기능을 방해할 수 있는 작업을 명확하게 분리해야 한다. 다운로드(download), 인증(authentication), 무결성 검사(integrity checking), 스테이징(staging)은 일반적으로 백그라운드(background)에서 수행할 수 있지만, 설치(installation), 재부팅(reboot), 부트 슬롯 전환(boot-slot switching), 안전 필수 소프트웨어(safety-critical software)의 교체는 연기해야 할 수 있다. 이러한 분리는 소프트웨어 유지보수가 물리적 운용을 방해하지 않으면서 시스템 중단시간(downtime)을 최소화한다.

따라서 업데이트 수명주기(update lifecycle)는 획득(acquisition), 준비(preparation), 활성화(activation), 확인(confirmation) 단계로 구분할 수 있다. 획득 단계에서는 업데이트 산출물(update artifact)을 로봇으로 전송하고, 준비 단계에서는 서명 검증, 호환성 검사, 저장공간 확보, 소프트웨어 스테이징을 수행한다. 충분한 컴퓨팅 및 통신 자원을 사용할 수 있다면 이러한 단계는 로봇이 작업하는 동안에도 수행할 수 있다. 반면 활성화는 실행 가능한 구성요소를 변경하거나 중요 서비스의 재시작을 요구할 수 있으므로 별도로 처리한다.

로봇 측 업데이트 관리자(robot-side update manager)는 플랫폼이 각 업데이트 단계를 수행하기에 적절한 상태인지 지속적으로 평가해야 한다. 관련 조건에는 임무 상태(mission state), 속도, 조작 작업(manipulation activity), 페이로드 상태(payload status), 배터리 수준, 충전 상태, 네트워크 품질, CPU 및 메모리 사용률, 열 상태(thermal condition), 안전 컨트롤러 상태(safety-controller status) 등이 포함될 수 있다. 따라서 업데이트 허용 여부는 소프트웨어 정책뿐 아니라 로봇의 현재 물리적 및 운용 상황(physical and operational context)에 따라 결정된다.

부분 업데이트(partial update)는 아키텍처와 종속성(dependency)이 격리를 허용하는 경우 선택된 구성요소를 독립적으로 업데이트할 수 있게 한다. 사용자 공간 애플리케이션(user-space application), 지도(map), 구성 파일(configuration file), AI 모델(AI model), 컨테이너 이미지(container image), 비중요 서비스(noncritical service)는 전체 운영체제 이미지를 업데이트하지 않고 교체할 수 있는 경우가 있다. 그러나 새로운 구성요소가 호환되지 않는 라이브러리, 인터페이스, 펌웨어 또는 통신 프로토콜과 함께 활성화되지 않도록 업데이트 시스템이 종속성을 이해해야 한다.

모든 소프트웨어 구성요소에 동일한 업데이트 정책을 적용해서는 안 되므로 구성요소 분류(component classification)가 중요하다. 내비게이션 지도(navigation map)나 진단 패키지(diagnostic package)는 동적 교체(dynamic replacement)를 허용할 수 있지만, 모터 제어 펌웨어(motor-control firmware), 안전 로직(safety logic), 커널 모듈(kernel module), 저수준 장치 드라이버(low-level device driver)는 통제된 유지보수 상태(controlled maintenance state)를 요구할 수 있다. 따라서 각 구성요소에는 백그라운드 다운로드, 실시간 활성화(live activation), 서비스 재시작, 시스템 재부팅 또는 완전한 로봇 종료 중 어떤 과정이 필요한지를 나타내는 업데이트 등급(update class)을 지정할 수 있다.

지연 업데이트(deferred update)는 소프트웨어를 전달하기 위한 결정과 이를 활성화하기 위한 결정을 분리한다. OTA 백엔드(OTA backend)는 릴리스를 할당하고 로봇은 이를 즉시 다운로드하여 검증할 수 있지만, 활성화는 정의된 조건이 충족될 때까지 대기 상태로 유지된다. 이는 새로운 소프트웨어 버전이 제공되었다는 이유만으로 운송, 검사, 청소, 조작 또는 자율주행 임무를 중단할 수 없는 이동 로봇(mobile robot)에 특히 유용하다.

지연 업데이트 정책(deferred update policy)은 단순한 시간 기준뿐 아니라 운용 이벤트(operational event)를 기준으로 활성화 시점을 정의할 수 있다. 적절한 트리거(trigger)에는 임무 완료(mission completion), 도킹 스테이션 복귀(return to docking station), 충전 연결, 유지보수 구역(maintenance area) 진입, 운영자 승인(operator approval), 예정된 서비스 시간(service period)의 시작 등이 포함될 수 있다. 이벤트 기반 활성화(event-based activation)는 소프트웨어 유지보수를 로봇의 운용 수명주기와 일치시켜 업데이트로 인한 안전하지 않거나 경제적으로 불필요한 작업 중단 가능성을 줄인다.

배터리 상태(battery state)는 특히 중요한 활성화 조건이다. 중간 수준의 충전 상태에서는 다운로드가 가능할 수 있지만 펌웨어 설치와 재부팅에는 전체 업데이트 및 잠재적인 복구 과정까지 완료할 수 있는 충분한 에너지가 필요하다. 이미 충전기에 연결된 로봇은 전원이 안정적이고 일반적으로 정지 상태이므로 더 적합한 환경을 제공한다. 또한 활성화 실패에 대비한 롤백(rollback) 또는 통제된 종료(controlled shutdown)에 필요한 충분한 에너지를 확보해야 한다.

네트워크 상태(network condition)는 획득 단계에 영향을 주지만 중단을 수반하는 설치가 시작된 이후에는 그 영향이 제한되어야 한다. 대용량 산출물은 연결 가능한 동안 점진적으로 다운로드하고 통신이 중단되면 재시도(retry)와 재개 가능한 전송(resumable transfer)을 사용할 수 있다. 활성화 전에 로봇은 필요한 모든 콘텐츠가 로컬에 저장되었으며 무결성 검증을 통과했는지 확인해야 한다. 이를 통해 재부팅된 시스템이 불안정한 무선 연결에 의존하여 부분적으로 획득된 소프트웨어 전환을 완료해야 하는 상황을 방지한다.

백그라운드 업데이트 작업이 자율 기능과 프로세서, 저장장치, 네트워크 인터페이스를 공유하는 경우 자원 관리(resource management)가 필요하다. 해시 계산(hash calculation), 압축 해제(decompression), 델타 재구성(delta reconstruction), 플래시 기록(flash writing)은 상당한 CPU, 메모리, 입출력 대역폭(I/O bandwidth), 열 용량(thermal capacity)을 소비할 수 있다. 업데이트 관리자는 내비게이션, 인지(perception), 계획(planning), 제어(control) 작업이 자원을 필요로 할 때 이러한 작업을 제한하거나 일시 정지하여 OTA 준비 작업이 실시간 로봇 운용보다 낮은 우선순위를 갖도록 해야 한다.

부분 업데이트는 명시적인 종속성 및 호환성 메타데이터(dependency and compatibility metadata)를 사용해야 한다. AI 인지 모델(AI perception model)이 특정 추론 런타임(inference runtime)을 요구하거나 내비게이션 구성요소가 특정 미들웨어 인터페이스(middleware interface)를 요구한다면 업데이트 프로그램은 활성화 전에 이러한 관계를 검증해야 한다. 서로 강하게 결합된 구성요소 그룹은 하나의 트랜잭션 업데이트 집합(transactional update set)으로 처리해야 할 수 있다. 이를 통해 각각의 패키지는 유효하지만 결합된 로봇 소프트웨어 구성이 유효하지 않은 혼합 버전 상태(mixed-version state)를 방지한다.

일부 구성요소는 전체 시스템 재부팅 대신 서비스 수준 재시작(service-level restart)을 지원할 수 있다. 비중요 애플리케이션은 다른 로봇 서비스가 계속 실행되는 동안 중지하고 교체한 후 검증하여 다시 시작할 수 있다. 그러나 실시간 교체(live replacement)가 본질적으로 안전하다고 가정해서는 안 된다. 하나의 서비스를 재시작하는 과정에서 전체 로봇 시스템이 예상치 못하게 불안정해지지 않도록 통신 종속성, 공유 메모리(shared memory), 장치 소유권(device ownership), 프로세스 감시(process supervision), 상태 복원(state restoration)을 평가해야 한다.

지연된 활성화가 최종적으로 전체 운영체제 업데이트를 요구하는 경우에도 A/B 파티셔닝(A/B partitioning)은 중요한 역할을 한다. 로봇이 정상 확인 활성 슬롯(known-good active slot)에서 운용되는 동안 새로운 이미지를 다운로드하여 비활성 슬롯(inactive slot)에 기록할 수 있다. 로봇이 승인된 유지보수 조건에 도달하면 부트로더(bootloader)가 후보 파티션(candidate partition)으로 전환한다. 이후 부팅 후 상태 검증(post-boot health validation)을 통해 후보를 커밋(commit)할지 이전 슬롯으로 복귀할지를 결정한다.

안전 필수 업데이트(safety-critical update)는 로봇 상태 머신(robot state machine)과 더욱 강력하게 연동되어야 한다. 활성화 전에 로봇의 움직임을 정지하고 액추에이터(actuator)를 적절한 안전 상태로 전환하며 페이로드를 안전하게 고정하고 자율 임무 실행을 중지해야 한다. 안전 감시 시스템(safety supervision)은 물리적 플랫폼이 소프트웨어 전환을 수행할 준비가 되었는지 확인해야 한다. 이러한 선행 조건을 만족하지 못하면 원격 배포 일정을 맞추기 위해 강제로 재부팅하는 대신 업데이트를 지연 상태로 유지해야 한다.

OTA 백엔드와 로봇은 이 과정 전체에서 명확한 상태 정보를 공유해야 한다. 유용한 상태에는 할당(assigned), 다운로드 중(downloading), 검증 완료(verified), 스테이징 완료(staged), 지연(deferred), 활성화 준비(ready for activation), 설치 중(installing), 재부팅 중(rebooting), 검증 중(validating), 커밋 완료(committed), 실패(failed), 롤백 완료(rolled back) 등이 있다. 또한 활성 임무, 배터리 부족, 안전하지 않은 위치, 높은 자원 사용률, 유지보수 승인 부재와 같은 지연 원인을 보고하면 로컬 안전 결정을 침해하지 않으면서 플릿 운영자에게 필요한 가시성(visibility)을 제공할 수 있다.

플릿 오케스트레이션(fleet orchestration)은 지연 업데이트와 단계적 롤아웃(staged rollout)을 결합할 수 있다. 새로운 릴리스는 먼저 소규모 카나리아 집단(canary population)에 할당하고 백그라운드에서 다운로드한 후 각 로봇이 독립적으로 적절한 운용 시간대에 도달했을 때만 활성화할 수 있다. 상태 검증이 성공하면 배포 범위를 점진적으로 확대한다. 이러한 방식은 플릿 전체가 동시에 중단되는 상황과 결함이 있는 릴리스가 빠르게 확산되는 상황을 모두 방지하면서 중앙 집중식 소프트웨어 버전 제어를 유지한다.

결과적으로 잘 설계된 운용 중 OTA 전략(operational OTA strategy)은 소프트웨어 전달(software delivery)과 소프트웨어 활성화(software activation)를 서로 다른 제어 문제로 취급한다. 안전한 백그라운드 작업은 자원이 허용되는 동안 수행하고 부분 업데이트는 불필요한 작업 중단을 줄이며 잠재적으로 위험한 전환은 로봇별 조건이 충족될 때까지 연기한다. 로컬 상태 인식(local state awareness), 종속성 관리(dependency management), A/B 복구(A/B recovery), 상태 검증(health validation), 플릿 오케스트레이션을 결합하면 안전성과 복구 가능성(recoverability)을 유지하면서 지속적인 로봇 운용과 OTA 유지보수를 공존시킬 수 있다.

## 10.09 OTA Security: Supply Chain Attack Defense / HSM

![](images/image9.png){width="7.268055555555556in" height="7.268055555555556in"}

OTA 보안(OTA security)은 서버와 로봇 사이의 통신 채널만 보호하는 것에 그쳐서는 안 된다. 전체 소프트웨어 공급망(software supply chain)에는 소스 저장소(source repository), 종속성(dependency), 빌드 시스템(build system), CI/CD 파이프라인(CI/CD pipeline), 아티팩트 저장소(artifact storage), 서명 서비스(signing service), 배포 서버(deployment server), 네트워크 전달, 부트로더(bootloader), 대상 장치가 포함된다. 어느 단계에서든 침해가 발생하면 전체 수명주기에서 신뢰를 독립적으로 검증하지 않는 한 운용상 정상적인 것으로 보이는 악성 펌웨어가 유입될 수 있다.

공급망 공격(supply-chain attack)은 배포된 로봇을 직접 공격하기보다 소프트웨어가 전달되기 전이나 전달되는 과정에서 소프트웨어를 침해하려는 공격이다. 공격자는 소스 코드를 변경하거나 서드파티 라이브러리(third-party library)를 교체하고, 빌드 도구(build tool)를 조작하거나 서명 자격증명(signing credential)을 탈취하며, 업데이트 아티팩트를 바꾸거나 배포 인프라를 침해할 수 있다. OTA 시스템은 소프트웨어를 플릿(fleet) 전체에 자동 배포하므로 하나의 상위 공급망 침해가 정상적인 업데이트 메커니즘을 통해 다수의 로봇으로 악성 코드를 확산시킬 수 있다.

방어는 개발(development), 빌드(build), 릴리스(release), 서명(signing), 배포(deployment), 실행(execution) 환경 사이에 명확한 신뢰 경계(trust boundary)를 설정하는 것에서 시작한다. 단순히 빌드가 성공했다는 이유만으로 소프트웨어가 배포 가능한 상태가 되어서는 안 된다. 각 전환 단계에서는 인증된 식별정보(authenticated identity), 통제된 권한(controlled permission), 추적 가능한 아티팩트(traceable artifact), 검증 가능한 증거(verifiable evidence)를 요구해야 한다. 책임을 분리하면 침해된 개발자 계정, CI 작업자 또는 배포 관리자가 독립적으로 승인되지 않은 운영 펌웨어를 생성하고 배포하는 능력을 제한할 수 있다.

소스 무결성(source integrity)은 신뢰 체인의 첫 번째 계층이다. 저장소는 인증된 접근(authenticated access), 보호된 브랜치(protected branch), 검토된 변경사항(reviewed change), 통제된 병합 정책(controlled merge policy)을 적용해야 한다. 특히 부트로더, 서명 검증(signature verification), 롤백 로직(rollback logic), 업데이트 에이전트(update agent)에 대한 변경은 이후의 보안 보호 기능을 무력화할 수 있으므로 더욱 엄격하게 검토해야 한다. 또한 외부 패키지의 예상하지 못한 변경이 동일한 소스 트리에서 생성되는 펌웨어를 조용히 변경하지 못하도록 종속성 버전(dependency version)을 통제해야 한다.

빌드 환경(build environment)은 보안에 민감한 인프라(security-sensitive infrastructure)로 취급해야 한다. 신뢰할 수 있는 소스 저장소를 사용하더라도 컴파일러(compiler), 빌드 컨테이너(build container), 스크립트(script), 종속성 캐시(dependency cache), CI 실행기(CI runner)가 침해되었다면 신뢰할 수 있는 펌웨어를 보장할 수 없다. 재현 가능하고 격리된 빌드(reproducible and isolated build)는 소프트웨어 생산을 더욱 결정론적이고 독립적으로 검증 가능하게 만들어 이러한 위험을 줄인다. 빌드 결과물에는 릴리스 과정으로 전달되기 전에 고유 식별정보, 암호학적 해시(cryptographic hash), 버전 정보, 추적 가능한 출처 정보(provenance)를 부여해야 한다.

소프트웨어 자재 명세서(SBOM, Software Bill of Materials)는 펌웨어 릴리스에 포함된 구성요소에 대한 가시성(visibility)을 제공한다. SBOM은 이후 공개된 취약점(vulnerability)과 관련될 수 있는 라이브러리, 패키지, 버전, 종속성을 식별할 수 있다. SBOM 자체가 침해를 방지하는 것은 아니지만 추적성(traceability)과 사고 대응(incident response)을 향상시킨다. 취약한 종속성이 발견되면 플릿 운영자는 어떤 펌웨어 릴리스와 로봇 집단이 해당 구성요소를 포함하고 있는지 판단할 수 있다.

아티팩트 저장소(artifact repository)는 또 다른 중요한 신뢰 경계를 형성한다. 펌웨어 이미지, 델타 패치(delta patch), 매니페스트(manifest), 컨테이너(container), 구성 패키지는 빌드 이후 승인되지 않은 변경으로부터 보호되어야 한다. 암호학적 다이제스트(cryptographic digest)는 예상하지 못한 변경을 식별할 수 있으며, 접근 제어(access control)와 불변 저장 정책(immutable storage policy)은 아티팩트 교체 가능성을 줄인다. 서명 대상으로 선택된 아티팩트는 이후 실제로 전달되는 아티팩트와 정확히 동일해야 하며, 릴리스 승인과 OTA 배포 사이에서 검증되지 않은 변경이 발생해서는 안 된다.

디지털 서명(digital signing)은 실행 가능한 소프트웨어에 대한 승인을 확립하지만 그 보호 수준은 개인 서명 키(private signing key)의 보안에 결정적으로 의존한다. 공격자가 운영 서명 키(production signing key)를 탈취하면 악성 펌웨어도 정상적인 서명 검증을 통과할 수 있다. 따라서 서명 자격증명은 일반적인 개발자 시스템과 CI 작업자로부터 격리되어야 한다. 운영 릴리스 프로세스는 컴파일, 패키징 또는 배포를 수행하는 장치에 개인 키를 배포하는 대신 통제된 서명 서비스(controlled signing service)를 호출해야 한다.

하드웨어 보안 모듈(HSM, Hardware Security Module)은 암호학적 키(cryptographic key)를 보호하고 민감한 암호 연산을 수행하기 위한 강화된 환경(hardened environment)을 제공한다. 개인 서명 키를 애플리케이션 메모리나 파일시스템 저장소로 내보내는 대신 HSM 내부에서 키를 생성하거나 가져온 후 보호된 경계 내부에 유지할 수 있다. 승인된 시스템은 서명 요청(signing request)을 제출하고 실제 암호 연산은 HSM 내부에서 수행된다. 이를 통해 가치가 높은 운영 키가 직접 노출될 가능성을 크게 줄일 수 있다.

HSM 통합(HSM integration)은 그 자체로 완전한 해결책으로 간주해서는 안 되며 강력한 승인 정책(authorization policy)과 결합해야 한다. 서명 작업에 대한 접근은 인증된 서비스 식별정보(authenticated service identity), 제한된 역할(restricted role), 승인 작업 흐름(approval workflow), 고위험 릴리스에 대한 복수 승인자 등을 요구할 수 있다. 감사 기록(audit record)은 어떤 식별정보가 서명을 요청했는지, 어떤 아티팩트 다이제스트가 서명되었는지, 작업이 언제 수행되었는지, 어떤 키가 사용되었는지를 기록해야 한다. 이를 통해 OTA 파이프라인에서 가장 강력한 승인 단계에 대한 책임 추적성(accountability)을 확보할 수 있다.

키 계층 구조(key hierarchy)는 운영 키가 침해되었을 때의 영향을 줄일 수 있다. 강력하게 보호되는 루트 키(root key)가 중간 키(intermediate key) 또는 제품별 서명 키(product-specific signing key)를 승인하도록 하고, 일상적인 릴리스에서는 더 제한된 범위의 하위 키(subordinate key)를 사용할 수 있다. 개발, 시험, 운영, 제품군(product family), 보안 도메인(security domain)에 각각 별도의 키를 할당할 수도 있다. 하나의 운영 키가 침해되면 배포된 플릿 전체의 신뢰 루트를 교체하지 않고도 해당 키를 폐기하고 새로운 키로 교체할 수 있다.

로봇은 서버 측 보안에만 의존하지 않고 모든 업데이트를 독립적으로 검증해야 한다. 설치 전에 OTA 클라이언트(OTA client)는 서명된 메타데이터를 인증하고 펌웨어 서명을 검증하며 예상 다이제스트를 확인하고 하드웨어 호환성과 버전 정책(version policy)을 적용해야 한다. 따라서 배포 서버가 침해되더라도 공격자가 장치가 신뢰하는 암호학적 승인 메커니즘(cryptographic authorization mechanism)까지 무력화하지 않는 한 로봇이 임의의 펌웨어를 실행하도록 만들 수 없어야 한다.

보안 부팅(secure boot)은 이러한 검증 체인을 OTA 설치 이후의 시작 과정까지 확장한다. 하드웨어에 고정된 신뢰 루트(hardware-anchored root of trust)가 부트로더 또는 초기 실행 단계를 검증하고, 이후 각 단계가 다음 소프트웨어를 실행하기 전에 검증한다. 이를 통해 변경 불가능하거나 보호된 키 자료(key material)에서 부팅 과정과 운영체제까지 이어지는 신뢰 체인(chain of trust)이 형성된다. OTA 서명과 보안 부팅은 서로 호환되는 신뢰 정책을 사용하여 배포 과정에서 승인된 소프트웨어가 로봇의 실제 시작 시에도 다시 인증되도록 해야 한다.

롤백 방지(anti-rollback protection)는 공격자가 알려진 취약점을 포함하고 있지만 과거에 정상적으로 서명된 릴리스를 설치하여 현재의 보안을 우회하는 것을 방지한다. 보안 버전 카운터(security version counter), 보호된 단조 증가 값(protected monotonic value) 또는 이에 상응하는 정책을 통해 허용 가능한 최소 소프트웨어 세대를 정의할 수 있다. 복구 로직(recovery logic)은 이 메커니즘과 연동되어야 하며, OTA 실패 후 자동 롤백은 요구되는 보안 기준선(security baseline)을 만족하는 승인된 정상 확인 이미지(known-good image)로만 수행되어야 한다.

장치 식별정보(device identity)와 서버 인증(server authentication)은 펌웨어 전달을 둘러싼 관리 관계를 보호한다. 각 로봇은 OTA 서비스가 정상적인 장치를 구분할 수 있는 자격증명(credential)을 가져야 하며, 로봇 역시 자신이 통신하는 서비스를 인증해야 한다. 상호 인증(mutual authentication)은 사칭 위험(impersonation risk)을 줄일 수 있으며, 필요한 경우 장치 자격증명을 신뢰 플랫폼 모듈(TPM, Trusted Platform Module), 보안 요소(secure element) 또는 기타 하드웨어 기반 메커니즘으로 보호할 수 있다. 하나의 로봇이 침해되더라도 플릿 전체의 자격증명이 노출되어서는 안 된다.

플릿 배포 정책(fleet deployment policy)은 암호학적 제어만으로 기능적인 악성 동작을 탐지할 수 없는 경우 추가적인 격리 계층(containment layer)을 제공한다. 새로운 릴리스는 전체 배포 전에 개발 시스템, 검증 로봇(validation robot), 소규모 카나리아 그룹(canary group)에 우선 적용할 수 있다. 런타임 텔레메트리(runtime telemetry), 상태 검사(health check), 예상하지 못한 네트워크 활동, 프로세스 동작(process behavior), 롤백 비율 등을 통해 비정상 동작을 탐지할 수 있다. 사전에 정의된 보안 또는 신뢰성 임계값을 초과하면 배포를 자동으로 일시 중지해야 한다.

사고 대응(incident response)은 키, 인프라 또는 아티팩트가 언젠가는 침해될 수 있다는 가정을 기반으로 해야 한다. 아키텍처는 서명 키 폐기(signing-key revocation), 신뢰 저장소 갱신(trust-store update), 자격증명 교체(credential rotation), 아티팩트 격리(artifact quarantine), 배포 중단(deployment suspension), 영향받은 장치 식별을 지원해야 한다. 출처 기록(provenance record), SBOM, 서명 로그(signing log), 배포 이력(deployment history), 장치 인벤토리(device inventory)를 활용하면 공급망 침해가 의심되는 상황에서 무엇이 빌드되고 승인되고 전달되고 실행되었는지를 재구성할 수 있다.

결과적으로 안전한 로봇 OTA 아키텍처(secure robotic OTA architecture)는 소스 코드에서 물리적 실행(physical execution)에 이르는 연속적이고 검증 가능한 신뢰 체인을 형성한다. 저장소 제어(repository control)는 개발을 보호하고, 격리된 빌드(isolated build)는 소프트웨어 생산을 보호하며, 출처 정보와 SBOM은 추적성을 제공하고, 아티팩트 제어는 릴리스 무결성을 보존한다. HSM 기반 서명(HSM-backed signing)은 승인 키를 보호하고, 장치 검증과 보안 부팅은 로컬에서 신뢰를 강제한다. 여기에 롤백 방지, 단계적 배포(staged deployment), 모니터링(monitoring), 사고 대응을 결합하면 소프트웨어 공급망 공격의 발생 가능성과 플릿 규모의 피해를 모두 제한할 수 있다.

## 10.10 Fleet OTA Orchestration: Staged Rollout Strategy

![](images/image10.png){width="7.268055555555556in" height="7.268055555555556in"}

플릿 OTA 오케스트레이션(fleet OTA orchestration)은 하나의 결함 있는 릴리스가 전체 운용 로봇 집단에 장애를 발생시키지 않도록 하면서 다수의 로봇에 대한 소프트웨어 배포를 조정한다. 개별 장치 업데이트와 달리 플릿 OTA는 서로 다른 하드웨어 리비전(hardware revision), 소프트웨어 기준선(software baseline), 임무 상태(mission state), 네트워크 조건(network condition), 운용 우선순위(operational priority)를 동시에 관리해야 한다. 따라서 오케스트레이션 계층(orchestration layer)은 어떤 소프트웨어를 배포할지뿐만 아니라 어떤 로봇이 이를 수신하고 언제 활성화하며 언제 배포를 중단해야 하는지도 제어한다.

플릿은 먼저 의미 있는 운용 및 기술적 차이를 반영하는 배포 그룹(deployment group)으로 구분해야 한다. 그룹은 하드웨어 세대(hardware generation), 로봇 모델(robot model), 사이트(site), 고객 환경(customer environment), 애플리케이션 역할(application role), 소프트웨어 기준선, 안전 등급(safety classification), 네트워크 영역(network region) 등을 기준으로 구성할 수 있다. 정확한 그룹화는 특정 구성에서 검증된 아티팩트(artifact)가 호환되지 않는 시스템에 의도치 않게 배포되는 것을 방지하고 통제된 단계적 롤아웃(staged rollout)에 필요한 구조를 제공한다.

배포를 시작하기 전에 오케스트레이션 서비스(orchestration service)는 각 대상 로봇에 대한 릴리스 적격성(release eligibility)을 평가해야 한다. 적격성 검사에는 장치 식별정보(device identity), 현재 펌웨어 버전, 하드웨어 호환성(hardware compatibility), 필요한 기준 이미지(base image), 사용 가능한 저장공간, 배터리 상태, 업데이트 정책(update policy), 보안 버전(security version) 등이 포함될 수 있다. 이러한 검사를 통과하지 못한 장치는 지원되지 않는 전환을 강제로 수행하는 대신 제외되거나 지연 상태(deferred state)로 유지된다. 이를 통해 릴리스 할당과 실제 설치 사이에 통제된 경계를 형성한다.

단계적 롤아웃은 플릿 전체를 동시에 업데이트하는 대신 점진적으로 더 큰 로봇 집단에 릴리스를 적용한다. 첫 번째 단계에는 일반적으로 실험실 시스템(laboratory system), 개발 장치(development device), 전용 검증 로봇(validation robot)이 포함된다. 이러한 시스템은 운용 로봇에 새로운 릴리스를 적용하기 전에 설치 동작, 부트 전환(boot transition), 애플리케이션 호환성, 텔레메트리(telemetry), 롤백 메커니즘(rollback mechanism)을 관찰할 수 있는 환경을 제공한다.

실험실 검증 이후에는 소규모 카나리아 그룹(canary group)에 후보 릴리스(candidate release)를 적용할 수 있다. 카나리아 로봇은 예상하지 못한 장애의 영향을 제한하면서 실제 운영 구성을 대표할 수 있어야 한다. 단순히 임의의 장치를 선택하는 대신 중요한 하드웨어 리비전, 작업 부하(workload), 사이트, 환경 조건을 포함하도록 선택해야 한다. 카나리아 단계의 목적은 통제된 실험실 테스트에서 재현되지 않을 수 있는 문제를 실제 환경에서 발견하는 것이다.

하나의 롤아웃 단계에서 다음 단계로의 승격(promotion)은 단순히 경과 시간에 의존하지 않고 측정 가능한 릴리스 상태(release health)를 기준으로 결정해야 한다. 오케스트레이션 시스템은 설치 성공률, 부팅 성공률, 애플리케이션 상태(application health), 롤백 빈도, 충돌률(crash rate), 워치독 리셋(watchdog reset), 통신 안정성, 임무 완료율 및 기타 운용 텔레메트리를 평가할 수 있다. 사전에 정의된 승인 임계값(acceptance threshold)이 관찰 기간 동안 지속적으로 충족되는 경우에만 다음 단계로 진행해야 한다.

배포 웨이브(deployment wave)는 통제된 확장의 다음 단계를 제공한다. 카나리아 운용이 성공하면 플릿을 점진적으로 증가하는 비율이나 사전에 정의된 그룹 단위로 업데이트할 수 있다. 개념적으로 검증 장치에서 소규모 카나리아 집단으로, 이후 제한된 운영 그룹, 지역별 집단(regional population), 최종적으로 전체 플릿으로 롤아웃을 확대할 수 있다. 구체적인 비율 자체보다 각 확장 단계 사이에 관찰 가능한 체크포인트(observable checkpoint)를 유지하는 것이 중요하다.

자동 일시 중지 조건(automatic pause condition)은 운영자가 플릿 전체의 장애 패턴을 충분히 빠르게 인식하지 못할 수 있으므로 필수적이다. 롤백 비율, 부팅 실패, 애플리케이션 충돌, 안전 장애(safety fault) 또는 기타 중요 지표가 설정된 임계값을 초과하면 오케스트레이션 시스템은 추가 로봇에 대한 릴리스 할당이나 활성화를 중지해야 한다. 이미 후보 버전을 실행하는 장치는 계속 모니터링하면서 엔지니어가 발생한 장애 패턴을 조사할 수 있다.

롤백 정보(rollback information)는 롤아웃 결정에 직접 반영되어야 한다. 하나의 로봇에서 발생한 롤백은 로컬 저장장치 손상, 하드웨어 결함 또는 특수한 구성에서 발생한 문제일 수 있지만, 서로 관련되지 않은 여러 장치에서 반복되는 롤백은 릴리스 수준의 문제(release-level problem)를 강하게 나타낸다. 오케스트레이션 서비스는 소프트웨어 버전, 하드웨어 유형, 사이트, 배포 단계별로 이러한 이벤트를 집계하여 추가 승격 전에 개별 장애와 시스템적인 결함(systematic defect)을 구분해야 한다.

가능한 경우 플릿 OTA는 아티팩트 전달(artifact delivery)과 활성화(activation)를 분리해야 한다. 로봇은 정상 임무를 계속 수행하면서 릴리스를 미리 다운로드하고 검증하여 실제 유지보수 시간(maintenance window)에 필요한 시간을 줄일 수 있다. 이후 임무 완료(mission completion), 도킹(docking), 충전(charging), 유지보수 모드(maintenance mode), 운영자 승인(operator approval) 등 로봇이 적절한 운용 조건에 도달했을 때 활성화를 수행할 수 있다. 이를 통해 중앙 집중식 배포 일정이 로컬의 물리적 안전 요구사항을 무시하는 것을 방지한다.

동시성 제어(concurrency control)는 지나치게 많은 로봇이 동시에 사용 불가능한 상태가 되는 것을 방지한다. 창고, 공장, 병원 또는 실외 플릿은 서비스를 유지하기 위해 최소한의 활성 로봇 수를 필요로 할 수 있다. 따라서 오케스트레이션 시스템은 특정 사이트나 기능 그룹에서 동시에 설치 또는 재부팅할 수 있는 장치 수를 제한할 수 있다. 이에 따라 업데이트 일정 관리는 단순한 소프트웨어 배포 문제가 아니라 플릿 가용성 관리(fleet availability management)의 일부가 된다.

로봇과 인프라 사이의 종속성(dependency)도 고려해야 한다. 플릿은 사이트 서버(site server), 지도, API, 통신 프로토콜, 트래픽 관리자(traffic manager), 공유 조정 서비스(shared coordination service)에 의존할 수 있다. 호환성 계획 없이 인터페이스 한쪽만 업데이트하면 각 구성요소가 개별적으로 정상이라도 장애가 발생할 수 있다. 따라서 배포 계획에서는 호환 가능한 버전 범위(compatible version range)를 정의하고 필요한 경우 상호 의존적인 시스템 구성요소의 업데이트를 조정해야 한다.

릴리스 채널(release channel)은 서로 다른 수준의 소프트웨어 성숙도(software maturity)를 관리하는 유용한 메커니즘을 제공한다. 개발(development), 검증(validation), 카나리아(canary), 안정(stable), 장기 지원(long-term support) 채널은 릴리스에 대한 점진적으로 높아지는 신뢰 수준을 나타낼 수 있다. 장치 또는 그룹은 역할에 따라 적절한 채널을 구독한다. 이후 승인된 아티팩트를 다시 빌드하지 않고 채널 사이에서 승격함으로써 이전 단계에서 테스트된 정확히 동일한 소프트웨어가 운영 환경에 배포되도록 한다.

아티팩트 자체는 단계적 롤아웃 전체에서 불변(immutable) 상태로 유지되어야 한다. 카나리아 테스트 이후 릴리스를 다시 빌드하면 소스 코드가 동일하게 보이더라도 서로 다른 바이너리가 생성되어 이전 단계에서 축적된 검증 증거의 일부가 무효화된다. 암호학적 해시(cryptographic hash), 서명(signature), 버전 식별자(version identifier), 빌드 출처 정보(build provenance), 소프트웨어 자재 명세서(SBOM, Software Bill of Materials)는 승격되는 정확한 릴리스를 식별해야 한다. 핵심 원칙은 하나의 아티팩트를 테스트하고, 그 아티팩트를 승인하며, 동일하게 인증된 아티팩트를 배포하는 것이다.

운용 텔레메트리(operational telemetry)는 단계적 배포를 피드백 제어 프로세스(feedback-controlled process)로 전환한다. OTA 상태만으로는 설치 성공 여부를 확인할 수 있지만, 로봇 텔레메트리는 위치추정 실패(localization failure) 증가, 인지 지연(perception latency), CPU 사용률, 열 부하(thermal load), 내비게이션 중단(navigation abort), 임무 수행 시간 증가와 같은 더욱 미묘한 성능 저하를 발견할 수 있다. 후보 릴리스의 동작을 기존 기준선(baseline)과 비교하면 설치 자체는 성공했지만 실제 운용 성능을 저하시키는 릴리스를 탐지할 수 있다.

보안 이벤트(security event) 역시 오케스트레이션에 영향을 주어야 한다. 예상하지 못한 서명 검증 실패(signature failure), 인증 이상(authentication anomaly), 비정상적인 네트워크 활동, 승인되지 않은 버전 전환 또는 의심스러운 장치 동작은 아티팩트, 자격증명(credential), 장치 또는 인프라 구성요소의 침해를 의미할 수 있다. 배포 시스템은 기능적 상태 검사가 정상이라는 이유만으로 배포를 계속하기보다 영향을 받은 장치나 아티팩트를 격리(quarantine)하고 롤아웃을 중단하며 증거를 보존할 수 있어야 한다.

플릿 가시성(fleet observability)을 위해서는 일관된 업데이트 상태 모델(update state model)이 필요하다. 각 로봇은 적격(eligible), 할당(assigned), 다운로드 중(downloading), 검증 완료(verified), 스테이징 완료(staged), 지연(deferred), 설치 중(installing), 재부팅 중(rebooting), 검증 중(validating), 커밋 완료(committed), 실패(failed), 롤백 완료(rolled back) 등의 상태를 보고할 수 있다. 백엔드는 이러한 상태를 배포 수준의 지표와 대시보드로 집계하여 롤아웃이 정상적으로 진행되는 위치, 장치가 대기 중인 위치, 특정 구성이나 환경에 장애가 집중되는지를 운영자가 파악할 수 있도록 한다.

결과적으로 완전한 단계적 롤아웃 전략(staged rollout strategy)은 단방향 브로드캐스트(one-way broadcast)가 아니라 폐쇄형 제어 루프(closed control loop)로 동작한다. 오케스트레이터(orchestrator)는 제한된 로봇 집단을 선택하고 불변의 서명된 릴리스를 배포하며, 안전한 로컬 활성화를 기다리고 업데이트 상태와 운용 상태를 관찰한 후 결과를 승격 기준(promotion criteria)과 비교하여 배포를 확대하거나 일시 중지하거나 롤백한다. 이러한 피드백 기반 접근 방식(feedback-driven approach)을 통해 대규모 로봇 플릿 전체의 안전성, 가용성, 보안성, 복구 가능성(recoverability)을 유지하면서 플릿 소프트웨어를 지속적으로 발전시킬 수 있다.
