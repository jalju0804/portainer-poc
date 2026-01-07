## Keycloak 애플리케이션 스택 (`keycloak`)

Keycloak 자체를 Portainer Stack (또는 Edge Stack GitOps)으로 배포하기 위한 Compose 구성입니다.  
DB(PostgreSQL)는 별도의 공용 Postgres 스택(`keycloak-postgres`)을 사용한다고 가정합니다.

### 폴더 구조

- `keycloak/`
  - `docker-compose.yml` : Portainer에서 참조할 메인 Compose 파일
  - `README.md` : Keycloak 스택 전용 설명
  - `realms/` : realm / 그룹 / 클라이언트 등을 JSON으로 정의하는 디렉터리 (코드 관리용)

---

### Portainer에서 사용 방법

1. **Stacks > Add stack** (또는 Edge Stacks > Add stack) 이동  
2. **Build method**를 `Repository`로 선택  
3. **Repository URL**에 이 레포지토리 Git 주소 입력  
4. **Compose path**에 아래 경로 입력

   ```text
   keycloak/docker-compose.yml
   ```

5. **Environment variables**에서 다음 값을 설정
   - (옵션) `KC_DB_NAME` : 기본값 `keycloak`
   - (옵션) `KC_DB_USER` : 기본값 `keycloak`
   - `KC_DB_PASSWORD` : Keycloak가 사용할 DB 계정의 비밀번호
   - (옵션) `KC_ADMIN_USER` : 기본값 `admin`
   - `KC_ADMIN_PASSWORD` : Keycloak 관리자 비밀번호 (필수)

6. **GitOps updates**를 사용하는 경우 `Automatic updates`(polling) 활성화

---

### Realm / 그룹을 JSON으로 코드 관리하는 방식

- `keycloak/realms/` 아래의 모든 `*.json` 파일을 Keycloak 부팅 시 `--import-realm` 옵션으로 자동 import 합니다.
- 예시:

  - `keycloak/realms/realm-aolda.json`

- 새 그룹/realm/클라이언트 추가 방법:
  1. `keycloak/realms/`에 JSON 파일을 수정 또는 추가
  2. Git 커밋 & 푸시
  3. Portainer Edge Stack / Stack이 GitOps로 자동 업데이트 → Keycloak 재시작 시 변경 내용 반영

> **주의**:  
> - `--import-realm`는 기본적으로 **존재하지 않는 realm은 생성**, 이미 있는 realm은 설정에 따라 병합/무시될 수 있습니다.  
> - 운영 환경에서는 수동 UI 변경보다는 JSON을 수정하고 다시 import하는 방식을 표준으로 가져가는 것을 권장합니다.

---

### 네트워크 및 Postgres 연동

- 이 스택에서는 공용 Postgres 스택(`keycloak-postgres`)이 미리 생성해 둔 네트워크를 **external 네트워크**로 재사용합니다.
- `keycloak/docker-compose.yml`의 네트워크 정의:

```yaml
networks:
  aolda-postgres-net:
    external: true
```

따라서 아래 조건을 만족해야 합니다.
  - 공용 Postgres 스택(`keycloak-postgres`)이 먼저 배포되어 있어야 하며
  - 해당 스택의 네트워크 이름이 `aolda-postgres-net` 이어야 함

Keycloak 컨테이너는 다음과 같은 방식으로 DB에 접속합니다.

- DB 호스트: `postgres` (공용 Postgres 서비스 이름)
- DB 포트: `5432`
- DB 이름/유저/패스워드: Keycloak 전용 DB/계정(`KC_DB_NAME`, `KC_DB_USER`, `KC_DB_PASSWORD`)을 따로 만들고 사용

---

### 운영 시 주의사항

- 공용 Postgres 스택(`keycloak-postgres`)은 여러 애플리케이션이 공유할 수 있으므로, Keycloak용 별도 DB/계정을 만들어 사용하는 것을 권장합니다.
- Keycloak 포트를 기본값인 `8080`으로 노출하고 있으므로, 같은 노드에 다른 웹 서비스가 있다면 포트 충돌 여부를 확인하십시오.
- 인증/인가 서비스 특성상, Keycloak는 반드시 HTTPS/TLS 종단(역프록시 등) 뒤에 두는 것을 권장합니다. 이 Compose는 최소 구성만 다루며, TLS/리버스 프록시는 별도 인프라 레이어에서 처리하는 시나리오를 가정합니다.


