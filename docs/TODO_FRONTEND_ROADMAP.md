# 🎨 Todo App 프론트엔드 ROADMAP - React + TypeScript

## 프로젝트 개요

**목표**: 백엔드 Todo API와 연결된 완전한 Todo 웹앱 개발

**기술 스택**:
- Frontend: React 18 + TypeScript
- State Management: Zustand
- HTTP Client: Axios
- Styling: Tailwind CSS
- UI Components: Shadcn/ui (선택)
- Authentication: JWT (로컬스토리지 + HTTP-Only 쿠키)
- Routing: React Router v6

**개발 시간**: 1주 (7-10일)

**평가 기준**:
- UI/UX (40%)
- 기능 완성도 (40%)
- 코드 품질 (20%)

---

## Phase 1️⃣: 프로젝트 기초 설정 (1-2시간)

### Task 1-1: React 프로젝트 생성 (15분)
- [ ] Create React App with TypeScript:
  ```bash
  npx create-react-app todo-app --template typescript
  cd todo-app
  npm install
  ```

### Task 1-2: 필수 라이브러리 설치 (20분)
- [ ] 설치:
  ```bash
  npm install axios zustand react-router-dom
  npm install -D tailwindcss postcss autoprefixer
  npm install @headlessui/react @heroicons/react
  ```

### Task 1-3: Tailwind CSS 설정 (15분)
- [ ] 초기화:
  ```bash
  npx tailwindcss init -p
  ```

- [ ] `tailwind.config.js`:
  ```javascript
  module.exports = {
    content: [
      "./index.html",
      "./src/**/*.{js,ts,jsx,tsx}",
    ],
    theme: {
      extend: {},
    },
    plugins: [],
  }
  ```

- [ ] `src/index.css`:
  ```css
  @tailwind base;
  @tailwind components;
  @tailwind utilities;

  body {
    @apply bg-gray-50 font-sans;
  }
  ```

### Task 1-4: 폴더 구조 설계 (20분)
- [ ] 생성:
  ```
  src/
  ├── components/
  │   ├── layout/
  │   │   ├── Header.tsx
  │   │   ├── Sidebar.tsx
  │   │   └── MainLayout.tsx
  │   ├── auth/
  │   │   ├── LoginForm.tsx
  │   │   ├── RegisterForm.tsx
  │   │   └── ProtectedRoute.tsx
  │   └── todo/
  │       ├── TodoList.tsx
  │       ├── TodoItem.tsx
  │       ├── TodoForm.tsx
  │       └── TodoCard.tsx
  ├── pages/
  │   ├── LoginPage.tsx
  │   ├── RegisterPage.tsx
  │   ├── TodoPage.tsx
  │   └── NotFound.tsx
  ├── services/
  │   ├── api.ts
  │   └── authService.ts
  ├── store/
  │   ├── authStore.ts
  │   └── todoStore.ts
  ├── types/
  │   ├── auth.ts
  │   └── todo.ts
  ├── utils/
  │   ├── constants.ts
  │   └── localStorage.ts
  ├── App.tsx
  ├── index.css
  └── index.tsx
  ```

### Task 1-5: 타입 정의 (20분)
- [ ] `src/types/auth.ts`:
  ```typescript
  export interface User {
    id: number
    name: string
    email: string
  }

  export interface AuthState {
    user: User | null
    accessToken: string | null
    isLoading: boolean
    error: string | null
    login: (email: string, password: string) => Promise<void>
    register: (name: string, email: string, password: string) => Promise<void>
    logout: () => void
  }
  ```

- [ ] `src/types/todo.ts`:
  ```typescript
  export interface Todo {
    id: number
    title: string
    description?: string
    completed: boolean
    userId: number
    createdAt: string
    updatedAt: string
  }

  export interface TodoStore {
    todos: Todo[]
    isLoading: boolean
    error: string | null
    fetchTodos: () => Promise<void>
    createTodo: (title: string, description?: string) => Promise<void>
    updateTodo: (id: number, data: Partial<Todo>) => Promise<void>
    deleteTodo: (id: number) => Promise<void>
  }
  ```

