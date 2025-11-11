# Build and Deploy to Target Server

## 📌 목적

프로젝트를 빌드하고 Docker 이미지를 생성한 후 타겟 서버로 전송합니다.

**입력**: `${1}` (project-name: admin, file, api, auth)
**출력**: 타겟 서버의 `~/transfer/{project-name}.tar` 파일

## 🚀 실행 단계

### 1. Docker 구동 확인
Docker daemon이 실행 중인지 확인합니다.

### 2. 프로젝트 폴더 이동
project-name에 따라 해당 프로젝트 폴더로 이동합니다:
- `admin` → `/Users/kyle/source/01.java/chalk_admin`
- `file` → `/Users/kyle/source/01.java/chalk_file`
- `api` → `/Users/kyle/source/01.java/chalk_api`
- `auth` → `/Users/kyle/source/01.java/chalk_auth`

### 3. 소스 빌드
```bash
./gradlew build -x test
```

### 4. Docker 이미지 빌드
빌드가 성공하면 Docker 이미지를 생성합니다:
```bash
docker build --platform linux/amd64 \
  --build-arg SPRING_PROFILES_ACTIVE=compose \
  --build-arg JAVA_OPTS='-XX:+UseG1GC -Xms512m -Xmx512m' \
  -t local-{project-name}:latest . \
  -f Dockerfile
```

### 5. Docker 이미지를 파일로 저장
```bash
docker save local-{project-name}:latest -o /Users/kyle/source/10.build/{project-name}.tar
```

### 6. 타겟 서버로 전송
```bash
scp -i /Users/kyle/source/credential/dev/dev-an2-ec2-keypare.pem \
  /Users/kyle/source/10.build/{project-name}.tar \
  ubuntu@ec2-13-209-76-128.ap-northeast-2.compute.amazonaws.com:~/transfer/{project-name}.tar
```

## 📋 주의사항

- 각 단계가 성공해야 다음 단계로 진행합니다
- 실패 시 명확한 오류 메시지를 출력합니다
- project-name은 반드시 admin, file, api, auth 중 하나여야 합니다

---

시작하겠습니다! 🚀