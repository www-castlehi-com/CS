## IoT란?
### 정의
- 사물 인터넷(Internet Of Thing)
- 연결된 디바이스의 공통 네트워크
- 디바이스와 클라우드 및 디바이스 간 통신을 용이하게 하는 기술
- 인터넷을 통해 다른 기기와 연결하여 데이터를 주고받을 수 있다.

### 특징
1. **자동화 및 통제** : 원격으로 모니터링 및 제어할 수 있어 자동화와 효율성을 향상시킨다.
2. **데이터 수집 및 분석** : 다양한 소스에서 수집된 데이터를 분석하여 통찰력을 제공하고 의사결정을 지원한다.
3. **상호연결성** : IoT 장치는 서로 혹은 인터넷에 연결되어 데이터를 주고받을 수 있다.

## IoT를 도입하면 해결할 수 있는 문제
1. **자원 관리 효율성** : 에너지, 물, 기타 자원의 사용을 최적화하여 관리 효율성을 개선한다.
	> 스마트 조명과 온도 조절 시스템은 에너지 사용을 줄일 수 있다.
2. **생산성 향상** : 제조 및 산업 분야에서 기계의 성능 모니터링, 고장 예측, 생산 공정의 최적화를 통해 생산성을 향상시킨다.
3. **보안 강화** : 보안 카메라, 모션 센서 등 IoT 장치를 통해 개인 주택 및 상업 시설의 보안을 강화할 수 있다.
4. **건강 관리 개선** : 공기 질, 수질, 토양 상태 등 환경적 요소를 모니터링하여 환경 관리에 도움을 준다.
5. **물류 및 공급망 최적화** : 운송 및 물류 분야에서 차량 추적, 재고 관리, 공급망 효율성을 개선할 수 있다.
6. **고객 서비스 개선** : 고객의 선호도와 행동을 분석하고, 이를 바탕으로 맞춤형 서비스를 제공한다.

## IoT 도입 시 고려 요소
1. **구체적인 서비스 이해** : 사용자에게 제공하려는 가치를 명확하게 이해해야 한다.
> 스마트 홈, 스마트 빌딩, 스마트 시티 등 다양한 영역에서 사용자의 편의성, 효율성, 지적호기심, 심미적 만족감 등을 높이는 것이 목적이다.
2. **맞춤형 기술** 융합 : 서비스의 특성에 맞게 다양한 기술들을 효율적으로 융합해야 한다. 각 서비스 영역(집, 건물, 도시)와 특징(전력 소모, 연결 기기 종류 및 수)에 따라 다를 수 있으므로 각각의 필요에 맞는 기술을 적용해야 한다.
3. **암호화** 및 **데이터 보안** : 중요한 데이터(의료 데이터)는 암호화하여 [[클라우드]]로 전송해야 한다. 모든 데이터를 암호화하는 것은 소형 IoT 디바이스의 성능 문제를 야기할 수 있어 디바이스의 성능 한계를 고려해야 한다.
4. **데이터 수집량** 고려 : 수집하는 데이터의 양과 그로 인한 네트워크 및 센서에 대한 스트레를 고려해야 한다.
	> 비행기 데이터 수집의 경우 많은 양의 데이터가 빠르게 생성되므로 네트워크와 센서에 더 많은 부담이 가해진다.

##  IoT 사용 예시

### 스마트 홈
	- 집안의 다양한 장치들(조명, 온도 조절기, 보안 카메라, 스마트 TV)이 인터넷에 연결되어 원격으로 제어되고 자동화
### 웨어러블 기기
	- 스마트 워치나 피트니스 트래커는 사용자의 건강 데이터를 모니터링하고 분석한다.
### 제조 업체 (스마트 팩토리)
    - 생산 라인에 장애 발생 시 탐지
    - 결과물의 손상도 측정
    - 생산공정을 최적화하여 운영비용 절감
### [[스마트팜]]
	- 농장에서 작물의 성장 조건을 모니터링하고, 물과 비료의 사용을 자동화
### 건강 관리
	- 환자 모니터링, 원격 의료 서비스, 의료 기기 관리 등에 사용
### 자동차
    - 장비 고장 감지
    - 원격으로 자동차의 시동 걸기
    - 자동차의 수명을 늘리고 운전자에게 원격으로 정보 제공 가능
    - 미래에는 차량이 스스로 서비스 센터에 예약을 하는 날도 올 것임.
### 운송 및 물류
    - 기상 조건, 차량 가용성을 기반으로 상품을 실어나르는 경로를 변경
    - 재고 추적, 온도 제어 모니터링

### 공공부문
    - 정부가 소유한 공익 사업체에서 사용자에게 대규모 정전, 전력, 하수도 서비스에 발생한 장애를 알림
    - 데이터를 빠르게 수집하여 장애 대응 속도 향상

## IoT 이용 시 예상되는 문제점과 해결 방안
### RISK 1 : 보안 위협
- **문제점** : 취약한 보안 조치로 인해 해킹의 위험에 노출된다. 사생활 침해, 데이터 유출, 장치 조작 등의 위험을 수반한다.
- **해결 방안** : 강력한 보안 솔루션과 암호화 기술의 도입이 필요하다. 장치 제조 단계부터 보안을 고려하고, 네트워크 보안을 강화하며, 정기적인 보안 업데이트와 유지보수를 실시해야한다.

### RISK 2 : 표준 및 호환성 문제
- **문제점** : 다양한 제조업체에서 제작된 장치들이 상호 호환되지 않을 수 있다. 효율적인 시스템 운영을 방해하고, 사용자 경험을 저하시킨다.
- **해결 방안** : 업계 표준의 채택과 준수를 통해 장치들 간의 호환성을 확보해야 한다. 개방형 표준과 프로토콜의 채택으로 다양한 장치들이 원활하게 통신할 수 있도록 해야한다.

### RISK 3 : 데이터 관리 및 개인정보 보호
- **문제점** : 일상에서의 데이터가 수집됨에 따라, 사용자의 개인 정보가 의식되지 않은채로 흘러갈 수 있다.
	> - 미국에서 AI 스피커에게 자신의 음성 녹음 데이터를 들려달라고 요청했는데, 다른 사람의 녹음 데이터 1700건을 제공한 일이 일어났다. 이러한 문제는 사용자에게 녹취작업에 대한 사전작업을 받지 않았음에도 발생할 수 있다는 점에서 더욱 커다란 문제라고 할 수 있다.
- **해결 방안** : 데이터의 수집, 저장, 처리 과정에서 개인정보 보호 규정을 준수해야 한다. 사용자의 동의를 기반으로 데이터를 처리([[마이데이터]])해야 하고, 사용자에게 데이터 관리에 대한 투명성을 제공해야 한다. 해당 산업에서 발생할 수 있는 개인 정보 문제는 반드시 법적으로 규제화한다.

## News

[인공지능 스피커의 반란](https://m.hankookilbo.com/News/Read/A2022030310320001649)