### Task 1-6: API 클라이언트 설정 (20min)
- [ ] `src/services/api.ts`:
  ```typescript
  import axios, { AxiosInstance } from 'axios'

  const API_BASE_URL = process.env.REACT_APP_API_URL || 'http://localhost:3000/api'

  export const apiClient: AxiosInstance = axios.create({
    baseURL: API_BASE_URL,
    withCredentials: true // 쿠키 포함 (Refresh Token)
  })

  // Access Token을 Authorization 헤더에 추가
  apiClient.interceptors.request.use((config) => {
    const token = localStorage.getItem('accessToken')
    if (token) {
      config.headers.Authorization = `Bearer ${token}`
    }
    return config
  })

  // Access Token 만료 시 자동 갱신
  apiClient.interceptors.response.use(
    (response) => response,
    async (error) => {
      const originalRequest = error.config

      if (error.response?.status === 401 && !originalRequest._retry) {
        originalRequest._retry = true
        try {
          const response = await axios.post(`${API_BASE_URL}/auth/refresh`, {}, {
            withCredentials: true
          })
          const { accessToken } = response.data
          localStorage.setItem('accessToken', accessToken)
          originalRequest.headers.Authorization = `Bearer ${accessToken}`
          return apiClient(originalRequest)
        } catch (refreshError) {
          localStorage.removeItem('accessToken')
          window.location.href = '/login'
        }
      }

      return Promise.reject(error)
    }
  )

  export default apiClient
  ```

### Task 1-7: .env 설정 (10분)
- [ ] `.env`:
  ```
  REACT_APP_API_URL=http://localhost:3000/api
  REACT_APP_NODE_ENV=development
  ```

### Task 1-8: 커밋
- [ ] `git init`
- [ ] `git add .`
- [ ] `git commit -m "feat: initial React project setup with Tailwind CSS"`

---

## Phase 2️⃣: 인증 시스템 (2-3시간)

### Task 2-1: 인증 Zustand Store (35min)
- [ ] `src/store/authStore.ts`:
  ```typescript
  import { create } from 'zustand'
  import { AuthState, User } from '../types/auth'
  import apiClient from '../services/api'

  export const useAuthStore = create<AuthState>((set) => ({
    user: null,
    accessToken: localStorage.getItem('accessToken'),
    isLoading: false,
    error: null,

    login: async (email: string, password: string) => {
      set({ isLoading: true, error: null })
      try {
        const response = await apiClient.post('/auth/login', { email, password })
        const { accessToken, user } = response.data
        
        localStorage.setItem('accessToken', accessToken)
        set({ accessToken, user, isLoading: false })
      } catch (error: any) {
        set({ error: error.response?.data?.error || 'Login failed', isLoading: false })
        throw error
      }
    },

    register: async (name: string, email: string, password: string) => {
      set({ isLoading: true, error: null })
      try {
        await apiClient.post('/users/register', { name, email, password })
        set({ isLoading: false })
      } catch (error: any) {
        set({ error: error.response?.data?.error || 'Registration failed', isLoading: false })
        throw error
      }
    },

    logout: () => {
      localStorage.removeItem('accessToken')
      set({ user: null, accessToken: null })
    }
  }))
  ```

