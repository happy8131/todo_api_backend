# 🚀 Todo API ROADMAP - 풀스택 개발 가이드

## 프로젝트 개요

**목표**: Prisma + Express.js + MariaDB를 사용한 완전한 Todo API 개발

**기술 스택**:
- Frontend: React (나중) / Postman (지금은 API 테스트)
- Backend: Express.js + TypeScript
- Database: MariaDB + Prisma ORM
- Authentication: JWT
- Validation: Zod

**개발 시간**: 1주 (7-10일)

**평가 기준**:
- 기능 완성도 (50%)
- 코드 품질 (30%)
- API 문서화 (20%)

---

## Phase 1️⃣: 프로젝트 기초 설정 (1-2시간)

### Task 1-1: 프로젝트 구조 설계 (20분)
- [ ] 폴더 구조 생성:
  ```bash
  my-sql-backend/
  ├── src/
  │   ├── index.ts (서버 시작)
  │   ├── routes/ (API 라우트)
  │   ├── controllers/ (비즈니스 로직)
  │   ├── middlewares/ (미들웨어)
  │   ├── services/ (DB 쿼리)
  │   └── utils/ (헬퍼 함수)
  ├── prisma/
  │   └── schema.prisma (스키마)
  ├── .env (환경변수)
  ├── tsconfig.json
  ├── package.json
  └── .gitignore
  ```

### Task 1-2: TypeScript 설정 (15분)
- [ ] `tsconfig.json` 생성:
  ```json
  {
    "compilerOptions": {
      "target": "ES2020",
      "module": "commonjs",
      "lib": ["ES2020"],
      "outDir": "./dist",
      "rootDir": "./src",
      "strict": true,
      "esModuleInterop": true,
      "skipLibCheck": true,
      "forceConsistentCasingInFileNames": true
    },
    "include": ["src"],
    "exclude": ["node_modules", "dist"]
  }
  ```
- [ ] `package.json` scripts 추가:
  ```json
  "scripts": {
    "dev": "ts-node src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "prisma:migrate": "prisma migrate dev",
    "prisma:studio": "prisma studio"
  }
  ```

### Task 1-3: 필수 라이브러리 설치 (10분)
- [ ] 설치 완료 확인:
  ```bash
  npm list | grep -E "express|prisma|typescript"
  ```

### Task 1-4: 환경변수 설정 (10분)
- [ ] `.env` 파일 확인:
  ```
  DATABASE_URL="mysql://todouser:todo123@localhost:3306/todo_app"
  JWT_SECRET="your-super-secret-key-change-this"
  PORT=3000
  NODE_ENV=development
  ```

### Task 1-5: Prisma 스키마 확인 (15분)
- [ ] `prisma/schema.prisma` 확인:
  ```prisma
  generator client {
    provider = "prisma-client-js"
  }

  datasource db {
    provider = "mysql"
    url      = env("DATABASE_URL")
  }

  model User {
    id    Int     @id @default(autoincrement())
    name  String
    email String  @unique
    password String
    todos Todo[]
    createdAt DateTime @default(now())
    updatedAt DateTime @updatedAt
  }

  model Todo {
    id        Int     @id @default(autoincrement())
    title     String
    description String?
    completed Boolean @default(false)
    userId    Int
    user      User    @relation(fields: [userId], references: [id], onDelete: Cascade)
    createdAt DateTime @default(now())
    updatedAt DateTime @updatedAt
  }
  ```

### Task 1-6: Prisma Client 생성 (10분)
- [ ] 실행:
  ```bash
  npx prisma generate
  npx prisma db push
  ```
- [ ] 결과: `✔ Database synced`

### Task 1-7: 기본 서버 파일 생성 (15분)
- [ ] `src/index.ts` 생성:
  ```typescript
  import express from 'express'
  import { PrismaClient } from '@prisma/client'

  const app = express()
  const prisma = new PrismaClient()
  const PORT = process.env.PORT || 3000

  app.use(express.json())

  // 건강 체크
  app.get('/health', (req, res) => {
    res.json({ status: 'OK', timestamp: new Date() })
  })

  // 서버 시작
  app.listen(PORT, () => {
    console.log(`✅ Server running on port ${PORT}`)
  })
  ```

