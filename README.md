# ELK 스택 구축 (리눅스 설치) 🔨
## Pipe Line 구축(개요)
<br/>

![Pipe Line 개요](Pipe_Line.png)
<br/><br/>
## 0. Review

**1단계 - jdk 실행 환경 수정**

-> ES 정상 실행, logstash, 실행시 문제 발생

-> 문제 해결책 : logstash의 jdk 경로 설정

방식 :
- cmd 창으로 관리자 모드로 실행 권장
- 셋팅 후 종료 재실행 후 logstash등 실행  
<br/>

**2단계 - 실행순서**

ES -> logstash -> filebeat

es/bin : es.bat

logstash/bin : logstash.exe?

filebeat : filebeat.exe

<br/>**3단계 - 실행을 위한 선행 단계**<br/>
csv파일에 데이터 저장 - filebeat은 이미 read한 파일은 파일명 수정을 해도 전혀 ~ 새로운 데이터로 간주하지 않아서 ES에 index가 추가되지 않음
-> filebeat.yml 수정 -> logstash/conf/*.conf 파일로 전처리 로직을 적용, filebeat로 요청받는 port, es(비정형)에 index 명칭 설정 후 output지정

<br/>**4단계 - 실행 명령어**

ES - 단순 실행

logstash - C:\02.devEnv\ELK\logstash-7.11.1\bin>logstash -f ./config/*.conf
filebeat - >filebeat -e -c filebeat.yml

<br/>**5단계 - 확인 및 test**

- 브라우저 head에서 생성 또는 이미 존재하는 index명인 경우 새로 추가된 데이터 확인
- csv 파일에 새로운 데잉터 추가 저장 후 콘솔찰 확인
- 브라우저 head에서 확인 <br/>
<br/>

## 1. 설치

### **Elastic**

- 리눅스에 설치하기 위한 코드

```bash
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.11.1-amd64.deb
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.11.1-amd64.deb.sha512
shasum -a 512 -c elasticsearch-7.11.1-amd64.deb.sha512 
sudo dpkg -i elasticsearch-7.11.1-amd64.deb
```

- 상태 확인

```bash
systemctl status elasticsearch.service
```

- localhost와 통신을 위한 curl 명령어 사용

```bash
curl -X GET "localhost:9200/"
```

### logstash

- wget을 통한 해당 버전 파일 다운

```bash
sudo systemctl stop logstash
sudo apt-get purge logstash
sudo apt-get autoremove
sudo rm -rf /etc/logstash
sudo rm -rf /var/lib/logstash
sudo rm -rf /usr/share/logstash

sudo apt update
sudo apt install apt-transport-https ca-certificates wget
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo sh -c 'echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" > /etc/apt/sources.list.d/elastic-7.x.list'
sudo apt update
```

- 7.11.1 버전 설치

```bash
sudo apt install logstash=1:7.11.1-1
```

### Filebeat

- 설치 방법

```bash
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.11.2-amd64.deb
sudo dpkg -i filebeat-7.11.2-amd64.deb
```

- 호스트 OS와 게스트 OS간의 통신을 위해 포트 포워딩

![Virtualbox 포트포워딩 수정](/Port.png)



- Elasticsearch.yml 파일 수정
    - sudo su 명령어를 통해 /etc/elasticsearch/ 접근
    - elasticsearch.yml<br/><br/>
    
    ```bash
    ## 수정한 부분
    
    ---------------path---------------
    path.data: /var/lib/elasticsearch
    path.logs: /var/log/elasticsearch
    ------------Network---------------
    **network.host: 0.0.0.0**
    ------------- Various --------------
    discovery.type: single-node 
    ```
<br/>  

- Filebeat.yml 파일 수정
    - sudo su 명령어를 통해 /etc/filebeat/ 접근
    - filebeat.yml
    ```bash
    ================ Filebeat inputs ================
    
    filebeat.inputs:
    
    - type: log

  enabled: true

  paths:
    - /home/username/bank.csv ## 기존 Win와는 다른 path
    
    ------------- Logstash Output -------------------
    output.logstash:
    
        hosts: ["localhost:5044"]
    ```

- bankfisa3.conf 파일
    - /etc/logstash 경로로 bankfisa3.conf 파일 생성<br/><br/>

- 실행 결과

![result1](/result1.png)<br/>
![result2](/result2.png)
<br/><br/>

## 2. 실패 예외 상황 😂

- 설치 파일들을 너무 많이 적재한 결과 용량이 오버되는 현상이 발생하여 설치에 있어 어려움을 겪었습니다. **→ VM에서 새로운 가상 머신을 생성하여 진행하였습니다.**
- Elasticsearch.yml 파일에서 ‘network.host: 0.0.0.0’ 주석을 해제하였을 때 systemctl start elasticsearch.service 에러가 계속해서 발생하였습니다.
**→ Various 부분에 discovery.type: single-node 명령어를 입력하여 해결하였습니다.**
- Kibana 까지 설치를 완료하지 못하였습니다. → 쭈글… <br/><br/>

## 3. 보완해야할 사항 🦾

- 반복작업을 줄이기 위해 실행했던 명령어들을 .sh파일로 정리 中 입니다.