### Task 2-2: 로그인 페이지 (40분)
- [ ] `src/pages/LoginPage.tsx`:
  ```typescript
  import { useState } from 'react'
  import { useNavigate } from 'react-router-dom'
  import { useAuthStore } from '../store/authStore'
  import LoginForm from '../components/auth/LoginForm'

  export default function LoginPage() {
    const navigate = useNavigate()
    const login = useAuthStore((state) => state.login)
    const [error, setError] = useState('')

    const handleLogin = async (email: string, password: string) => {
      try {
        await login(email, password)
        navigate('/todos')
      } catch (err) {
        setError('Login failed')
      }
    }

    return (
      <div className="min-h-screen flex items-center justify-center bg-gray-50">
        <div className="bg-white p-8 rounded-lg shadow-md w-full max-w-md">
          <h1 className="text-2xl font-bold mb-6">Login</h1>
          {error && <div className="bg-red-100 text-red-700 p-3 rounded mb-4">{error}</div>}
          <LoginForm onSubmit={handleLogin} />
          <p className="mt-4 text-center text-gray-600">
            Don't have an account? <a href="/register" className="text-blue-600">Register</a>
          </p>
        </div>
      </div>
    )
  }
  ```

- [ ] `src/components/auth/LoginForm.tsx`:
  ```typescript
  import { useState } from 'react'

  interface LoginFormProps {
    onSubmit: (email: string, password: string) => Promise<void>
  }

  export default function LoginForm({ onSubmit }: LoginFormProps) {
    const [email, setEmail] = useState('')
    const [password, setPassword] = useState('')
    const [loading, setLoading] = useState(false)

    const handleSubmit = async (e: React.FormEvent) => {
      e.preventDefault()
      setLoading(true)
      try {
        await onSubmit(email, password)
      } finally {
        setLoading(false)
      }
    }

    return (
      <form onSubmit={handleSubmit} className="space-y-4">
        <div>
          <label className="block text-sm font-medium mb-1">Email</label>
          <input
            type="email"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
            className="w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
            required
          />
        </div>
        <div>
          <label className="block text-sm font-medium mb-1">Password</label>
          <input
            type="password"
            value={password}
            onChange={(e) => setPassword(e.target.value)}
            className="w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
            required
          />
        </div>
        <button
          type="submit"
          disabled={loading}
          className="w-full bg-blue-600 text-white py-2 rounded-lg hover:bg-blue-700 disabled:opacity-50"
        >
          {loading ? 'Logging in...' : 'Login'}
        </button>
      </form>
    )
  }
  ```

### Task 2-3: 회원가입 페이지 (35min)
- [ ] `src/pages/RegisterPage.tsx`:
  ```typescript
  import { useState } from 'react'
  import { useNavigate } from 'react-router-dom'
  import { useAuthStore } from '../store/authStore'
  import RegisterForm from '../components/auth/RegisterForm'

  export default function RegisterPage() {
    const navigate = useNavigate()
    const register = useAuthStore((state) => state.register)
    const [error, setError] = useState('')

    const handleRegister = async (name: string, email: string, password: string) => {
      try {
        await register(name, email, password)
        navigate('/login')
      } catch (err) {
        setError('Registration failed')
      }
    }

    return (
      <div className="min-h-screen flex items-center justify-center bg-gray-50">
        <div className="bg-white p-8 rounded-lg shadow-md w-full max-w-md">
          <h1 className="text-2xl font-bold mb-6">Register</h1>
          {error && <div className="bg-red-100 text-red-700 p-3 rounded mb-4">{error}</div>}
          <RegisterForm onSubmit={handleRegister} />
          <p className="mt-4 text-center text-gray-600">
            Already have an account? <a href="/login" className="text-blue-600">Login</a>
          </p>
        </div>
      </div>
    )
  }
  ```

### Task 2-4: 보호된 라우트 (25min)
- [ ] `src/components/auth/ProtectedRoute.tsx`:
  ```typescript
  import { Navigate } from 'react-router-dom'
  import { useAuthStore } from '../../store/authStore'

  interface ProtectedRouteProps {
    children: React.ReactNode
  }

  export default function ProtectedRoute({ children }: ProtectedRouteProps) {
    const accessToken = useAuthStore((state) => state.accessToken)

    if (!accessToken) {
      return <Navigate to="/login" replace />
    }

    return <>{children}</>
  }
  ```