### Task 1-8: Git 초기화 및 커밋 (10분)
- [ ] `git init`
- [ ] `git add .`
- [ ] `git commit -m "feat: initial project setup with Prisma and Express"`

---

## Phase 2️⃣: 기본 API 엔드포인트 (2-3시간)

### Task 2-1: 유저 생성 API (30분)

#### 2-1-1: Service 레이어
- [ ] `src/services/userService.ts` 생성:
  ```typescript
  import { PrismaClient } from '@prisma/client'
  import bcrypt from 'bcrypt'

  const prisma = new PrismaClient()

  export const createUser = async (name: string, email: string, password: string) => {
    const hashedPassword = await bcrypt.hash(password, 10)
    
    return prisma.user.create({
      data: {
        name,
        email,
        password: hashedPassword
      }
    })
  }

  export const getUserByEmail = async (email: string) => {
    return prisma.user.findUnique({
      where: { email }
    })
  }
  ```

#### 2-1-2: Controller 레이어
- [ ] `src/controllers/userController.ts` 생성:
  ```typescript
  import { Request, Response } from 'express'
  import { createUser } from '../services/userService'

  export const register = async (req: Request, res: Response) => {
    try {
      const { name, email, password } = req.body

      if (!name || !email || !password) {
        return res.status(400).json({ error: 'Missing required fields' })
      }

      const user = await createUser(name, email, password)
      
      res.status(201).json({
        message: 'User created successfully',
        user: {
          id: user.id,
          name: user.name,
          email: user.email
        }
      })
    } catch (error) {
      res.status(500).json({ error: 'Failed to create user' })
    }
  }
  ```

#### 2-1-3: Route 등록
- [ ] `src/routes/userRoutes.ts` 생성:
  ```typescript
  import { Router } from 'express'
  import { register } from '../controllers/userController'

  const router = Router()

  router.post('/register', register)

  export default router
  ```

#### 2-1-4: 메인 서버에 연결
- [ ] `src/index.ts`에 추가:
  ```typescript
  import userRoutes from './routes/userRoutes'

  app.use('/api/users', userRoutes)
  ```

### Task 2-2: Todo 생성 API (30분)
- [ ] `src/services/todoService.ts`:
  ```typescript
  export const createTodo = async (title: string, userId: number, description?: string) => {
    return prisma.todo.create({
      data: {
        title,
        description,
        userId
      }
    })
  }
  ```

- [ ] `src/controllers/todoController.ts`:
  ```typescript
  export const createTodo = async (req: Request, res: Response) => {
    try {
      const { title, description, userId } = req.body

      if (!title || !userId) {
        return res.status(400).json({ error: 'Missing required fields' })
      }

      const todo = await createTodoService(title, userId, description)
      res.status(201).json(todo)
    } catch (error) {
      res.status(500).json({ error: 'Failed to create todo' })
    }
  }
  ```

- [ ] `src/routes/todoRoutes.ts` 생성 및 등록

### Task 2-3: Todo 조회 API (GET /todos) (30분)
- [ ] 모든 Todo 조회:
  ```typescript
  export const getAllTodos = async () => {
    return prisma.todo.findMany({
      include: { user: true },
      orderBy: { createdAt: 'desc' }
    })
  }
  ```

- [ ] 사용자별 Todo 조회:
  ```typescript
  export const getUserTodos = async (userId: number) => {
    return prisma.todo.findMany({
      where: { userId },
      orderBy: { createdAt: 'desc' }
    })
  }
  ```

### Task 2-4: Todo 수정 API (PUT /todos/:id) (30분)
- [ ] Service:
  ```typescript
  export const updateTodo = async (id: number, data: { title?: string; description?: string; completed?: boolean }) => {
    return prisma.todo.update({
      where: { id },
      data
    })
  }
  ```

