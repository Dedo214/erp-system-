# Akt Group ERP

نظام إدارة المقاولات والمشروعات والمحاسبة والمشتريات والمخزون والموارد البشرية.

## المتطلبات

- Python 3.12 أو أحدث.
- PostgreSQL 16 للإنتاج والاختبارات الكاملة.
- Docker Compose عند استخدام النشر المعبأ.

## تشغيل محلي سريع

```bash
python3 -m venv .venv
. .venv/bin/activate
pip install -r requirements.txt
export AKT_SECRET_KEY="$(openssl rand -hex 32)"
export DATABASE_URL="sqlite:///./akt.db"
python -m pytest
uvicorn app.main:create_app --factory --reload
```

يفتح التطبيق على `http://127.0.0.1:8000`.

## تشغيل PostgreSQL

في بيئة الإنتاج يجب استخدام PostgreSQL وعدم الاعتماد على SQLite:

```bash
export DATABASE_URL='postgresql+psycopg://akt:password@localhost:5432/akt'
export AKT_SECRET_KEY="$(openssl rand -hex 32)"
export AKT_AUTO_CREATE=0
alembic upgrade head
python -m app.cli create-admin admin
uvicorn app.main:create_app --factory --host 0.0.0.0 --port 8000
```

## Docker Compose

انسخ `.env.example` إلى `.env`، وعدّل `DOMAIN` و`DB_PASSWORD` و`AKT_SECRET_KEY`، ثم شغّل:

```bash
docker compose config
docker compose up -d --build
docker compose ps
docker compose logs --tail=200
```

لا تُعرّض PostgreSQL للإنترنت. يجب توجيه DNS للنطاق إلى الخادم حتى يصدر Caddy شهادة HTTPS.

## الاختبارات

```bash
pytest
pytest tests/test_phase1.py -q
```

إعداد `pytest.ini` يقصر جمع الاختبارات على `tests`. اختبار التزامن يعمل تلقائيًا عند وجود PostgreSQL، ويتخطى نفسه عند عدم توفره.

## اختبار التحمل

الأداة ليست اختبار pytest وتُشغّل يدويًا فقط:

```bash
python tools/loadtest.py http://localhost:8766 <username> '<password>' [entries] [threads]
```

شغّلها على بيئة اختبار، لأنها تنشئ مشاريع وقيودًا تجريبية.

## النسخ والاستعادة

إنشاء نسخة تطبيقية يتم عبر `POST /admin/backup` ويتطلب صلاحية `backup.run`. الاستعادة تتم عبر `POST /admin/restore` وتتطلب سببًا وصلاحية `backup.restore`.

يجب كذلك تشغيل نسخ PostgreSQL الخارجية، واختبار الاستعادة دوريًا على بيئة منفصلة قبل استخدام النظام على بيانات حقيقية.

## الترحيلات

```bash
alembic upgrade head
alembic check
```

لا تعدّل جداول الإنتاج يدويًا. أي تغيير في المخطط يجب أن يكون Migration قابلة للمراجعة والاختبار.

## التوثيق

- `docs/GUIDE_AR.md`: دليل التشغيل والاستخدام.
- `docs/API.md`: قائمة API.
- `docs/DATABASE.md`: مخطط قاعدة البيانات.
- `docs/SECURITY_REVIEW.md`: المراجعة الأمنية الحالية.
- `docs/FINAL_REPORT.md`: حالة المراحل والقيود المعروفة.
- `docs/CHANGELOG.md`: سجل تغييرات الإصدارات.
