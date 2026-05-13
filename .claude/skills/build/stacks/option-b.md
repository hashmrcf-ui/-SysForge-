# الخيار B — النمو المنظم

## متى تختاره
- 10-200 مستخدم
- تريد إضافة ميزات تدريجياً
- شركة واحدة لكن بنية قابلة للتوسع
- ستضيف تطبيق موبايل لاحقاً

## التقنيات

| الطبقة | التقنية | السبب |
|---|---|---|
| الواجهة | Next.js 14 (App Router) | قوي + SEO + SSR |
| الـ UI | shadcn/ui + Tailwind | محترف وقابل للتخصيص |
| الخادم | FastAPI (Python) | منفصل، سريع، AI-native |
| قاعدة البيانات | PostgreSQL + SQLAlchemy | قوي، مرن |
| وكيل AI | LangChain + Claude/GPT-4 | قدرات AI أقوى |
| المصادقة | Clerk أو JWT يدوي | أسهل وأكثر مرونة |
| النشر | Vercel (Frontend) + Railway (Backend) | سهل ورخيص |

## هيكل المشروع

```
my-system/
├── frontend/                      ← Next.js منفصل
│   ├── app/
│   │   ├── (auth)/
│   │   ├── (dashboard)/
│   │   │   ├── layout.tsx
│   │   │   ├── page.tsx
│   │   │   └── [module]/
│   │   └── api/                   ← Next.js API (proxy only)
│   ├── components/
│   │   ├── ui/
│   │   ├── layout/
│   │   └── [module]/
│   ├── lib/
│   │   └── api-client.ts          ← استدعاءات FastAPI
│   └── package.json
│
├── backend/                       ← FastAPI منفصل
│   ├── routers/
│   │   ├── auth.py
│   │   ├── [module].py            ← router لكل وحدة
│   │   └── ai.py
│   ├── models/                    ← SQLAlchemy models
│   │   └── [module].py
│   ├── schemas/                   ← Pydantic schemas
│   │   └── [module].py
│   ├── services/                  ← Business logic
│   │   ├── [module]_service.py
│   │   └── ai_service.py
│   ├── core/
│   │   ├── config.py
│   │   ├── database.py
│   │   └── security.py
│   ├── main.py
│   └── requirements.txt
│
├── database/
│   └── alembic/                   ← DB migrations
│
└── docker-compose.yml             ← تشغيل محلي كامل
```

## أوامر البدء

```bash
# === Frontend ===
npx create-next-app@latest frontend --typescript --tailwind --app
cd frontend
npm install axios @tanstack/react-query
npx shadcn@latest init

# === Backend ===
mkdir backend && cd backend
python -m venv venv
venv\Scripts\activate          # Windows
pip install fastapi uvicorn sqlalchemy psycopg2-binary
pip install python-jose passlib alembic anthropic langchain
pip freeze > requirements.txt

# === تشغيل محلي ===
# docker-compose up -d
```

## docker-compose.yml

```yaml
version: '3.8'
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"

  backend:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: postgresql://user:password@db:5432/mydb
      ANTHROPIC_API_KEY: ${ANTHROPIC_API_KEY}
    depends_on:
      - db

  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      NEXT_PUBLIC_API_URL: http://localhost:8000
```

## متغيرات البيئة

```env
# backend/.env
DATABASE_URL=postgresql://user:password@localhost:5432/mydb
SECRET_KEY=your-jwt-secret
ANTHROPIC_API_KEY=sk-ant-...

# frontend/.env.local
NEXT_PUBLIC_API_URL=http://localhost:8000
```

## عدد الجلسات المتوقعة: 8-12 جلسة