### Task 2-5: Todo 삭제 API (DELETE /todos/:id) (20분)
- [ ] Service:
  ```typescript
  export const deleteTodo = async (id: number) => {
    return prisma.todo.delete({
      where: { id }
    })
  }
  ```

### Task 2-6: 에러 처리 미들웨어 (20분)
- [ ] `src/middlewares/errorHandler.ts`:
  ```typescript
  export const errorHandler = (err: any, req: Request, res: Response, next: Function) => {
    console.error(err)
    res.status(500).json({ error: 'Internal server error' })
  }
  ```

### Task 2-7: CORS 설정 (10분)
- [ ] `src/index.ts`에 추가:
  ```typescript
  import cors from 'cors'

  app.use(cors({
    origin: 'http://localhost:3000',
    credentials: true
  }))
  ```

### Task 2-8: 커밋
- [ ] `git commit -m "feat: implement basic CRUD APIs for Users and Todos"`

---

## Phase 3️⃣: JWT + Refresh Token 인증 시스템 (3-4시간)

### Task 3-1: Prisma 스키마에 RefreshToken 모델 추가 (15분)
- [ ] `prisma/schema.prisma` 수정:
  ```prisma
  model User {
    id    Int     @id @default(autoincrement())
    name  String
    email String  @unique
    password String
    refreshTokens RefreshToken[]
    todos Todo[]
    createdAt DateTime @default(now())
    updatedAt DateTime @updatedAt
  }

  model RefreshToken {
    id        Int     @id @default(autoincrement())
    token     String  @unique
    userId    Int
    user      User    @relation(fields: [userId], references: [id], onDelete: Cascade)
    expiresAt DateTime
    revoked   Boolean @default(false)
    createdAt DateTime @default(now())
  }

  model Todo {
    id        Int     @id @default(autoincrement())
    title     String
    description String?
    completed Boolean @default(false)
    userId    Int
    user      User    @relation(fields: [userId], references: [id], onDelete: Cascade)
    createdAt DateTime @default(now())
    updatedAt DateTime @updatedAt
  }
  ```

- [ ] 마이그레이션:
  ```bash
  npx prisma migrate dev --name add-refresh-token
  ```

### Task 3-2: bcrypt 비밀번호 해싱 (20분)
- [ ] 이미 Task 2-1에서 구현됨
- [ ] `npm install bcrypt @types/bcrypt`

### Task 3-3: JWT 유틸 (Access + Refresh Token) (40분)
- [ ] `npm install jsonwebtoken @types/jsonwebtoken`
- [ ] `src/utils/jwt.ts`:
  ```typescript
  import jwt from 'jsonwebtoken'

  export interface TokenPayload {
    userId: number
  }

  // Access Token (짧은 만료 시간: 15분)
  export const generateAccessToken = (userId: number): string => {
    return jwt.sign({ userId }, process.env.JWT_SECRET || '', {
      expiresIn: '15m'
    })
  }

  // Refresh Token (긴 만료 시간: 7일)
  export const generateRefreshToken = (userId: number): string => {
    return jwt.sign({ userId }, process.env.JWT_REFRESH_SECRET || '', {
      expiresIn: '7d'
    })
  }

  // Access Token 검증
  export const verifyAccessToken = (token: string): TokenPayload | null => {
    try {
      return jwt.verify(token, process.env.JWT_SECRET || '') as TokenPayload
    } catch (error) {
      return null
    }
  }

  // Refresh Token 검증
  export const verifyRefreshToken = (token: string): TokenPayload | null => {
    try {
      return jwt.verify(token, process.env.JWT_REFRESH_SECRET || '') as TokenPayload
    } catch (error) {
      return null
    }
  }
  ```

