# Avikus_Android_assignment_swg
## 어플리케이션 실행 화면
![Screenshot_20250418_180319](https://github.com/user-attachments/assets/0ac5f82a-8247-43da-bf93-33e3ac1402ad) ![Screenshot_20250418_180411](https://github.com/user-attachments/assets/2e3dab82-7548-4e41-97b4-8c73200d89de)

## 1. 프로젝트 실행 방법
   - git clone https://github.com/seowongill/2025_Avikus_Android_SW_assignment_swg.git 을 통해 source code clone
   - Android Studio를 통해 프로젝트 Open
   - Gradle Sync 진행
   - Android Emulator 또는 USB로 연결된 실제 디바이스를 선택하여 앱을 실행합니다.

## 2. 개발 환경
- **Gradle**: 8.6
- **Kotlin**: 1.9.0
- **JVM Target**: 1.8
- **Minimum SDK**: 24
- **Target SDK**: 35
- **Compile SDK**: 35

## 3. 사용한 기술 스택 및 구조 설명

### 사용 기술
- **언어 및 플랫폼**: Kotlin, Android  
- **UI 프레임워크**: Jetpack Compose  
- **비동기 처리**: Kotlin Coroutines + Flow  
- **아키텍처**: MVVM  
- **상태 관리**: StateFlow + collectAsStateWithLifecycle  
- **의존성 관리**: Android Gradle

### 구조 개요
- **View**
  - `MainActivity.kt`: 앱 진입점
  - `MainScreen.kt`: UI 구현 (위치, 속도, 방향 표시 및 계기판)
- **ViewModel**
  - `MainViewModel.kt`: 상태 관리 (Flow 수집 및 StateFlow 변환)
- **Model**
  - `BoatStatus.kt`: 보트 상태 데이터 정의 (위치, 속도, 방향)
  - `BoatStatusRepository.kt`: 인터페이스 정의
  - `BoatStatusRepositoryImpl.kt`: 데이터 생성 로직 구현

#### 계층 간 데이터 흐름
- [Model]BoatStatusRepositoryImpl (emit) -> [ViewModel] MainViewModel.collectLatest() -> MutableStateFlow(_boatStatus) → StateFlow(boatStatus) -> [View] MainScreen.collectAsStateWithLifecycle()

## 4. 기능별 핵심 구현 방식

   * 데이터 수신
     - BoatStatusRepositoryImpl에서 Flow<BoatStatus> 형태로 데이터를 emit
     - MainViewModel에서 이를 수집하여 StateFlow로 변환 후 UI에서 관찰.

   * 위치 정보 표시
     - BoatStatus의 Coordinate(latitude, longitude)를 주기적으로 수신하여 텍스트로 표시합니다.
     - isNaN() 검사를 통해 유효하지 않은 데이터가 수신되었을 때 "위치 정보 없음"으로 텍스트를 구성합니다.

   * 속도 정보 표시
     - BoatStatus의 speed를 주기적으로 수신하여 텍스트로 표시합니다.
     - 기본적으로 표시되는 speed는 m/s 단위로 수신되어지는 speed를 knots 단위로 변환하여 표시합니다.
     - Compose의 DropDownMenu를 통해 단위 선택 드롭다운 메뉴를 구현하였습니다.
     - 사용자는 단위 선택 드롭다운 메뉴를 통해 `knots`, `mph`, `kmp` 중 하나를 선택할 수 있습니다.
     - 사용자가 특정 단위를 선택하면, m/s 단위로 수신한 speed 값을 선택된 단위를 기준으로 하기 공식을 적용하여 변환된 speed를 텍스트로 표시합니다.
        * knots: speed * 1.94384
        * mph: speed * 2.23694
        * kmp: speed * 3.6
     - isNaN() 검사를 통해 유효하지 않은 데이터가 수신되었을 때 "속도 정보 없음"으로 텍스트를 구성합니다.

  * 방향 정보 표시
    - BoatStatus의 heading값을 주기적으로 수신하여 텍스트로 표시 및 나침반 형태의 UI로 표현합니다.
    - 배경 원은 Compose의 Canvas를 이용하여 직접 그렸으며, graphicsLayer를 이용해 나침반 바늘을 heading값에 따라 회전하도록 하였습니다.
    - 중심에는 현재 heading 각도를 roundToInt()를 통해 정수값으로 변환하여 텍스트로 표시하였습니다.
    - 나침반 주변에 동/서/남/북 방향을 표시하는 "N", "S", "W", "E"를 각 방향에 텍스트로 표시하였습니다.
    - isNaN() 검사를 통해 유효하지 않은 데이터가 수신되었을 때 "방향 정보 없음"으로 텍스트를 구성 및 나침반 UI를 표시하지 않습니다.

## 5. 설계 및 구현 포인트
   - MVVM 아키텍쳐를 적용하여 UI와 비즈니스 로직 분리하였습니다.
   - ViewModel에서는 Model에서 제공하는 Flow를 collectLatest를 통해 수신하여 항상 최신 상태를 반영하도록 하였습니다.
   - MutableStateFlow와 StateFlow를 분리하여 외부에는 읽기 전용으로만 상태를 제공하도록 하였습니다.
   - collectAsStateWithLifecycle을 통해 Activity 생명주기 상태에 따라 불필요한 collect를 방지하였습니다.
   - 수신한 BoatStatus에 대한 null 값 검사를 하여 null이 아닐 경우에만 위치, 속도, 방향 정보를 표시하도록 하였습니다.
   - 실시간 데이터 수신 과정에서 발생할 수 있는 NaN 값에 대응하기 위해 NaN 값에 대한 검사 로직을 추가하여 예외 상황에서도 안정적으로 UI를 유지하도록 설계하였습니다.
   - 나침반 바늘의 회전을 자연스럽게 하기 위하여 animateFloatAsState를 활용하였습니다.
   - UI는 Compose만으로 구성하였으며, Box, Row, Column 등의 레이아웃을 조합해 UI를 구현하였습니다.

## 6. 참고 사항
   - 나침반 바늘을 표현하기위해 res/drawable 경로에 direction.png 파일을 추가하였습니다.
   - 나침반의 background color에 사용되는 color 값 "CompassBackground"를 제공해주신 color.kt에 추가하였습니다.
