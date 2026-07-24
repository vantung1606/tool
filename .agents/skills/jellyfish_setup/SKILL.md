---
name: jellyfish_setup
description: Huong dan cai dat va chay du an Jellyfish - ung dung tao phim ngan bang AI. Kich hoat khi nguoi dung hoi ve cach chay du an, cai dat, loi khi khoi dong, hoac khi workspace hien tai la thu muc Jellyfish.
---

# Jellyfish - Huong dan Cai dat & Chay Du an

Day la du an tao phim ngan bang AI, gom:
- **Backend:** Python / FastAPI + SQLite
- **Frontend:** React / Vite (chay tren port 7788)
- **AI:** Gemini (text) + Luma AI (video)

---

## Yeu cau truoc khi cai

| Cong cu | Phien ban | Link tai |
|---------|-----------|----------|
| Python | >= 3.11 | https://www.python.org/downloads/ |
| Node.js | >= 18 | https://nodejs.org/ |
| uv | moi nhat | `pip install uv` |
| pnpm | moi nhat | `npm install -g pnpm` |

---

## Buoc 1: Clone du an

```bash
git clone https://github.com/vantung1606/tool.git Jellyfish
cd Jellyfish
```

---

## Buoc 2: Cai dat & Chay Backend

```bash
cd backend

# Cai dependencies
uv sync

# Khoi tao database
uv run python init_db.py

# Chay backend (port 8000)
uv run uvicorn app.main:app --reload --port 8000
```

> Backend: http://localhost:8000
> Swagger docs: http://localhost:8000/docs

---

## Buoc 3: Cai dat & Chay Frontend

Mo terminal moi (giu backend dang chay):

```bash
cd front

# Cai dependencies
pnpm install

# Chay frontend (port 7788)
pnpm dev
```

> Frontend: http://localhost:7788

---

## Buoc 4: API Key da co san trong backend/.env

File `backend/.env` da chua san:
- GEMINI_API_KEY (Google Gemini)
- LUMA_API_KEY (Luma AI video)

Neu muon cap nhat key moi, vao http://localhost:7788 -> Cai dat -> Nha cung cap.

---

## Xu ly loi thuong gap

### Loi: PowerShell khong chay duoc pnpm
```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

### Loi: 429 Quota exceeded
API key Gemini free tier het quota ngay. Cho den ngay mai hoac tao key moi tai:
https://aistudio.google.com/apikey

### Loi: 503 Unsupported provider
Vao Cai dat -> Nha cung cap -> kiem tra API key da duoc nhap chua.

### Loi: unexpected model name format
Chay lenh fix:
```bash
cd backend
uv run python -c "
import asyncio
from app.core.db import async_session_maker
from app.models.llm import Model
from sqlalchemy import select

async def fix():
    async with async_session_maker() as db:
        name_map = {
            'Gemini 1.5 Flash': 'gemini-1.5-flash',
            'Gemini 2.0 Flash': 'gemini-2.0-flash',
            'Luma Dream Machine': 'dream-machine',
        }
        models = (await db.execute(select(Model))).scalars().all()
        for m in models:
            if m.name in name_map:
                m.name = name_map[m.name]
        await db.commit()
        print('Fixed!')

asyncio.run(fix())
"
```

---

## Cau truc du an

```
Jellyfish/
├── backend/          # FastAPI server
│   ├── app/
│   ├── .env          # API keys
│   ├── jellyfish.db  # SQLite database
│   └── init_db.py    # Script khoi tao DB
├── front/            # React frontend
│   └── src/locales/vi-VN/  # Ban dich tieng Viet
└── .agents/
    └── skills/jellyfish_setup/SKILL.md
```

---

## Luu y quan trong

- Khong can Docker hay Redis - du an chay hoan toan offline
- Tasks se chay bang Python threading (khong can Celery)
- Database la SQLite - khong can PostgreSQL
- UI mac dinh la tieng Viet