### Task 3-4: Refresh Token Service (35분)
- [ ] `src/services/tokenService.ts`:
  ```typescript
  import { PrismaClient } from '@prisma/client'
  import { generateRefreshToken, verifyRefreshToken } from '../utils/jwt'

  const prisma = new PrismaClient()

  // Refresh Token DB에 저장
  export const saveRefreshToken = async (userId: number, token: string) => {
    const expiresAt = new Date(Date.now() + 7 * 24 * 60 * 60 * 1000) // 7일

    return prisma.refreshToken.create({
      data: {
        token,
        userId,
        expiresAt
      }
    })
  }

  // 저장된 Refresh Token 검증
  export const validateRefreshToken = async (token: string) => {
    const refreshToken = await prisma.refreshToken.findUnique({
      where: { token },
      include: { user: true }
    })

    if (!refreshToken || refreshToken.revoked || new Date() > refreshToken.expiresAt) {
      return null
    }

    return refreshToken
  }

  // Refresh Token 무효화 (로그아웃)
  export const revokeRefreshToken = async (token: string) => {
    return prisma.refreshToken.update({
      where: { token },
      data: { revoked: true }
    })
  }

  // 사용자의 모든 토큰 무효화 (모든 기기 로그아웃)
  export const revokeAllUserTokens = async (userId: number) => {
    return prisma.refreshToken.updateMany({
      where: { userId },
      data: { revoked: true }
    })
  }

  // 만료된 토큰 삭제 (정리)
  export const deleteExpiredTokens = async () => {
    return prisma.refreshToken.deleteMany({
      where: {
        expiresAt: {
          lt: new Date()
        }
      }
    })
  }
  ```

### Task 3-5: 로그인 API (Access + Refresh Token 발급) (35분)
- [ ] `src/controllers/authController.ts`:
  ```typescript
  import { Request, Response } from 'express'
  import bcrypt from 'bcrypt'
  import { generateAccessToken, generateRefreshToken } from '../utils/jwt'
  import { saveRefreshToken } from '../services/tokenService'
  import { getUserByEmail } from '../services/userService'

  export const login = async (req: Request, res: Response) => {
    try {
      const { email, password } = req.body

      // 사용자 확인
      const user = await getUserByEmail(email)
      if (!user) {
        return res.status(401).json({ error: 'Invalid credentials' })
      }

      // 비밀번호 검증
      const isPasswordValid = await bcrypt.compare(password, user.password)
      if (!isPasswordValid) {
        return res.status(401).json({ error: 'Invalid credentials' })
      }

      // 토큰 생성
      const accessToken = generateAccessToken(user.id)
      const refreshToken = generateRefreshToken(user.id)

      // Refresh Token DB에 저장
      await saveRefreshToken(user.id, refreshToken)

      // Refresh Token을 HTTP-Only 쿠키로 저장 (보안)
      res.cookie('refreshToken', refreshToken, {
        httpOnly: true,
        secure: process.env.NODE_ENV === 'production',
        sameSite: 'strict',
        maxAge: 7 * 24 * 60 * 60 * 1000 // 7일
      })

      res.json({
        message: 'Login successful',
        accessToken,
        user: {
          id: user.id,
          name: user.name,
          email: user.email
        }
      })
    } catch (error) {
      console.error(error)
      res.status(500).json({ error: 'Login failed' })
    }
  }
  ```

### Task 3-6: Token Refresh API (35분)
- [ ] `src/controllers/authController.ts` 추가:
  ```typescript
  export const refreshAccessToken = async (req: Request, res: Response) => {
    try {
      // 쿠키에서 Refresh Token 가져오기
      const refreshToken = req.cookies.refreshToken

      if (!refreshToken) {
        return res.status(401).json({ error: 'No refresh token provided' })
      }

      // Refresh Token 검증
      const refreshTokenData = await validateRefreshToken(refreshToken)
      if (!refreshTokenData) {
        return res.status(401).json({ error: 'Invalid or expired refresh token' })
      }

      // 새로운 Access Token 발급
      const newAccessToken = generateAccessToken(refreshTokenData.userId)

      res.json({
        accessToken: newAccessToken
      })
    } catch (error) {
      res.status(500).json({ error: 'Token refresh failed' })
    }
  }
  ```

