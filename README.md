# Sistemi Genit Cloud — Versioni Django

Ripërtim i **Sistemi Genit Cloud** (ERP cloud multi-tenant / multi-company / multi-magazinë)
i ndërtuar me **Django 5**. Projekti origjinal ishte një monorepo Node.js/Express + React + PostgreSQL;
ky version ruan qëllimin dhe modulet kryesore të sistemit, por me arkitekturë Django (MVT).

## Modulet e implementuara

- **Cloud Core** — Konfigurimi i parë (organizatë, kompani, magazinë, super admin), login/logout,
  Dashboard me KPI, Kompanitë, Magazinat, Përdoruesit & rolet, Audit Log.
- **Regjistra** — Artikujt (produkte), Furnitorët, Klientët.
- **Blerje & Peshim** — Formulari i Peshave (me llogaritje neto/zbritje/vlerë dhe konfirmim që
  krijon lëvizje stoku), Kërkesa për Ofertë, Porosi Blerjeje, Pranime, Fatura Blerjeje.
- **Shitje & Magazinë** — Oferta Shitjeje, Porosi Shitjeje, Fletë-Dalje, Fatura Shitjeje, Stoku.
- **Stoku** — Lëvizje stoku të grumbulluara sipas kompanisë/magazinës/artikullit.

## Struktura

```text
genit-cloud-django/
├── manage.py
├── requirements.txt
├── README.md
├── genit_cloud/          # settings, urls, wsgi
├── core/                 # aplikacioni kryesor
│   ├── models.py         # Tenant, Company, Warehouse, User, AuditLog, Product,
│   │                     # BusinessPartner, WeightTicket, StockMovement,
│   │                     # BusinessDocument (+Item)
│   ├── forms.py
│   ├── views.py
│   ├── services.py       # audit, lëvizje stoku, konfirmim dokumentesh
│   ├── urls.py
│   ├── admin.py
│   ├── static/css/app.css
│   └── templates/core/   # base, login, setup, dashboard, lista, forma, detaje
└── db.sqlite3            # krijohet pas migrate
```

## Si ta nisësh

```bash
# 1. Instalo varësitë
pip install -r requirements.txt

# 2. Krijo dhe aplikonigrimet
python manage.py makemigrations
python manage.py migrate

# 3. Nis serverin
python manage.py runserver
```

Hap `http://127.0.0.1:8000/` — do të shfaqet **Konfigurimi i parë** ku krijon
organizatën, kompaninë, magazinën dhe Super Administratorin. Pas ruajtjes, sistemi
kalon në login normal.

### PostgreSQL (opsionale)

```bash
export DATABASE_URL=postgresql://USER:PASS@HOST:5432/DB
pip install psycopg2-binary dj-database-url
python manage.py migrate
```

## Logjika e biznesit

- **Formulari i Peshave**: peshë bruto − ambalazh = neto; neto × (1 − zbritje%) = e pranuar;
  e pranuar × çmim = vlerë. Konfirmimi krijon lëvizje stoku `WEIGHT_RECEIPT`.
- **Dokumentet e biznesit**: RFQ, Porosi Blerjeje, Pranime, Faturë Blerjeje, Ofertë,
  Porosi Shitjeje, Fletë-Dalje, Faturë Shitjeje. Shto rreshta, llogarit totalet
  (neto + TVSH), konfirmo (krijon lëvizje stoku hyrje/dalje sipas llojit) ose anulo.
- **Stoku**: grumbullim i lëvizjeve sipas kompanisë, magazinës dhe artikullit.
- **Audit Log**: çdo veprim regjistrohet me përdorues, veprim, objekt, kompani dhe IP.

## Kredencialet demo (të krijuara gjatë testit)

- Username: `admin` / Password: `admin12345` (organizata "Demo Org", kompania "BIOBES SHPK")

> Për një instalim të pastër, fshij `db.sqlite3` dhe rikrijo me `migrate` + setup.