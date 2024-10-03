# Cisco Nexus 스위치 트래픽 모니터링 그라파나 대시보드 
nexus_traffic_monitor (NTM), Monitors Cisco Nexus 9000 Switches



Cisco Nexus 스위치와 TIG(Telegraf, InfluxDB, Grafana) 연동 설치 가이드
1. 개요
이 문서는 Cisco Nexus 스위치와 TIG Stack(Telegraf, InfluxDB, Grafana)을 연동하여 네트워크 트래픽을 모니터링하는 방법에 대한 설치 및 구성 절차를 설명합니다. 이 가이드는 Ubuntu 환경을 기반으로 하며, 각각의 구성 요소가 올바르게 동작할 수 있도록 상세한 설치 지침과 설정 사항을 포함하고 있습니다.
2. 설치 환경
•	운영체제: Ubuntu 20.04
•	대상 장비: Cisco Nexus Switch
•	사용 소프트웨어:
o	Telegraf - 모니터링 데이터 수집 에이전트 (1.16)
o	InfluxDB - 시계열 데이터 저장소 (1.x)
o	Grafana - 시각화 도구 (10.4.5)
3. 설치 및 설정 절차
3.1. InfluxDB 설치
InfluxDB는 시계열 데이터를 저장하기 위해 사용됩니다. 아래의 명령어를 통해 설치 및 설정을 진행합니다.
1.	InfluxDB 설치:
bash
코드 복사
sudo apt-get update
sudo apt-get install influxdb -y
2.	InfluxDB 활성화 및 상태 확인:
bash
코드 복사
sudo systemctl enable --now influxdb
sudo systemctl start influxdb
sudo systemctl status influxdb
3.	InfluxDB 초기 설정: InfluxDB CLI를 통해 데이터베이스 및 유저를 생성합니다.
bash
코드 복사
influx
InfluxDB CLI에 접속 후 아래 명령어를 입력하여 설정을 완료합니다.
sql
코드 복사
CREATE DATABASE telegrafdb;
CREATE USER telegraf WITH PASSWORD 'your_password';
GRANT ALL ON telegrafdb TO telegraf;
3.2. Grafana 설치
Grafana는 모니터링 데이터를 시각화하는 데 사용됩니다.
1.	Grafana 설치:
bash
코드 복사
sudo apt-get install -y adduser libfontconfig1
wget https://dl.grafana.com/oss/release/grafana_10.4.5_amd64.deb
sudo dpkg -i grafana_10.4.5_amd64.deb
2.	Grafana 활성화 및 서비스 시작:
bash
코드 복사
sudo systemctl daemon-reload
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
sudo systemctl status grafana-server
3.	Grafana 웹 UI 접속:
o	브라우저에서 http://<Grafana 서버 IP>:3000으로 접속
o	초기 계정: admin / admin
o	로그인 후 패스워드 변경
3.3. Telegraf 설치 및 설정
Telegraf는 다양한 소스로부터 메트릭을 수집하고 InfluxDB로 전달하는 역할을 합니다.
1.	Telegraf 설치:
bash
코드 복사
wget https://dl.influxdata.com/telegraf/releases/telegraf_1.16.2-1_amd64.deb
sudo dpkg -i telegraf_1.16.2-1_amd64.deb
2.	Telegraf 활성화 및 서비스 시작:
bash
코드 복사
sudo systemctl enable --now telegraf
sudo systemctl start telegraf
sudo systemctl status telegraf
3.	Telegraf 설정 파일 수정: 설정 파일 telegraf.conf를 수정하여 Nexus 스위치의 트래픽 데이터를 수집합니다.
bash
코드 복사
sudo vi /etc/telegraf/telegraf.conf
아래 내용을 [[inputs.exec]] 섹션에 추가합니다:
ini
코드 복사
[[inputs.exec]]
  commands = ["sudo python3 /home/jaheo/nexus_traffic_monitor/telegraf/nexus_traffic_monitor_high_frequency.py /home/jaheo/nexus_traffic_monitor/telegraf/switch1.txt influxdb-lp -vv"]
  interval = "5s"
  timeout = "28s"
  data_format = "influx"
4.	Telegraf에 권한 부여: Telegraf가 외부 스크립트를 실행하기 위해 권한을 부여합니다.
bash
코드 복사
sudo visudo
아래 항목을 추가합니다:
sql
코드 복사
telegraf ALL=(ALL) NOPASSWD:ALL
5.	Telegraf 재시작:
bash
코드 복사
sudo systemctl restart telegraf
3.4. Grafana 데이터 소스 설정
Telegraf가 InfluxDB로 전송한 데이터를 Grafana에서 시각화하기 위해 데이터 소스를 설정합니다.
1.	Grafana UI에 접속하여 InfluxDB 데이터 소스 추가:
o	Configuration -> Data Sources -> Add data source 선택
o	데이터베이스 유형으로 InfluxDB 선택
o	URL에 http://localhost:8086 입력
o	데이터베이스: telegrafdb
o	사용자명 및 비밀번호: telegraf / your_password
2.	데이터 소스 연결 테스트: Save & Test 버튼을 클릭하여 연결이 성공적으로 되었는지 확인합니다.
3.5. 대시보드 구성 및 시각화
1.	Grafana 대시보드 템플릿 추가:
o	좌측 메뉴에서 + -> Import를 선택
o	템플릿 ID를 입력하여 대시보드 추가
2.	실시간 데이터 확인: Nexus 스위치의 트래픽 데이터를 실시간으로 모니터링하고, 필요한 메트릭을 대시보드에 추가하여 원하는 형태로 시각화합니다.
4. 트러블슈팅
•	InfluxDB Bad Gateway 에러 발생 시:
o	Grafana에서 데이터 소스 URL을 올바르게 입력했는지 확인합니다.
o	http://<InfluxDB 서버 IP>:8086 형태로 입력합니다.
•	Telegraf 권한 문제 발생 시:
o	sudo visudo를 통해 Telegraf에 올바른 권한이 부여되었는지 확인합니다.
5. 데이터 보존 정책 설정
•	30일 이상 데이터 보존 정책 적용:
sql
코드 복사
CREATE RETENTION POLICY "one_month" ON "telegrafdb" DURATION 30d REPLICATION 1 DEFAULT;
o	해당 정책이 적용되었는지 확인하려면 아래 명령어를 사용합니다.
sql
코드 복사
SHOW RETENTION POLICIES ON "telegrafdb";
6. 참고 링크
•	Nexus Traffic Monitoring 설치 가이드
•	Grafana, InfluxDB, Telegraf 설치 가이드
7. 마무리
이 문서에서는 Cisco Nexus 스위치와 TIG 스택을 이용하여 트래픽 데이터를 수집하고 시각화하는 방법에 대해 설명했습니다. 모든 절차를 따라 완료한 후, Cisco 장비의 네트워크 트래픽을 효과적으로 모니터링할 수 있습니다.
문서의 모든 설정이 정상적으로 이루어졌는지 확인하고, 필요 시 로그와 설정 파일을 점검하여 문제를 해결하시기 바랍니다.

