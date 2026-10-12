---
tags: [project, iot, microbit, django, sensor]
aliases: [micro2023, Mr.Sensor's Data]
created: 2023-03
---

> [!NOTE]
> **AI 사용 내역**: 프로젝트 코드·데이터·보고서는 직접 작성. AI(Claude)는 `README.md` 작성에만 사용

# micro2023

micro:bit 센서 데이터 실시간 웹 모니터링 — Mr.Sensor's Data

> [!NOTE]
> micro:bit 센서값 → 시리얼 수신 → MySQL 저장 → Django 웹 대시보드(1초 갱신)

## 개요

- micro:bit이 센서값을 시리얼로 전송
- PC에서 시리얼 수신 후 DB에 저장
- Django 웹페이지가 최신 레코드를 1초 간격으로 조회해 표시

## 데이터 흐름

```
micro:bit ──USB 시리얼──▶ getserial.py / upload.py ──▶ MySQL ──▶ Django /update (JSON) ──▶ 웹페이지 (AJAX 1초)
```

## 측정 항목

| 항목 | 설명 | 표시 |
| --- | --- | --- |
| 초음파 (`ult`) | 거리 | cm |
| 온도 (`temp`) | 온도 | °C |
| 자이로 (`gyro`) | 가속도 X, Y | ° |
| 시간 (`times`) | 저장 시각 | — |

> [!WARNING]
> `microbit.js`의 초음파·온도값은 현재 `randint(0, 10)` 더미 데이터. 가속도(X, Y)만 실측값

## 시리얼 프로토콜

- 115200 baud, 1초 간격 전송
- 형식: `초음파%%온도%%가속도X.가속도Y`

## 구조

```
micro2023/
├── microbit.js      # micro:bit 코드 (센서 읽기 + 시리얼 전송)
├── getserial.py     # 시리얼 수신·파싱 (COM3)
├── db.py            # MySQL 입력·초기화
├── upload.py        # 실행 진입점 (수신 → DB 저장)
├── DB.txt           # DB 설계 메모
└── micro/           # Django 프로젝트
    ├── manage.py
    ├── micro/       # settings, urls, wsgi, asgi
    ├── mainapp/
    │   ├── views.py            # index, update (최신 레코드 JSON)
    │   ├── urls.py
    │   ├── static/             # update.js (AJAX), style.css
    │   └── templates/mainapp/index.html
    └── .config/
        ├── nginx/micro.conf    # Nginx 설정
        └── uwsgi/              # uWSGI 설정, systemd 서비스
```

## 구성 요소

| 파일 | 역할 |
| --- | --- |
| `microbit.js` | 가속도·(더미)센서값을 `%%` 구분자로 시리얼 출력 |
| `getserial.py` | `COM3` 시리얼 읽기 → 리스트 `[ult, temp, gyro]` 변환 |
| `db.py` | `input()` 레코드 삽입, `reset()` 전체 삭제 + AUTO_INCREMENT 초기화 |
| `upload.py` | 모드 선택 실행 (`del` / `debug` / 기본 업로드) |
| `views.py` | `/` 페이지, `/update/` 최신 1건 JSON 응답 |
| `update.js` | 1초마다 `/update` 호출 → DOM 갱신 |

## DB

- MySQL, DB명 `micro`, 테이블 `DATA`

| 컬럼 | 설명 |
| --- | --- |
| `times` | 저장 시각 (`NOW()`) |
| `orders` | 순번 (PK, AUTO_INCREMENT) |
| `ult` | 초음파 |
| `temp` | 온도 |
| `gyro` | 가속도 `X.Y` 문자열 |

## 실행 환경

- 하드웨어: BBC micro:bit, USB 연결
- Python 3
- 의존성

```bash
pip install django pymysql pyserial
```

## 배포

- Nginx + uWSGI + systemd
- HTTPS: Certbot (Let's Encrypt)
- 설정 파일: `micro/.config/`

## 제작

- 윤영서 ([@yys0527](https://github.com/yys0527))