### Task 3-7: 로그아웃 API (20분)
- [ ] `src/controllers/authController.ts` 추가:
  ```typescript
  import { revokeRefreshToken } from '../services/tokenService'

  export const logout = async (req: Request, res: Response) => {
    try {
      const refreshToken = req.cookies.refreshToken

      if (refreshToken) {
        // Refresh Token 무효화
        await revokeRefreshToken(refreshToken)
      }

      // 쿠키 삭제
      res.clearCookie('refreshToken')

      res.json({ message: 'Logged out successfully' })
    } catch (error) {
      res.status(500).json({ error: 'Logout failed' })
    }
  }
  ```

### Task 3-8: Access Token 검증 미들웨어 (25분)
- [ ] `src/middlewares/authMiddleware.ts`:
  ```typescript
  import { Request, Response, NextFunction } from 'express'
  import { verifyAccessToken } from '../utils/jwt'

  export interface AuthRequest extends Request {
    userId?: number
  }

  export const authMiddleware = (req: AuthRequest, res: Response, next: NextFunction) => {
    try {
      // Authorization 헤더에서 토큰 추출 (Bearer 스킴)
      const authHeader = req.headers.authorization
      if (!authHeader || !authHeader.startsWith('Bearer ')) {
        return res.status(401).json({ error: 'No token provided' })
      }

      const token = authHeader.substring(7) // "Bearer " 제거
      const decoded = verifyAccessToken(token)

      if (!decoded) {
        return res.status(401).json({ error: 'Invalid or expired token' })
      }

      req.userId = decoded.userId
      next()
    } catch (error) {
      res.status(500).json({ error: 'Authentication failed' })
    }
  }
  ```

### Task 3-9: 인증 라우트 등록 (20분)
- [ ] `src/routes/authRoutes.ts` 생성:
  ```typescript
  import { Router } from 'express'
  import { login, refreshAccessToken, logout } from '../controllers/authController'

  const router = Router()

  router.post('/login', login)
  router.post('/refresh', refreshAccessToken)
  router.post('/logout', logout)

  export default router
  ```

- [ ] `src/index.ts`에 추가:
  ```typescript
  import authRoutes from './routes/authRoutes'
  import cookieParser from 'cookie-parser'

  app.use(cookieParser()) // 쿠키 파싱
  app.use('/api/auth', authRoutes)
  ```

### Task 3-10: .env 파일 업데이트 (10분)
- [ ] `.env`:
  ```
  JWT_SECRET=your-access-token-secret-key
  JWT_REFRESH_SECRET=your-refresh-token-secret-key
  ```

### Task 3-11: 보호된 라우트 업데이트 (15분)
- [ ] `src/routes/todoRoutes.ts`:
  ```typescript
  import { authMiddleware } from '../middlewares/authMiddleware'

  router.post('/todos', authMiddleware, createTodo)
  router.get('/todos', authMiddleware, getUserTodos)
  router.put('/todos/:id', authMiddleware, updateTodo)
  router.delete('/todos/:id', authMiddleware, deleteTodo)
  ```

### Task 3-12: 라이브러리 설치 (5분)
- [ ] `npm install cookie-parser @types/cookie-parser`

### Task 3-13: 커밋
- [ ] `git commit -m "feat: implement JWT with Refresh Token"`

---

## 📋 **인증 플로우**

```
1️⃣ 로그인
   POST /api/auth/login
   └─ Access Token (15분) + Refresh Token (7일, 쿠키)

2️⃣ API 요청
   GET /api/todos
   Authorization: Bearer <AccessToken>
   └─ 성공 → 데이터 반환

3️⃣ Access Token 만료 시
   POST /api/auth/refresh
   (쿠키에서 Refresh Token 자동)
   └─ 새로운 Access Token 발급

4️⃣ 로그아웃
   POST /api/auth/logout
   └─ Refresh Token 무효화 + 쿠키 삭제
```

---

## 🔐 **보안 특징**

