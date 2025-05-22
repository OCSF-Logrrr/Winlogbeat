# 📥 Winlogbeat 8.6.2 설치 및 Logstash 연동 가이드 (Windows)

이 문서는 Windows 환경에서 Winlogbeat 8.6.2를 설치하고, Logstash로 로그를 전송하는 방법을 설명합니다.


## 1️⃣ Winlogbeat 8.6.2 다운로드 & 설치
1. 공식 다운로드 페이지에서 Winlogbeat 8.6.2 ZIP 파일을 받습니다:  
   https://artifacts.elastic.co/downloads/beats/winlogbeat/winlogbeat-8.6.2-windows-x86_64.zip
    
2. 압축을 풀어 `C:\Program Files\Winlogbeat\`에 복사합니다.

  
## 2️⃣ winlogbeat.yml 설정
`C:\Program Files\Winlogbeat\winlogbeat.yml` 파일을 열어, 아래 내용만 포함되도록 수정합니다:
```
winlogbeat.event_logs:            # Winlogbeat에서 수집할 로그 종류
  - name: Application
    ignore_older: 72h
  - name: System
  - name: Security

output.logstash:
  hosts: ["Logstash 서버 IP:5044"]   # ❗❗ 반드시 수정하세요: 실제 Logstash IP로 변경 필요 ❗❗
```
> 💡 YAML은 들여쓰기에 민감합니다. 반드시 **스페이스(공백)** 만 사용할 것!

## 3️⃣ Winlogbeat 서비스 등록 & 자동 실행
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
🖼️ PowerShell에서 Winlogbeat 서비스 시작 후 상태 확인 결과

![image](https://github.com/user-attachments/assets/18f9881a-2aec-47af-973a-38021d03b8e4)

## ❗ 설치 중 오류 발생 시 대처법
![image](https://github.com/user-attachments/assets/7527f6f2-0b6b-415d-9682-69156ef26547)

⚠️ 오류 발생 이유 : 
Powershell 실행 정책 때문에 발생한 것으로, 기본적으로 Powershell에서 .ps1 스크립트 실행을 차단하고 있기 때문에 발생

✅ 해결 방법 :
```
Set-ExecutionPolicy RemoteSigned
입력 후 Y 또는 A 입력
```
---

![image](https://github.com/user-attachments/assets/c6dfb100-4f0d-4ecc-a529-8c75743de8a3)

⚠️ 오류 발생 이유 : 
- PowerShell에서 디지털 서명되지 않은 스크립트를 실행하지 못하게 막는 정책 때문에 발생

- RemoteSigned 정책은 로컬에서 만든 스크립트는 허용하지만, 인터넷에서 다운로드한 파일은 서명되지 않으면 차단

✅ 해결 방법 : 
Unblock-File 명령어로 잠금 해제
```
Unblock-File -Path "C:\경로\스크립트이름.ps1"
```

## 4️⃣ Logstash 재시작 (Docker 환경)
```
docker restart basic-elk-logstash-1
docker logs -f basic-elk-logstash-1
```

## 5️⃣ Kibana에서 수집 확인
1. 웹 브라우저에서 Kibana 접속: http://<KIBANA_HOST>:5601

2. 왼쪽 메뉴 → Discover

3. 인덱스 패턴 드롭다운에서 winlogbeat-* 선택

4. 최신 로그(@timestamp)가 실시간으로 쌓이는지 확인

🎉 실제 이벤트가 보이면 성공입니다!

## 6️⃣ 추가 팁
- 로그 발생 테스트:
  - Powershell 열기 -> Get-Process 실행
  - Windows 이벤트 뷰어(eventvwr.msc) -> Application, Security 탭 확인