### Task 2-5: 라우팅 설정 (25min)
- [ ] `src/App.tsx`:
  ```typescript
  import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom'
  import LoginPage from './pages/LoginPage'
  import RegisterPage from './pages/RegisterPage'
  import TodoPage from './pages/TodoPage'
  import ProtectedRoute from './components/auth/ProtectedRoute'

  export default function App() {
    return (
      <BrowserRouter>
        <Routes>
          <Route path="/login" element={<LoginPage />} />
          <Route path="/register" element={<RegisterPage />} />
          <Route
            path="/todos"
            element={
              <ProtectedRoute>
                <TodoPage />
              </ProtectedRoute>
            }
          />
          <Route path="/" element={<Navigate to="/todos" replace />} />
        </Routes>
      </BrowserRouter>
    )
  }
  ```

### Task 2-6: 커밋
- [ ] `git commit -m "feat: implement authentication with login/register"`

---

## Phase 3️⃣: Todo Zustand Store (1-1.5시간)

### Task 3-1: Todo Store 생성 (35min)
- [ ] `src/store/todoStore.ts`:
  ```typescript
  import { create } from 'zustand'
  import { TodoStore, Todo } from '../types/todo'
  import apiClient from '../services/api'

  export const useTodoStore = create<TodoStore>((set) => ({
    todos: [],
    isLoading: false,
    error: null,

    fetchTodos: async () => {
      set({ isLoading: true, error: null })
      try {
        const response = await apiClient.get('/todos')
        set({ todos: response.data, isLoading: false })
      } catch (error: any) {
        set({ error: error.response?.data?.error || 'Failed to fetch', isLoading: false })
      }
    },

    createTodo: async (title: string, description?: string) => {
      set({ isLoading: true, error: null })
      try {
        const response = await apiClient.post('/todos', { title, description })
        set((state) => ({
          todos: [response.data, ...state.todos],
          isLoading: false
        }))
      } catch (error: any) {
        set({ error: error.response?.data?.error || 'Failed to create', isLoading: false })
      }
    },

    updateTodo: async (id: number, data: Partial<Todo>) => {
      set({ isLoading: true, error: null })
      try {
        const response = await apiClient.put(`/todos/${id}`, data)
        set((state) => ({
          todos: state.todos.map((todo) => (todo.id === id ? response.data : todo)),
          isLoading: false
        }))
      } catch (error: any) {
        set({ error: error.response?.data?.error || 'Failed to update', isLoading: false })
      }
    },

    deleteTodo: async (id: number) => {
      set({ isLoading: true, error: null })
      try {
        await apiClient.delete(`/todos/${id}`)
        set((state) => ({
          todos: state.todos.filter((todo) => todo.id !== id),
          isLoading: false
        }))
      } catch (error: any) {
        set({ error: error.response?.data?.error || 'Failed to delete', isLoading: false })
      }
    }
  }))
  ```

### Task 3-2: useEffect로 초기 데이터 로드 (20min)
- [ ] `src/pages/TodoPage.tsx`:
  ```typescript
  import { useEffect } from 'react'
  import { useTodoStore } from '../store/todoStore'

  export default function TodoPage() {
    const fetchTodos = useTodoStore((state) => state.fetchTodos)

    useEffect(() => {
      fetchTodos()
    }, [])

    return <div>{/* Todo 컴포넌트들 */}</div>
  }
  ```

### Task 3-3: 커밋
- [ ] `git commit -m "feat: implement Zustand store for todos"`

---

## Phase 4️⃣: Todo UI 컴포넌트 (2-3시간)

