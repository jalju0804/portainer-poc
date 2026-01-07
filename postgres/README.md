## 공용 Postgres 스택 (`keycloak-postgres`)

여러 애플리케이션(예: Keycloak, 내부 서비스 등)이 공유해서 사용할 수 있는 공용 PostgreSQL 인스턴스를  
Portainer Stack (또는 Edge Stack GitOps)으로 배포하기 위한 최소 구성입니다.

### 폴더 구조

- `keycloak-postgres/`
  - `docker-compose.yml` : Portainer에서 참조할 메인 Compose 파일
  - `README.md` : 이 스택 전용 설명

> **권장**: 실제 Keycloak 애플리케이션 스택(`keycloak/`)은 별도 Compose 파일에서 관리하고, DB 연결 정보만 이 스택에 맞게 세팅합니다.

---

### Portainer에서 사용 방법

1. **Stacks > Add stack** (또는 Edge Stacks > Add stack) 이동  
2. **Build method**를 `Repository`로 선택  
3. **Repository URL**에 이 레포지토리 Git 주소 입력  
4. **Compose path**에 아래 경로 입력

   ```text
   keycloak-postgres/docker-compose.yml
   ```

5. **Environment variables**에서 다음 값을 설정
   - `PG_SUPERUSER_PASSWORD` : Postgres 슈퍼유저 비밀번호 (필수)
   - (옵션) `PG_SUPERUSER` : 기본값 `aolda`
   - (옵션) `PG_DEFAULT_DB` : 기본값 `aolda`

6. **GitOps updates**를 사용하는 경우 `Automatic updates`(polling) 활성화

---

### 애플리케이션에서 연결 예시 (개념)

- DB 호스트: `postgres` (서비스 이름)  
- DB 포트: `5432`  
- DB 이름/유저/패스워드:
  - 간단하게는 `PG_SUPERUSER`, `PG_SUPERUSER_PASSWORD`, `PG_DEFAULT_DB`를 그대로 사용하거나
  - 필요 시, 애플리케이션별로 별도 DB/계정을 생성하여 사용하는 것을 권장합니다.

> **주의**:  
> - 동일한 Ceph 경로(`/mnt/cephfs/prod/postgres/main_data`)를 사용하는 Postgres 컨테이너가 2개 이상 뜨지 않도록 반드시 Portainer에서 Stack 단위로만 관리하세요.  
> - 실제 Ceph 마운트 경로나 네트워크 이름은 운영 환경에 맞게 조정해야 합니다.

