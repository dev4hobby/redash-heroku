# Redash on Heroku

서비스를 운영하면서 주로 사용하는 쿼리를 Redash에 저장하여 쿼리 결과를 조회하거나 대시보드로 구성할 수 있습니다.  
이 저장소는 Redash를 Heroku 환경에서 사용하기위해 생성된 저장소입니다.

## 환경 생성

컨테이너 기반 환경을 생성해줍니다.

```bash
heroku create --stack=container ${YOUR_HEROKU_APP_NAME}
```

## Add-ons 설치

Heroku는 다양한 애드온을 제공하고있습니다.  

> Redash는 설정정보 관리 및 캐싱을 위해 [Postgres, Redis]를 사용합니다.

### 필수 요소

- [heroku-postgresql](https://elements.heroku.com/addons/heroku-postgresql)
- [heroku-redis](https://elements.heroku.com/addons/heroku-redis)

위의 두 Add-ons는 설정정보 관리 및 캐싱을 위한 필수 설치요소입니다.
두 Add-ons를 설치하면 DATABASE_URL, REDIS_URL 환경변수가 자동으로 추가됩니다.

### 추가 요소

이메일 전송을 위한 Add-ons를 설치해도 되지만, 굳이 필요없다면 하지않아도 좋습니다.
다만 쿼리 실행 실패 및 기타 Alert 발생시 이메일로 알람을 받기위해 설치해야 합니다.

Sendgrid 혹은 기타 이메일 전송을 위한 Add-ons를 추가하면 됩니다.

## Heroku 환경변수

서비스 운영을 위해 필요한 환경변수를 Heroku에서 설정해야합니다.

```yml
PYTHONUNBUFFERED=0
QUEUES=queries,scheduled_queries,celery
DATABASE_URL=postgres://${USERNAME}:${PASSWORD}@${DOMAIN}:${PORT}/${DATABASE}
REDASH_COOKIE_SECRET=${YOUR_SECRET_TOKEN}
REDASH_SECRET_KEY=${YOUR_SECRET_KEY}
REDIS_URL=${YOUR_REDIS_URL}
REDIS_TLS_URL=${YOUR_REDIS_TLS_URL}
REDASH_LOG_LEVEL=INFO
```

환경변수는 아래와 같이 heroku config:set 을 사용하여 설정할 수 있습니다.  

> heroku config:set KEY=VALUE

생략된 환경변수는 [Redash > Open Source & Self Hosted > Environment Variables Settings](https://redash.io/help/open-source/admin-guide/env-vars-settings) 참조

## 배포 설정

Heroku에서 GitHub 저장소를 연결하고 `Automatic deploys`를 활성화하고 배포합니다.
> redash가 자동배포됩니다.

## DB 초기화

### 테이블 생성

배포 이후 redash 기본 구성을 위한 테이블 초기화 작업입니다.

```bash
heroku run /app/manage.py database create_tables
```

### 테이블 제거

테이블 생성 후 하지 않아도 되는 작업이지만
만약 테이블 정보를 날리고 다시 시작하고 싶다면 아래와 같이 입력합니다.

```bash
heroku run /app/manage.py database drop_tables
```

## 서비스 시작

Dyno를 실행하면 서비스가 실행됩니다.