### Task 4-1: Todo 목록 컴포넌트 (35min)
- [ ] `src/components/todo/TodoList.tsx`:
  ```typescript
  import { useTodoStore } from '../../store/todoStore'
  import TodoItem from './TodoItem'

  export default function TodoList() {
    const todos = useTodoStore((state) => state.todos)
    const isLoading = useTodoStore((state) => state.isLoading)

    if (isLoading) return <div className="text-center py-8">Loading...</div>

    if (todos.length === 0) {
      return (
        <div className="text-center py-12 bg-gray-50 rounded-lg">
          <p className="text-gray-500">No todos yet. Create one to get started!</p>
        </div>
      )
    }

    return (
      <div className="space-y-2">
        {todos.map((todo) => (
          <TodoItem key={todo.id} todo={todo} />
        ))}
      </div>
    )
  }
  ```

### Task 4-2: Todo 항목 컴포넌트 (30min)
- [ ] `src/components/todo/TodoItem.tsx`:
  ```typescript
  import { Todo } from '../../types/todo'
  import { useTodoStore } from '../../store/todoStore'

  interface TodoItemProps {
    todo: Todo
  }

  export default function TodoItem({ todo }: TodoItemProps) {
    const updateTodo = useTodoStore((state) => state.updateTodo)
    const deleteTodo = useTodoStore((state) => state.deleteTodo)

    return (
      <div className="bg-white p-4 rounded-lg border flex items-center justify-between hover:shadow-md transition">
        <div className="flex items-center gap-3 flex-1">
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() => updateTodo(todo.id, { completed: !todo.completed })}
            className="w-5 h-5 text-blue-600"
          />
          <div>
            <p className={`font-medium ${todo.completed ? 'line-through text-gray-400' : ''}`}>
              {todo.title}
            </p>
            {todo.description && <p className="text-sm text-gray-500">{todo.description}</p>}
          </div>
        </div>
        <button
          onClick={() => deleteTodo(todo.id)}
          className="text-red-600 hover:text-red-700 px-3 py-1 text-sm"
        >
          Delete
        </button>
      </div>
    )
  }
  ```

### Task 4-3: Todo 생성 폼 (40min)
- [ ] `src/components/todo/TodoForm.tsx`:
  ```typescript
  import { useState } from 'react'
  import { useTodoStore } from '../../store/todoStore'

  export default function TodoForm() {
    const [title, setTitle] = useState('')
    const [description, setDescription] = useState('')
    const [loading, setLoading] = useState(false)
    const createTodo = useTodoStore((state) => state.createTodo)

    const handleSubmit = async (e: React.FormEvent) => {
      e.preventDefault()
      if (!title.trim()) return

      setLoading(true)
      try {
        await createTodo(title, description)
        setTitle('')
        setDescription('')
      } finally {
        setLoading(false)
      }
    }

    return (
      <form onSubmit={handleSubmit} className="bg-white p-4 rounded-lg border mb-6">
        <input
          type="text"
          value={title}
          onChange={(e) => setTitle(e.target.value)}
          placeholder="Add a new todo..."
          className="w-full px-4 py-2 border rounded-lg mb-3 focus:outline-none focus:ring-2 focus:ring-blue-500"
        />
        <textarea
          value={description}
          onChange={(e) => setDescription(e.target.value)}
          placeholder="Add a description (optional)"
          className="w-full px-4 py-2 border rounded-lg mb-3 focus:outline-none focus:ring-2 focus:ring-blue-500"
          rows={2}
        />
        <button
          type="submit"
          disabled={loading || !title.trim()}
          className="w-full bg-blue-600 text-white py-2 rounded-lg hover:bg-blue-700 disabled:opacity-50"
        >
          {loading ? 'Creating...' : 'Add Todo'}
        </button>
      </form>
    )
  }
  ```

