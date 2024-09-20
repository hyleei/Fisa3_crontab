# 🖥️ 일일 시스템 모니터링 툴

🌟 매일 아침, 신선한 시스템 보고서와 함께 상쾌한 하루를 시작하세요! 🌄

## 🙋 팀원

- [박현서](https://github.com/hyleei)
- [최나영](https://github.com/na-rong)
- [박지원](https://github.com/jiione)

## 📅 프로젝트 기간

2024년 9월 19일 

## 📋 개요

이 프로젝트는 Linux 시스템의 CPU, 메모리, 디스크 사용량을 주기적으로 체크하고 일일 보고서를 생성합니다.

## ✨ 주요 기능

- 🕒 정기적인 시스템 상태 체크 (15분마다)
- 📊 일일 시스템 사용 보고서 생성
- 🧹 오래된 로그 및 보고서 자동 정리 (7일 이상 된 파일)

## 🛠 설치 방법

1. 스크립트 디렉토리 생성
   ```bash
   mkdir -p $HOME/scripts
   ```

2. `check_system.sh` 파일 생성 및 내용 추가
   ```bash
   nano $HOME/scripts/check_system.sh
   ```
   
   ```bash
   #!/bin/bash

   timestamp=$(date "+%Y-%m-%d %H:%M:%S")
   cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2 + $4}')
   memory_usage=$(free | awk '/Mem:/ {printf "%.2f", $3/$2 * 100}')
   disk_usage=$(df -h / | awk '/\// {print $(NF-1)}' | sed 's/%//')

   echo "$timestamp - CPU: ${cpu_usage}%, Memory: ${memory_usage}%, Disk: ${disk_usage}%"
   ```

3. `generate_daily_report.sh` 파일 생성 및 내용 추가
   ```bash
   nano $HOME/scripts/generate_daily_report.sh
   ```
   
   ```bash
   #!/bin/bash

   log_file="$HOME/logs/system_status.log"
   yesterday=$(date -d "yesterday" "+%Y-%m-%d")

   echo "Daily System Report for $yesterday"
   echo "=================================="

   grep "$yesterday" "$log_file" > temp_log.txt

   if [ -s temp_log.txt ]; then
       cpu_avg=$(awk -F'[:,]' '{sum+=$3} END {print sum/NR}' temp_log.txt)
       memory_avg=$(awk -F'[:,]' '{sum+=$4} END {print sum/NR}' temp_log.txt)
       disk_avg=$(awk -F'[:,]' '{sum+=$5} END {print sum/NR}' temp_log.txt)

       echo "Average CPU Usage: ${cpu_avg}%"
       echo "Average Memory Usage: ${memory_avg}%"
       echo "Average Disk Usage: ${disk_avg}%"

       cpu_peak=$(awk -F'[:,]' '{if($3>max) max=$3} END {print max}' temp_log.txt)
       memory_peak=$(awk -F'[:,]' '{if($4>max) max=$4} END {print max}' temp_log.txt)
       disk_peak=$(awk -F'[:,]' '{if($5>max) max=$5} END {print max}' temp_log.txt)

       echo "Peak CPU Usage: ${cpu_peak}%"
       echo "Peak Memory Usage: ${memory_peak}%"
       echo "Peak Disk Usage: ${disk_peak}%"
   else
       echo "No data available for yesterday."
   fi

   rm -f temp_log.txt

   echo "=================================="
   echo "End of Report"
   ```

4. 실행 권한 부여
   ```bash
   chmod +x $HOME/scripts/check_system.sh
   chmod +x $HOME/scripts/generate_daily_report.sh
   ```

5. 로그 및 보고서 디렉토리 생성
   ```bash
   mkdir -p $HOME/logs $HOME/reports
   ```

## ⚙️ Crontab 설정

crontab을 열어 다음 내용을 추가합니다:

```bash
crontab -e
```

```cron
# 15분마다 시스템 체크
*/15 * * * * $HOME/scripts/check_system.sh >> $HOME/logs/system_status.log

# 매일 오전 6:55에 일일 보고서 생성
55 6 * * * $HOME/scripts/generate_daily_report.sh > $HOME/reports/daily_report_$(date +\%Y\%m\%d).txt

# 매일 오전 7시에 오래된 로그 및 보고서 정리
0 7 * * * find $HOME/logs -mtime +7 -type f -delete && find $HOME/reports -mtime +7 -type f -delete
```

## 📊 사용 방법

설치 및 crontab 설정 후, 시스템이 자동으로 모니터링 시작

- 시스템 상태 로그 확인:
  ```bash
  cat $HOME/logs/system_status.log
  ```

- 최신 일일 보고서 확인:
  ```bash
  cat $HOME/reports/daily_report_$(date +%Y%m%d).txt
  ```

## ❗ 주의사항

- 스크립트 경로가 홈 디렉토리 기준으로 설정 -> 필요시 절대 경로로 변경
- 로그 파일이 너무 커지지 않도록 주기적으로 확인 필요

## 🐛 문제 해결

로그나 보고서가 생성되지 않는 경우
1. 스크립트 실행 권한 확인
2. cron 서비스 실행 중인지 확인: `sudo systemctl status cron`
3. cron 로그 확인: `grep CRON /var/log/syslog`

---



