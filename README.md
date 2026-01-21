# ROS 2 Humble + CycloneDDS 네트워크 설정 (PC ↔ ROBOT)
이 저장소는 ROS 2 Humble 환경에서 CycloneDDS를 사용하여 두 장비(예: 노트북 PC와 로봇)를 같은 네트워크에서 통신하도록 설정하는 방법을 정리한 매뉴얼입니다. 
> 대상 환경: Ubuntu 22.04 + ROS 2 Humble, 기본 RMW: Fast DDS → CycloneDDS로 교체 설정입니다.

> (예시 인터페이스: `wlo1` (Wi‑Fi), 예시 IP: `192.168.30.109`(PC), `192.168.30.120`(ROBOT))

---

## 1. 사전 준비
두 장비 모두 다음 조건을 만족해야 합니다. 
- Ubuntu 22.04에서 ROS 2 Humble이 설치되어 있어야 합니다. (`ros-humble-desktop` 또는 `ros-humble-ros-base`). 
- 두 장비가 같은 서브넷의 네트워크에 연결되어 있어야 합니다. (예: 192.168.30.XXX 대역). 
- 각 장비의 네트워크 인터페이스 이름과 IP 주소를 확인해 둡니다. 
- 인터페이스 이름 확인 명령어:  
```bash
ip a
```
```bash
ifconfig
```

---

## 2. CycloneDDS RMW 설치
ROS 2 Humble 기본 DDS는 Fast DDS이므로, CycloneDDS용 RMW 패키지를 설치해야 합니다. 
```bash
sudo apt update
sudo apt install ros-humble-rmw-cyclonedds-cpp -y
```

---

## 3. CycloneDDS 설정 파일 작성
두 장비 모두 홈 디렉토리에 CycloneDDS 설정 파일을 생성합니다. 
경로: `~/.config/cyclonedds.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CycloneDDS xmlns="https://cdds.io/config">
  <Domain>
    <General>
      <Interfaces>
        <!-- 사용 중인 네트워크 인터페이스 이름으로 변경 -->
        <NetworkInterface name="wlo1"/>
      </Interfaces>
    </General>

    <Discovery>
      <Peers>
        <!-- PC IP -->
        <Peer address="192.168.30.XXX"/>
        <!-- ROBOT IP -->
        <Peer address="192.168.30.XXX"/>
      </Peers>
    </Discovery>
  </Domain>
</CycloneDDS>
```
- `NetworkInterface name`에는 실제 사용하는 인터페이스 이름(`wlo1`, `wlan0`, `eth0` 등)을 적습니다. 
- `<Peer address="..."/>`에는 통신에 참여하는 모든 장비의 IP를 나열합니다. 

---

## 4. 환경 변수 설정 (.bashrc)
각 장비에서 다음 명령으로 CycloneDDS를 기본 RMW로 사용하도록 설정합니다. 
```bash
echo 'export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp' >> ~/.bashrc
echo 'export CYCLONEDDS_URI=file://$HOME/.config/cyclonedds.xml' >> ~/.bashrc
```
설정 반영:
```bash
source ~/.bashrc
```

---

## 5. 통신 테스트 방법
두 장비에서 ROS 2 노드가 서로 보이는지 간단히 테스트할 수 있습니다. 
### 5.1 공통 준비
두 장비 모두에서:
```bash
source /opt/ros/humble/setup.bash
source ~/.bashrc     # (필요 시)
```

### 5.2 PC: talker 실행 예시
```bash
ros2 run demo_nodes_cpp talker
```

### 5.3 ROBOT: listener 실행 예시
```bash
ros2 run demo_nodes_cpp listener
```
- ROBOT 터미널에서 토픽 메시지(`Hello World: ...`)가 출력되면 CycloneDDS 기반 통신이 정상 작동하는 것 입니다.

---

## 6. 문제 해결
통신이 되지 않을 때 점검할 사항들입니다. 
- 두 장비 모두 `RMW_IMPLEMENTATION=rmw_cyclonedds_cpp`로 설정되어 있는지 확인합니다.  
- `CYCLONEDDS_URI`가 올바른 파일 경로를 가리키는지 확인합니다.  
- `~/.config/cyclonedds.xml`의 인터페이스 이름과 IP 주소 목록이 실제 네트워크 환경과 일치하는지 확인합니다. 