### Task 4-4: Todo 페이지 통합 (30min)
- [ ] `src/pages/TodoPage.tsx` 업데이트:
  ```typescript
  import { useEffect } from 'react'
  import { useTodoStore } from '../store/todoStore'
  import { useAuthStore } from '../store/authStore'
  import MainLayout from '../components/layout/MainLayout'
  import TodoForm from '../components/todo/TodoForm'
  import TodoList from '../components/todo/TodoList'

  export default function TodoPage() {
    const fetchTodos = useTodoStore((state) => state.fetchTodos)
    const user = useAuthStore((state) => state.user)

    useEffect(() => {
      fetchTodos()
    }, [])

    return (
      <MainLayout>
        <div className="max-w-2xl mx-auto py-8">
          <h1 className="text-3xl font-bold mb-2">My Todos</h1>
          <p className="text-gray-600 mb-6">Welcome, {user?.name}!</p>
          <TodoForm />
          <TodoList />
        </div>
      </MainLayout>
    )
  }
  ```

### Task 4-5: 커밋
- [ ] `git commit -m "feat: implement Todo CRUD UI with React components"`

---

## Phase 5️⃣: 레이아웃 및 네비게이션 (1-1.5시간)

### Task 5-1: Header 컴포넌트 (25min)
- [ ] `src/components/layout/Header.tsx`:
  ```typescript
  import { useNavigate } from 'react-router-dom'
  import { useAuthStore } from '../../store/authStore'

  export default function Header() {
    const navigate = useNavigate()
    const user = useAuthStore((state) => state.user)
    const logout = useAuthStore((state) => state.logout)

    const handleLogout = () => {
      logout()
      navigate('/login')
    }

    return (
      <header className="bg-white border-b">
        <div className="max-w-7xl mx-auto px-4 py-4 flex justify-between items-center">
          <h1 className="text-2xl font-bold text-blue-600">Todo App</h1>
          <div className="flex items-center gap-4">
            <span className="text-gray-600">{user?.email}</span>
            <button
              onClick={handleLogout}
              className="bg-red-600 text-white px-4 py-2 rounded-lg hover:bg-red-700"
            >
              Logout
            </button>
          </div>
        </div>
      </header>
    )
  }
  ```

### Task 5-2: MainLayout (25min)
- [ ] `src/components/layout/MainLayout.tsx`:
  ```typescript
  import Header from './Header'

  interface MainLayoutProps {
    children: React.ReactNode
  }

  export default function MainLayout({ children }: MainLayoutProps) {
    return (
      <div className="min-h-screen bg-gray-50">
        <Header />
        <main>{children}</main>
      </div>
    )
  }
  ```

### Task 5-3: 커밋
- [ ] `git commit -m "feat: implement layout with header and navigation"`

---

## Phase 6️⃣: 에러 처리 및 로딩 상태 (1-1.5시간)

### Task 6-1: 에러 바운더리 (25min)
- [ ] `src/components/ErrorBoundary.tsx`:
  ```typescript
  import { Component, ReactNode } from 'react'

  interface Props {
    children: ReactNode
  }

  interface State {
    hasError: boolean
  }

  export default class ErrorBoundary extends Component<Props, State> {
    constructor(props: Props) {
      super(props)
      this.state = { hasError: false }
    }

    static getDerivedStateFromError() {
      return { hasError: true }
    }

    render() {
      if (this.state.hasError) {
        return (
          <div className="min-h-screen flex items-center justify-center bg-red-50">
            <div className="bg-white p-8 rounded-lg shadow-md">
              <h1 className="text-2xl font-bold text-red-600 mb-4">Oops!</h1>
              <p className="text-gray-600">Something went wrong. Please refresh the page.</p>
            </div>
          </div>
        )
      }

      return this.props.children
    }
  }
  ```

### Task 6-2: 로딩 스켈레톤 (25min)
- [ ] `src/components/Skeleton.tsx`:
  ```typescript
  export default function Skeleton() {
    return (
      <div className="bg-white p-4 rounded-lg border animate-pulse">
        <div className="h-4 bg-gray-300 rounded mb-3 w-3/4"></div>
        <div className="h-3 bg-gray-300 rounded w-1/2"></div>
      </div>
    )
  }
  ```

