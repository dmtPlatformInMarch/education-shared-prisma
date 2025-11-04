# Education Shared Prisma

공유 Prisma 스키마 및 클라이언트를 제공하는 패키지입니다. 교육 관련 마이크로서비스들이 공통으로 사용하는 데이터베이스 스키마를 관리합니다.

## 📋 목차
- [설치](#설치)
- [설정](#설정)
- [사용법](#사용법)
- [스크립트 명령어](#스크립트-명령어)
- [모델 구조](#모델-구조)
- [개발 가이드](#개발-가이드)

## 🚀 설치

### 1. 의존성 설치
```bash
npm install
```

### 2. 환경 변수 설정
프로젝트 루트에 `.env.local` 과 `.env.dev` 파일을 생성합니다:

#### .env.local (로컬 개발용)
```env
DATABASE_URL="postgresql://username:password@localhost:5432/education"
```

#### .env.dev (개발 서버용)
```env
DATABASE_URL="postgresql://root:root@121.131.148.39:7003/education"
```

### 3. Prisma 클라이언트 생성
```bash
npm run prisma:generate
```

### 4. 데이터베이스 마이그레이션
```bash
# 개발 환경에서 마이그레이션 생성 및 적용
npm run migrate:dev

# 프로덕션 환경에서 마이그레이션 적용
npm run migrate:deploy
```

## 💻 사용법

### NestJS에서 사용하기

#### 1. 패키지 설치
다른 마이크로서비스에서 이 패키지를 사용하려면:

```bash
# npm link를 사용한 로컬 개발
cd education-shared-prisma
npm link

cd ../your-service
npm link shared-prisma
```

#### 2. Module Import
```typescript
import { Module } from '@nestjs/common';
import { PrismaModule } from 'shared-prisma';

@Module({
  imports: [PrismaModule],
  // ...
})
export class AppModule {}
```

#### 3. Service에서 사용
```typescript
import { Injectable } from '@nestjs/common';
import { PrismaService } from 'shared-prisma';

@Injectable()
export class UserService {
  constructor(private prisma: PrismaService) {}

  async findAllUsers() {
    return this.prisma.user.findMany();
  }

  async createUser(data: CreateUserDto) {
    return this.prisma.user.create({
      data,
    });
  }
}
```

## 📜 스크립트 명령어

| 명령어 | 설명 |
|--------|------|
| `npm run build` | TypeScript 컴파일 |
| `npm run prisma:generate` | Prisma 클라이언트 생성 |
| `npm run prisma:studio` | Prisma Studio 실행 (DB GUI) |
| `npm run migrate:dev` | 개발 환경 마이그레이션 생성/적용 |
| `npm run migrate:deploy` | 프로덕션 마이그레이션 적용 |
| `npm run migrate:reset` | DB 초기화 및 재생성 (주의!) |
| `npm run db:push` | 스키마 변경사항 DB에 직접 적용 |
| `npm run db:pull` | DB 스키마를 Prisma 스키마로 가져오기 |
| `npm run seed` | 시드 데이터 실행 |
| `npm run env:local` | 로컬 환경으로 전환 |
| `npm run env:dev` | 개발 환경으로 전환 |

## 🗄️ 모델 구조

### 주요 모델

#### User (사용자)
- 모든 사용자 정보 관리
- Role: STUDENT_CHILD, STUDENT, TEACHER, ADMIN, ADMIN_WAITING, PARENT, UNDEFINED
- Plan: FREE, STANDARD, PREMIUM

#### Organization (조직)
- 학교, 학원 등의 조직 정보
- Type: KINDERGARDEN, ELEMENTARY, MIDDLE, HIGH, INSTITUTE

#### Class (클래스)
- 조직 내의 클래스/반 정보
- 코드 기반 가입 시스템

#### Board & Post (게시판)
- 클래스별 게시판 시스템
- 권한 관리: OWNER, TEACHER, STUDENT, PARENT

#### TimeSlot (시간표/상담)
- 교사-학생 간 일정 관리
- 상태: CLOSED, PENDING, APPROVED, DECLINED, CANCELED, COMPLETED

### 관계 테이블
- UserRelation: 사용자 간 관계 (부모-자식 등)
- UserToOrganization: 사용자-조직 매핑
- ClassToUser: 클래스-사용자 매핑

## 🛠️ 개발 가이드

### 스키마 수정 워크플로우

1. **스키마 수정**
   ```bash
   # prisma/schema.prisma 파일 수정
   ```

2. **마이그레이션 생성**
   ```bash
   npm run migrate:dev -- --name your_migration_name
   ```

3. **클라이언트 재생성**
   ```bash
   npm run prisma:generate
   ```

4. **빌드**
   ```bash
   npm run build
   ```

5. **연결된 서비스에서 업데이트**
   ```bash
   # 각 마이크로서비스에서
   npm run build
   ```

### 환경별 작업

#### 로컬 개발
```bash
npm run env:local
npm run migrate:dev
npm run prisma:studio
```

#### 개발 서버 작업
```bash
npm run env:dev
npm run migrate:deploy
```

### 시드 데이터

`prisma/seed.ts` 파일을 생성하여 초기 데이터를 설정할 수 있습니다:

```typescript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

async function main() {
  // 시드 데이터 로직
  const user = await prisma.user.create({
    data: {
      email: 'admin@example.com',
      name: 'Admin User',
      role: 'ADMIN',
      // ...
    },
  });
  console.log({ user });
}

main()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

## ⚠️ 주의사항

1. **마이그레이션 관리**
   - 프로덕션에서는 항상 백업 후 마이그레이션 실행
   - `migrate:reset`은 모든 데이터를 삭제하므로 주의

2. **환경 변수**
   - `.env` 파일은 절대 Git에 커밋하지 않음
   - 환경별로 적절한 DATABASE_URL 사용

3. **타입 안정성**
   - 스키마 변경 후 반드시 `prisma:generate` 실행
   - TypeScript 빌드 확인

4. **연결된 서비스**
   - 스키마 변경 시 모든 마이크로서비스 재빌드 필요
   - Breaking change는 버전 관리 고려

## 📞 지원

문제가 발생하거나 질문이 있으면 이슈를 생성해주세요.