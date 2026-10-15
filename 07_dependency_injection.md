# Moduli 07: Dependency Injection (`Depends`) 💉

<span class="kenyan-tag">Moduli 07</span>

Kama kuna kipengele kimoja kinachofanya ma-developer wakubwa wa backend kuipenda FastAPI kuliko frameworks zingine zote, ni mfumo wake wa **Dependency Injection** kupitia kitendakazi cha **`Depends`**.

Fikiria unajenga API kubwa yenye endpoints 50. Katika endpoints zote hizo:
* Unahitaji kufungua connection ya database na kuifunga baada ya kumaliza kazi.
* Unahitaji kuangalia kama mtumiaji ametuma security token halali (Authentication).
* Unahitaji kusoma query parameters za pagination (`skip` na `limit`).

Badala ya kuandika code hizo zile zile mistari kwa mistari ndani ya kila function, unaziandika mara moja tu kama **Dependency**, kisha unaiambia FastAPI: *"Hii endpoint inategemea (depends on) hii kazi kabla haijaanza."*

---

## 1. Mfano Rahisi: Shared Query Parameters 🔄

Hebu tuone jinsi ya kushiriki (share) logic ya pagination kati ya endpoints tofauti:

```python
from fastapi import FastAPI, Depends

app = FastAPI()

# 1. Hii ndiyo Dependency yetu (Function ya kawaida ya Python):
def common_parameters(skip: int = 0, limit: int = 10):
    return {"skip": skip, "limit": limit}

# 2. Endpoint ya Bidhaa (Inatumia Depends):
@app.get("/items/")
def read_items(commons: dict = Depends(common_parameters)):
    return {
        "endpoint": "items",
        "pagination_info": commons,
        "items": ["Laptop", "Simu", "Earbuds"][commons["skip"] : commons["skip"] + commons["limit"]]
    }

# 3. Endpoint ya Watumiaji (Inatumia Depends ile ile moja):
@app.get("/users/")
def read_users(commons: dict = Depends(common_parameters)):
    return {
        "endpoint": "users",
        "pagination_info": commons,
        "users": ["Alice", "Bob", "Charlie", "David"][commons["skip"] : commons["skip"] + commons["limit"]]
    }
```

### Nini Kinafanyika Nyuma ya Pazia?
Wakati request inapofika kwa `/items/?skip=1&limit=2`:
1. FastAPI inaona `Depends(common_parameters)`.
2. Inachukua parameters kutoka kwenye URL na kuziita ndani ya `common_parameters(skip=1, limit=2)`.
3. Inachukua matokeo yaliyotoka (dictionary) na kuyapitisha kama variable ya `commons` ndani ya `read_items`!
4. **Kwenye Swagger UI (`/docs`)**: FastAPI inasoma dependency hiyo na kuonyesha masanduku ya `skip` na `limit` kiotomatiki kwa endpoints zote mbili!

---

## 2. Authentication Dependency (Ulinzi wa Token) 🔑

Hebu tuone mfano halisi wa ulinzi: jinsi ya kuzuia endpoint isifikiwe bila Header maalum ya ulinzi:

```python
from fastapi import FastAPI, Depends, HTTPException, Header, status

app = FastAPI()

# Dependency ya kuthibitisha API Key:
def verify_api_key(x_api_key: str = Header(...)):
    # Hapa tunathibitisha kama key ni sahihi:
    if x_api_key != "siri-ya-nairobi-2026":
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Ufikiaji umekataliwa: API Key uliyotuma si sahihi au imeisha muda."
        )
    return x_api_key

# Endpoint ya kawaida (Wazi kwa kila mtu):
@app.get("/public")
def public_route():
    return {"message": "Karibu! Ukurasa huu ni wazi kwa kila mtu."}

# Endpoint ya siri (Inalindwa na verify_api_key):
@app.get("/admin/dashboard", dependencies=[Depends(verify_api_key)])
def admin_dashboard():
    return {
        "revenue_kes": 1500000.0,
        "active_users": 4200,
        "status": "Admin access granted"
    }
```

Mtu akijaribu kuita `/admin/dashboard` bila header ya `X-API-Key: siri-ya-nairobi-2026`, FastAPI itamsimamisha moja kwa moja kabla hata code ya `admin_dashboard` haijaanza kufanya kazi!

---

## 3. Dependencies zenye `yield` (Database Sessions & Cleanup) 🗄️

Wakati unafanya kazi na Database (kama vile PostgreSQL, MySQL, au SQLite kupitia SQLAlchemy):
1. Lazima ufungue database session.
2. Utekeleze query.
3. **Na zaidi ya yote: lazima ufunge session hiyo hata kama error imetokea!** (Kuzuia database connection leaks).

FastAPI inaruhusu kutumia neno la Python **`yield`** kufanikisha hili kwa urahisi wa hali ya juu:

```python
# Mfano wa jinsi Database Sessions zinavyofanya kazi:
def get_db():
    db = FakeDatabaseConnection()
    try:
        print("🟢 [DB CONNECTED] Database session imefunguliwa.")
        yield db # Hapa ndipo endpoint inapofanya kazi zake
    finally:
        print("🔴 [DB CLOSED] Database session imefungwa salama.")
        db.close()

@app.get("/orders/")
def get_all_orders(db = Depends(get_db)):
    # Code inatumia 'db' hapa kwa amani...
    return {"orders": ["Order #1", "Order #2"]}
```

* Kila kitu kilicho **kabla** ya `yield` kinatekelezwa **kabla** ya endpoint yako kuitwa.
* Kila kitu kilicho **baada** ya `yield` (ndani ya block ya `finally`) kinatekelezwa kiotomatiki **baada** ya response kutumwa kwa client!

---

## Muhtasari wa Moduli ya 07 📌
* **Dependency Injection (`Depends`)** huwezesha kuandika logic mara moja na kuitumia tena na tena bila kurudia code (DRY principle).
* Inafaa sana kwa mambo ya **Pagination**, **Authentication/Security**, na **Database Connections**.
* FastAPI inajumuisha dependencies zote kiotomatiki kwenye Swagger UI (`/docs`).
* Dependencies zinazotumia **`yield`** zinahakikisha rasilimali (kama database connections au files) zinafungwa kwa usalama baada ya request kukamilika.

Kwenye moduli inayofuata, tutaangalia jinsi ya kuruhusu tovuti za frontend (React, Vue, Flutter) kuwasiliana na API yako bila kuzuiwa na browser kupitia **CORS & Middleware**!

👉 Endelea na **[Moduli 08: CORS & Kuunganisha Frontend](08_cors_and_frontend_integration.md)**.