### Task 6-3: 토스트 알림 (25min)
- [ ] `src/components/Toast.tsx`:
  ```typescript
  interface ToastProps {
    message: string
    type: 'success' | 'error' | 'info'
    onClose: () => void
  }

  export default function Toast({ message, type, onClose }: ToastProps) {
    const bgColor = type === 'success' ? 'bg-green-100' : type === 'error' ? 'bg-red-100' : 'bg-blue-100'
    const textColor = type === 'success' ? 'text-green-700' : type === 'error' ? 'text-red-700' : 'text-blue-700'

    return (
      <div className={`${bgColor} ${textColor} p-4 rounded-lg mb-4 flex justify-between items-center`}>
        <p>{message}</p>
        <button onClick={onClose} className="font-bold">×</button>
      </div>
    )
  }
  ```

### Task 6-4: 커밋
- [ ] `git commit -m "feat: add error handling and loading states"`

---

## Phase 7️⃣: 최적화 및 배포 (1-1.5시간)

### Task 7-1: 응답성 개선 (25min)
- [ ] 낙관적 업데이트 추가
- [ ] 불필요한 리렌더링 방지 (useMemo, useCallback)

### Task 7-2: 빌드 최적화 (25min)
- [ ] 프로덕션 빌드:
  ```bash
  npm run build
  ```

### Task 7-3: README.md 작성 (20min)
- [ ] 프로젝트 설명
- [ ] 설치 방법
- [ ] 사용 방법
- [ ] 스크린샷

### Task 7-4: 커밋
- [ ] `git commit -m "feat: optimize build and add documentation"`

---

## 📋 **최종 체크리스트**

```
프로젝트 기초:
- [ ] React 프로젝트 생성
- [ ] Tailwind CSS 설정
- [ ] 라우팅 설정

인증:
- [ ] 로그인 페이지
- [ ] 회원가입 페이지
- [ ] 보호된 라우트
- [ ] JWT 토큰 저장

Todo 기능:
- [ ] Todo 목록 조회
- [ ] Todo 생성
- [ ] Todo 수정
- [ ] Todo 삭제
- [ ] Todo 완료 표시

UI/UX:
- [ ] 헤더 + 네비게이션
- [ ] 로딩 상태
- [ ] 에러 처리
- [ ] 반응형 디자인

배포:
- [ ] 프로덕션 빌드
- [ ] README 작성
- [ ] GitHub 푸시
```

---

## 🚀 **시작하기**

```bash
# 1. 프로젝트 생성
npx create-react-app todo-app --template typescript
cd todo-app

# 2. 라이브러리 설치
npm install axios zustand react-router-dom tailwindcss postcss autoprefixer

# 3. 개발 서버 시작
npm start

# 4. 백엔드 실행 (다른 터미널)
cd ../my-sql-backend
npm run dev
```

---

## 📱 **UI 흐름**

```
로그인 페이지
  ↓
레지스트 / 로그인
  ↓
Todo 페이지 (보호됨)
  ├─ 헤더 (로그아웃)
  ├─ Todo 생성 폼
  └─ Todo 목록
      ├─ 완료 체크박스
      ├─ 수정 (클릭하면 수정)
      └─ 삭제
```

---

## 🎯 **각 Phase 완료 후**

| Phase | 예상 시간 | 완료 신호 |
|-------|---------|---------|
| 1 | 1-2h | `npm start` 작동 |
| 2 | 2-3h | 로그인/회원가입 작동 |
| 3 | 1-1.5h | Zustand store 작동 |
| 4 | 2-3h | Todo CRUD UI 완성 |
| 5 | 1-1.5h | 레이아웃 완성 |
| 6 | 1-1.5h | 에러 처리 완성 |
| 7 | 1-1.5h | 빌드 및 배포 준비 |
| **합계** | **10-15h** | 풀스택 완성! |

---

**백엔드(8-13h) + 프론트(10-15h) = 풀스택 Todo App 완성!** 🎉

**이제 둘 다 준비 완료!** 🚀

각 Phase마다 신호해줄래? 💪
