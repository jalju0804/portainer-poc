## Keycloak Custom SPI 사용 가이드

이 디렉터리는 Keycloak Custom SPI(JAR)를 배포하기 위한 위치 예시입니다.

### 1. SPI 프로젝트 구조(예시)

- `keycloak/`
  - `custom-spi/`
    - `build/libs/custom-spi-1.0.0.jar`  ← Gradle 빌드 결과물이 생성되는 위치

실제 SPI 구현(Java 코드)은 별도 레포지토리이거나, 이 디렉터리 안에 Gradle 프로젝트로 두는 방식을 권장합니다.

### 2. 빌드 & 배포 절차

1. SPI 프로젝트에서 Gradle 빌드 실행

   ```bash
   cd keycloak/custom-spi
   ./gradlew build
   ```

   빌드 후 `build/libs/custom-spi-1.0.0.jar` 가 생성되었다고 가정합니다.

2. `keycloak/docker-compose.yml`에서 다음 볼륨 매핑 주석을 해제

   ```yaml
   volumes:
     - ./realms:/opt/keycloak/data/import:ro
     # - ./custom-spi/build/libs/custom-spi-1.0.0.jar:/opt/keycloak/providers/custom-spi.jar:ro
   ```

3. Portainer Stack(또는 Edge Stack)을 재배포하면 Keycloak이 JAR를 로드하여 SPI를 활성화합니다.

### 3. 주의사항

- JAR 파일명(`custom-spi-1.0.0.jar`)이 변경되면 `docker-compose.yml`의 경로도 같이 수정해야 합니다.
- SPI 변경 시마다 **빌드 → Git 커밋 → Stack 재배포** 순서로 진행하면, Keycloak 설정/코드가 전부 GitOps로 관리됩니다.


