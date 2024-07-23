
1. Elasticsearch

필요한 종속성을 설치
apt install curl wget gnupg2 wget -y

Elasticsearch GPG 키를 추가
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | gpg --dearmor -o /usr/share/keyrings/elastic.gpg

Elasticsearch 저장소를 APT에 추가
echo "deb [signed-by=/usr/share/keyrings/elastic.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" | tee -a /etc/apt/sources.list.d/elastic-7.x.list

저장소의 캐시를 업데이트
apt update -y

Elasticsearch를 설치
apt install elasticsearch -y

Elasticsearch 구성 파일을 편집
network.host : 0.0.0.0
http.port : 9200
discovery.seed_host:["127.0.0.1"]

2. Kibana 설치

kibana GPG key
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -

kibana repository 
sudo sh -c 'echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" > /etc/apt/sources.list.d/elastic-7.x.list'

packager list 업데이트 & kibana 설치
sudo apt-get update
sudo apt-get install kibana

kibana 시작
sudo systemctl start kibana
sudo systemctl enable kibana

kibana 설정파일 수정
sudo vi /etc/kibana/kibana.yml
    elasticsearch.hosts: ["http://localhost:9200"]

웹브라우저에서 확인
http://localhost:5601

3. filebeat 설치

filebeat 설치
sudo apt-get install filebeat

filebeat 설정파일 수정
sudo vi /etc/filebeat/filebeat.yml

    output.elasticsearch:
        hosts: ["localhost:9200"]

    output.logstash:
        hosts: ["localhost:5044"]

    filebeat.inputs:
   - type: log
     enabled: true
     paths: #읽어들일 파일 경로 설정
       - /var/log/*.log

filebeat 실행
sudo systemctl enable filebeat
sudo systemctl start filebeat

4. logstash 파일 설치

logstash 파일 설치
sudo apt-get update
sudo apt-get install logstash

logstash 설정 파일 수정
sudo vi /etc/logstash/conf.d/logstash.conf

input {
  beats {
    port => 5044
  }
}

filter {
  # Add any filters here
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => #elasticsearch에서 보일 인덱스 명
  }
  stdout { codec => rubydebug }
}

logstash 실행
sudo systemctl enable logstash
sudo systemctl start logstash
