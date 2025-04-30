# 🛠 Kiosk 프로젝트 - 5일간 문제 요약 및 원인 분석 (with README)

## 📍 프로젝트 개요
React + Spring Boot 기반 키오스크 프로젝트의 배포 과정에서, 화면 미출력 및 결제 실패 등의 이슈가 5일간 지속됨. 원인은 **초기 설정 실수**와 **배포 경로 불일치**로 확인됨.

---

## ✅ 주요 문제 흐름 요약

### 1. **초기 Nginx 설정**
```nginx
root /home/ubuntu/kiosk-frontend;  # ✅ 설정은 이 경로로 되어 있었음
```
- 즉, Nginx는 `/home/ubuntu/kiosk-frontend/index.html`을 보여주는 구조

### 2. **React 빌드 파일 복사 경로 실수**
```bash
# ⛔ 다음과 같이 잘못된 위치로 복사 진행함
scp -i Lightsail.pem -r dist/* ubuntu@서버:/var/www/html/
```

### ✅ 이로 인해 발생한 현상
| 증상 | 원인 |
|------|------|
| 화면이 이전 상태로 유지됨 | dist를 Nginx가 보지 않는 곳에 복사함 |
| QR 결제 테스트 실패 | 잘못된 화면 + 잘못된 메뉴 ID로 주문 요청됨 |
| 메뉴 목록 조회 안됨 | 프론트에서 빌드가 반영되지 않아 요청이 일치하지 않음 |

---

## ❌ ChatGPT의 책임 부분
- 초기 질문에서 root 경로 확인을 놓치고 `scp → /var/www/html` 방식으로 안내함
- 이후 유저가 계속 배포 실패를 겪는 원인을 **서버/도메인 문제로 착각**하게 만듦

---

## 🧾 지금까지의 흐름 정리

### 📂 디렉토리 구조 확인
```bash
# root 설정 확인
sudo vi /etc/nginx/sites-available/default
# 실제 배포 파일 위치 확인
ls /home/ubuntu/kiosk-frontend
```

### ✅ 수정된 배포 명령
```bash
# React에서 빌드
npm run build

# 올바른 위치로 dist 파일 복사
scp -i Lightsail.pem -r dist/* ubuntu@서버:/home/ubuntu/kiosk-frontend/
```

### ✅ 이후 명령
```bash
# 백엔드 실행
cd /home/ubuntu/kiosk-system
sudo java -jar kiosk-backend-0.0.1-SNAPSHOT.jar

# 프론트 재시작 필요 없음 (Nginx가 정적 파일만 서빙)
```

---

## 📦 서버 구성 체크리스트 (매번 확인)

- [x] Nginx root 경로가 어디인지 확인함?
- [x] React `dist` 파일을 정확히 그 경로에 복사했는가?
- [x] `.env` 또는 `axiosInstance.js`의 baseURL이 배포 주소인지 확인함?
- [x] React build 후 dist 폴더 안에 index.html과 assets가 있는지 확인함?
- [x] 서버에서 `.jar` 실행이 잘 되었는가?

---

## 🔁 제안: 재배포 전체 순서

1. 로컬 React 프로젝트에서 `npm run build`
2. `scp` 명령어로 `/home/ubuntu/kiosk-frontend/`에 복사
3. 백엔드 `.jar` 서버 다시 실행 (변경점 있으면 재빌드)
4. 카카오 설정 및 API key 점검 (CID, adminKey 등)
5. `/api/payment/ready` 요청 정상 작동 확인
6. 결제 흐름 최종 점검 (성공/취소/실패 리다이렉트 경로 포함)

---

## ✅ 결론

> 문제의 핵심은 "Nginx가 바라보는 경로"와 "실제로 복사한 dist 경로"가 **일치하지 않아** 발생한 것입니다. 이는 서버 문제도, 카카오 문제도 아니었으며, **정적 파일 위치 불일치**가 핵심 원인이었습니다.

---

## 📌 참고

- nginx root 위치 확인: `/etc/nginx/sites-available/default`
- React build: `npm run build`
- jar 실행: `java -jar [파일명]`
- nginx 재시작 (변경 시): `sudo systemctl restart nginx`

---

위 내용을 README.md로 저장하여 프로젝트 내 공유하는 것을 추천합니다.

