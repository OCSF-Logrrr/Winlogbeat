# 📥 Winlogbeat 8.6.2 설치 및 설정 가이드 (Windows)

이 문서는 Windows 환경에서 Winlogbeat 8.6.2를 설치하고 TLS를 통해 Logstash로 로그를 전송하는 방법을 설명합니다.


## 1️⃣ Winlogbeat 8.6.2 다운로드 & 설치
1. 공식 다운로드 페이지에서 Winlogbeat 8.6.2 ZIP 파일을 받습니다:  
   https://artifacts.elastic.co/downloads/beats/winlogbeat/winlogbeat-8.6.2-windows-x86_64.zip
    
2. 압축을 풀어 `C:\Program Files\Winlogbeat\`에 복사합니다.

## 2️⃣ 인증서 파일 배치 과정
1. Logstash 컨테이너 접속
   - 터미널에서 `sudo docker exec -it basic-elk-logstash-1 bash`를 실행합니다.

2. CA 인증서 디렉터리로 이동
   - 컨테이너 내부에서 `cd /usr/share/logstash/certs/ca`를 입력해 CA 파일이 있는 폴더로 이동합니다.

3. `ca.crt` 내용 복사 및 윈도우에 저장
   - 아래 명령으로 인증서 내용을 출력합니다.
     - `cat ca.crt`
   - 터미널에 표시된 `-----BEGIN CERTIFICATE-----`부터 `-----END CERTIFICATE-----`까지 전체를 복사합니다.
   - 윈도우의 메모장 등을 열어 복사한 내용을 붙여넣고, `C:\Program Files\Winlogbeat\ca.crt` 경로에 저장합니다.
   - 저장 후 파일을 열어 줄바꿈이 온전하게 유지됐는지 반드시 확인하세요.
  
## 3️⃣ winlogbeat.yml 설정
`C:\Program Files\Winlogbeat\winlogbeat.yml` 파일을 열어, 아래 내용만 포함되도록 수정합니다:
```
winlogbeat.event_logs:
  - name: Application
    ignore_older: 72h
  - name: System
  - name: Security

output.logstash:
  hosts: ["192.168.0.58:5044"]    # Logstash 서버 IP:5044 -> 수정필요
  ssl.enabled: true               # TLS 사용
  ssl.certificate_authorities:    # CA 인증서 경로
    - "C:/Program Files/Winlogbeat/ca.crt"
```

## 4️⃣ Winlogbeat 서비스 등록 & 자동 실행
관리자 권한 PowerShell에서 아래 명령을 순서대로 실행합니다:

```
# 설치 디렉토리로 이동
cd "C:\Program Files\Winlogbeat"

# 서비스 등록
.\install-service-winlogbeat.ps1

# 자동 시작 설정
Set-Service -Name winlogbeat -StartupType Automatic

# 서비스 시작
Start-Service winlogbeat

# 상태 확인
Get-Service winlogbeat
```

## 5️⃣ Logstash 재시작 (Docker 환경)
```
docker-compose restart logstash
docker-compose logs -f logstash
```

## 6️⃣ Kibana에서 수집 확인
1. 웹 브라우저에서 Kibana 접속: http://<KIBANA_HOST>:5601

2. 왼쪽 메뉴 → Discover

3. 인덱스 패턴 드롭다운에서 winlogbeat-* 선택

4. 최신 로그(@timestamp)가 실시간으로 쌓이는지 확인

🎉 실제 이벤트가 보이면 성공입니다!

## 7️⃣ 추가 팁
- 로그 발생 테스트:
  - Powershell 열기 -> Get-Process 실행
  - Windows 이벤트 뷰어(eventvwr.msc) -> Application, Security 탭 확인