```
✅ Access Token: 짧은 만료 시간 (15분)
   → 탈취되어도 위험 최소화

✅ Refresh Token: 긴 만료 시간 (7일)
   → 사용자 편의성

✅ HTTP-Only 쿠키: 자동 전송
   → XSS 공격 방지

✅ 토큰 DB 저장: 무효화 가능
   → 강제 로그아웃 가능

✅ 만료된 토큰 정리
   → 데이터베이스 관리
```

---

## 📝 **API 테스트 순서**

```
1. POST /api/users/register (사용자 생성)
2. POST /api/auth/login (로그인 → Access/Refresh Token)
3. GET /api/todos (Access Token으로 인증)
4. POST /api/auth/refresh (Access Token 갱신)
5. POST /api/auth/logout (로그아웃)
```

---

## Phase 4️⃣: 데이터 검증 및 최적화 (1-2시간)

### Task 4-1: Zod 검증 (30분)
- [ ] `npm install zod`
- [ ] `src/schemas/todoSchema.ts`:
  ```typescript
  import { z } from 'zod'

  export const createTodoSchema = z.object({
    title: z.string().min(1).max(255),
    description: z.string().optional(),
    userId: z.number()
  })

  export const updateTodoSchema = z.object({
    title: z.string().optional(),
    description: z.string().optional(),
    completed: z.boolean().optional()
  })
  ```

### Task 4-2: 검증 미들웨어 (20분)
- [ ] `src/middlewares/validateSchema.ts`:
  ```typescript
  export const validateSchema = (schema: z.ZodSchema) => 
    (req: Request, res: Response, next: Function) => {
      try {
        schema.parse(req.body)
        next()
      } catch (error) {
        res.status(400).json({ error: 'Validation failed' })
      }
    }
  ```

### Task 4-3: 쿼리 최적화 (20분)
- [ ] `select` 사용하여 필요한 필드만 조회:
  ```typescript
  export const getUserTodos = async (userId: number) => {
    return prisma.todo.findMany({
      where: { userId },
      select: {
        id: true,
        title: true,
        completed: true,
        createdAt: true
      },
      orderBy: { createdAt: 'desc' }
    })
  }
  ```

### Task 4-4: 페이지네이션 (25분)
- [ ] `src/utils/pagination.ts`:
  ```typescript
  export const getPagination = (page: number = 1, limit: number = 10) => {
    const skip = (page - 1) * limit
    return { skip, take: limit }
  }
  ```

### Task 4-5: 커밋
- [ ] `git commit -m "feat: add data validation with Zod and pagination"`

---

## Phase 5️⃣: API 문서화 및 테스트 (1-2시간)

### Task 5-1: Swagger/OpenAPI 설정 (30분)
- [ ] `npm install swagger-ui-express swagger-jsdoc`
- [ ] `src/swagger.ts`:
  ```typescript
  import swaggerUi from 'swagger-ui-express'
  import swaggerJsdoc from 'swagger-jsdoc'

  const options = {
    definition: {
      openapi: '3.0.0',
      info: {
        title: 'Todo API',
        version: '1.0.0'
      },
      servers: [
        {
          url: 'http://localhost:3000/api',
          description: 'Development'
        }
      ]
    },
    apis: ['./src/routes/*.ts']
  }

  export const swaggerSpec = swaggerJsdoc(options)
  export const swaggerUI = swaggerUi
  ```

### Task 5-2: API 엔드포인트 문서화 (30분)
- [ ] 각 라우트에 JSDoc 추가:
  ```typescript
  /**
   * @swagger
   * /users/register:
   *   post:
   *     summary: Register a new user
   *     requestBody:
   *       required: true
   *       content:
   *         application/json:
   *           schema:
   *             type: object
   *             properties:
   *               name: { type: string }
   *               email: { type: string }
   *               password: { type: string }
   *     responses:
   *       201: { description: 'User created' }
   *       400: { description: 'Invalid input' }
   */
  ```

