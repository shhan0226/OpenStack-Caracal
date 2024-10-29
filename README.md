# OpenStack-Caracal
Installation Script for OpenStack Caracal(2024.1) on Ubuntu 22.04 LTS

## 1. Overview
- 이 예제는 OpenStack 공식 문서인 [install-guide](https://docs.openstack.org/install-guide/overview.html)을 기준으로 작성합니다.

### 1.1. Example architecture
- Hardware Requirements
  - Controller Node1 
    - CPU 1~2, Memory 4 GB+, Storage 5 GB+
  - Compute Node1
    - CPU 2~4, Memory 2 GB+, Storage 10 GB+
- Networking Option 2: Self-service networks
  - ![image1](https://docs.openstack.org/install-guide/_images/network2-services.png)


## 2. Environment
- Identity, Image, Compute, Networking, Dashboard, Object Storage의 서비스 등은 독립적으로 작동할 수 있다. 
- 다만, Object Storage 서비스만 사용할 경우 아래를 예제를 사용한다
  - [Object Storage Installation Guide for 2023.2 (Bobcat)](https://docs.openstack.org/swift/2023.2/install/)
  - Block Storage와 같은 옵션 서비스가 있는 설치의 경우 Logical Volume Manager(LVM)를 고려해야 한다.
- 모든 명령어는 Root 계정으로 사용한다.
- 시스템 요구사항 가이드는 아래를 참고합니다.
  - [OpenStack 2023.2(Bobcat) 관리자 가이드](https://docs.openstack.org/2023.2/admin/)


## 3. Host networking

### 3.1. Controller Node
- 네트워크 인터페이스 구성
  - `/etc/network/interfaces`
    ```
    # The provider network interface
    auto INTERFACE_NAME
    iface INTERFACE_NAME inet manual
    up ip link set dev $IFACE up
    down ip link set dev $IFACE down
    ```
- 호스트 이름 구성
  - 노드의 host name은 `controller`로 설정
  - `/etc/hosts`
    ```
    # controller
    10.0.0.11       controller

    # compute1
    10.0.0.31       compute1

    # block1
    #10.0.0.41       block1

    # object1
    #10.0.0.51       object1

    # object2
    #10.0.0.52       object2
    ```
  - /etc/hosts에서 호스트 이름을 다른 루프백 IP(`127.0.1.1`) 주소 항목으로 추가될 경우가 있으므로, 이 항목을 주석 처리하거나 제거해야 합니다. `127.0.0.1` 항목은 제거하지 마십시오.    

  ### 3.2. Compute Node
- 네트워크 인터페이스 구성
  - `/etc/network/interfaces`
    ```
    # The provider network interface
    auto INTERFACE_NAME
    iface  INTERFACE_NAME inet manual
    up ip link set dev $IFACE up
    down ip link set dev $IFACE down
    ```
- 호스트 이름 구성
  - 노드의 host name은 `compute1`로 설정
  - `/etc/hosts`
    ```
    # controller
    10.0.0.11       controller

    # compute1
    10.0.0.31       compute1

    # block1
    #10.0.0.41       block1

    # object1
    #10.0.0.51       object1

    # object2
    #10.0.0.52       object2
    ```
  - /etc/hosts에서 호스트 이름을 다른 루프백 IP(`127.0.1.1`) 주소 항목으로 추가될 경우가 있으므로, 이 항목을 주석 처리하거나 제거해야 합니다. `127.0.0.1` 항목은 제거하지 마십시오.  

  ### 3.3. Verify connectivity
  - 서로 다른 노드의 연결을 확인한다.
    ```
    ping -c 4 compute1
    ```


## 4. Network Time Protocol (NTP)

### 4.1. Controller Node
- Install and configure components
  ```
  apt install chrony
  ```
- Edit the `/etc/chrony.conf`
  ```
  NTP_SERVER=192.168.1.113
  ALLOW_SERVER=192.168.0.0/22
  ```
  - NTP_SERVER 설정
    ```
    cat <<EOF | sudo tee test.conf
    server $NTP_SERVER iburst  
    EOF
    ```
  - ALLOW_SERVER 설정
    ```
    cat <<EOF | sudo tee -a test.conf  
    allow $ALLOW_SERVER
    EOF
    ```
- Restart the NTP service
  ```
  service chrony restart
  ```  
- Verify operation
  ```
  chronyc sources
  ```
  
### 4.2. Other Node
- Install and configure components
  ```
  apt install chrony
  ```
- Edit the `/etc/chrony.conf`
  ```
  CONTROLLER_SERVER=192.168.1.113  
  ```
  - NTP_SERVER 설정
    ```
    cat <<EOF | sudo tee test.conf
    server $CONTROLLER_SERVER iburst  
    EOF
    ```
- Restart the NTP service
  ```
  service chrony restart
  ```  
- Verify operation
  ```
  chronyc sources
  ```