### Task 5-3: Postman/Thunder Client로 테스트 (30분)
- [ ] 테스트할 API:
  - [ ] POST /users/register
  - [ ] POST /auth/login
  - [ ] POST /todos
  - [ ] GET /todos
  - [ ] PUT /todos/:id
  - [ ] DELETE /todos/:id

### Task 5-4: 환경별 설정 (20분)
- [ ] `.env.example` 생성:
  ```
  DATABASE_URL=mysql://user:password@localhost:3306/todo_app
  JWT_SECRET=your-secret-key
  PORT=3000
  NODE_ENV=development
  ```

### Task 5-5: README.md 작성 (20분)
- [ ] 프로젝트 설명
- [ ] 설치 방법
- [ ] API 엔드포인트 목록
- [ ] 예제

### Task 5-6: 커밋
- [ ] `git commit -m "feat: add API documentation with Swagger"`

---

## Phase 6️⃣: 배포 및 성능 최적화 (1시간)

### Task 6-1: 환경 최적화 (15분)
- [ ] Production 빌드:
  ```bash
  npm run build
  npm start
  ```

### Task 6-2: 로깅 추가 (20분)
- [ ] `npm install pino`
- [ ] `src/utils/logger.ts`:
  ```typescript
  import pino from 'pino'

  export const logger = pino({
    level: process.env.NODE_ENV === 'production' ? 'info' : 'debug'
  })
  ```

### Task 6-3: 속도 개선 (15min)
- [ ] 쿼리 N+1 문제 해결
- [ ] 인덱스 추가 (필요시)

### Task 6-4: 안전성 강화 (10분)
- [ ] Rate limiting 추가:
  ```typescript
  import rateLimit from 'express-rate-limit'

  const limiter = rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 100
  })

  app.use(limiter)
  ```

### Task 6-5: 커밋
- [ ] `git commit -m "feat: add logging and security improvements"`

---

## 🎯 API 엔드포인트 목록

### 사용자 관련
```
POST /api/users/register
  - 새 사용자 등록
  - Body: { name, email, password }

POST /api/auth/login
  - 로그인 (JWT 토큰 발급)
  - Body: { email, password }
  - Response: { token, user }
```

### Todo 관련 (모두 인증 필요)
```
POST /api/todos
  - Todo 생성
  - Body: { title, description?, userId }

GET /api/todos
  - 사용자의 모든 Todo 조회
  - Query: ?page=1&limit=10

GET /api/todos/:id
  - 특정 Todo 조회

PUT /api/todos/:id
  - Todo 수정
  - Body: { title?, description?, completed? }

DELETE /api/todos/:id
  - Todo 삭제
```

---

## 📋 최종 체크리스트

```
- [ ] 프로젝트 기초 설정 완료
- [ ] Express.js + TypeScript 설정
- [ ] Prisma ORM 연결
- [ ] 5가지 CRUD API 구현
- [ ] JWT 인증 시스템
- [ ] 데이터 검증 (Zod)
- [ ] API 문서화 (Swagger)
- [ ] Postman으로 테스트
- [ ] README.md 작성
- [ ] Git 히스토리 깔끔함
- [ ] Production 빌드 가능
```

---

## 🚀 시작하기

```bash
# 현재 상태
npm list
npm run dev

# 테스트
POST http://localhost:3000/api/users/register
{
  "name": "John",
  "email": "john@example.com",
  "password": "password123"
}
```

---

## 💡 각 Phase 완료 후

| Phase | 예상 시간 | 완료 신호 |
|-------|---------|---------|
| 1 | 1-2h | `npm run dev` 작동 |
| 2 | 2-3h | 5개 CRUD API 작동 |
| 3 | 2-3h | JWT 로그인 작동 |
| 4 | 1-2h | 검증 + 페이지네이션 |
| 5 | 1-2h | Swagger 문서 완성 |
| 6 | 1h | Production 빌드 |
| **합계** | **8-13h** | 풀스택 완성! |

---

**이제 시작해! 각 Phase마다 신호해줄래?** 🚀

`npm run dev` 먼저 실행해봐! 💪
