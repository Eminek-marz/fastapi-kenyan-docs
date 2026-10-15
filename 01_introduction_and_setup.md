# Moduli 01: Utangulizi wa FastAPI & Setup ya Mazingira 🚀

<span class="kenyan-tag">Moduli 01</span>

Karibu kwa module ya kwanza ya mwongozo wetu wa **FastAPI**! Katika module hii, tutaweka development environment, ku-install vitu vinavyohitajika, kuandika API yetu ya kwanza kabisa, na kuona maajabu ya interactive documentation ya Swagger UI (`/docs`).

---

## 1. Virtual Environment Ni Nini na Kwanini Ni Lazima? 🧱

Kabla ya ku-install package yoyote ya Python, sheria ya kwanza ya software engineering ni: **Usiwahi ku-install packages globally kwa computer yako yote**.

Fikiria **Virtual Environment** kama chumba kidogo kilichotengwa ndani ya project yako. Inahakikisha kwamba version za packages unazotumia kwenye project hii hazibishani na projects zingine ulizo nazo kwenye mashine yako.

### Hatua ya 1: Fungua Terminal na Utengeneze Folder
Kwa terminal yako (PowerShell au Command Prompt):

```bash
# Tengeneza folder ya project na uingie ndani
mkdir fastapi_demo
cd fastapi_demo

# Tengeneza virtual environment inayoitwa 'venv'
python -m venv venv
```

### Step 2: Ku-activate Virtual Environment
* **Kwa Windows (PowerShell):**
  ```powershell
  .\venv\Scripts\Activate.ps1
  ```
  *(Kama ukiona error ya permission/execution policy, andika: `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` kisha u-run tena).*

* **Kwa Mac / Linux:**
  ```bash
  source venv/bin/activate
  ```

Ukiona alama ya `(venv)` imetokea mwanzoni mwa mstari wako wa terminal, inamaanisha uko salama ndani ya environment yako!

---

## 2. Ku-install FastAPI na Uvicorn 📦

FastAPI yenyewe ni framework ya kuandika code, lakini inahitaji **ASGI server** ili iweze ku-listen kwa requests za network na kuzijibu kwa kasi ya juu. Hapa ndipo **Uvicorn** inapoingia:

```bash
pip install "fastapi[standard]"
```

> [!TIP]
> Kuweka `[standard]` kunaiambia pip i-install FastAPI pamoja na Uvicorn na dependencies zote za recommended tools kama email validators na Pydantic extensions.

---

## 3. Kuandika API Yako ya Kwanza (`main.py`) ✍️

Tengeneza file mpya inayoitwa `main.py` kwenye folder yako ya project na uandike code hizi:

```python
from fastapi import FastAPI

# Hapa tunatengeneza instance(hii ni mojawapo ya concepts ambazo zinatumika kwa OOP simply kumaanisha," that is a physical illustration of class ama reality of a thought," e.g Man is an instance of the universal information.) kuu ya application yetu
app = FastAPI(
    title="Nairobi Store API",
    description="API ya kwanza ya kujifunzia FastAPI kwa Kiswahili cha Kenya 🇰🇪",
    version="1.0.0"
)

# Hii inaitwa Route Decorator(wadau najua mnauliza route decorator inamaanisha nini na simply it comes from two words route which inamanisha giving a direction to specific data(hii itakuwa tu kumatch an incoming http request with a specific function of the code ndio ifanyiwe certain thing, Nayo decorators ni hiyo function which is like an extension of the behaviour of the function/class )): inasema request ikija kwa GET kwenye root ("/") fanya kazi hii:
#To be clear the @app.get("/") inatell python to look for anyone visiting the homepage and immediately execute the function below.

@app.get("/")
async def root():
    return {
        "status": "success",
        "message": "Karibu Nairobi Store API!",
        "developer": "Eminek",
        "country": "Kenya 🇰🇪"
    }

@app.get("/health")
def health_check():
    return {"status": "ok", "uptime": "Server is running smoothly"}
```

### Hebu Tuchambue Code Hii:
1. **`from fastapi import FastAPI`**: Tuna-import class ya `FastAPI` which is located from the fasstapi library.
2. **`app = FastAPI()`**: Hii ndiyo object/instance(coz at this particular point its creating a real thing) kuu itakayosimamia endpoints na routing zote za backend yako.
3. **`@app.get("/")`**: Hii inaitwa **Path Operation Decorator**:
   * `@app`: Inaunganishwa na application yetu.(inawrap the operation get na path of that specific part of your application with the function below)
   * `.get`: Inamaanisha HTTP Method inayokubalika ni `GET` (kusoma data).
   * `"/"`: Hii ndiyo path (URL endpoint) — hapa ni root.
4. **`async def root()`**: Function inayoitwa path operation function. Inaweza kuwa `async def` (asynchronous) au `def` ya kawaida. FastAPI ni mwerevu: ukitumia `async def`, inaiendesha bila ku-block event loop ya server!
5. **`return {...}`**: Umeona jinsi tulivyorudisha Python Dictionary ya kawaida? FastAPI inachukua dictionary hiyo automatically na kui-convert kuwa **JSON** safi yenye header ya `content-type: application/json`. Huna haja ya ku-import `json` wala kuita `json.dumps()`!

---

## 4. Running the Server ⚡

Sasa hebu tuwaopen server yetu kwa kutumia **Uvicorn**:

```bash
uvicorn main:app --reload
```

### Maneno Haya Yanamaanisha Nini?
* **`main`**: Jina la Python file lako (`main.py`).
* **`app`**: Jina la variable ya `FastAPI()` enye tulidescribe ndani ya `main.py` (`app = FastAPI()`).
* **`--reload`**: Hii ni flag ya maana sana wakati wa development! Inaambia Uvicorn: *"Kila nikibadilisha code na ku-save (Ctrl+S), reload server automatically bila mimi kuizima na kuiwasha tena."*

Utaona output kama hii kwenye terminal yako:
```text
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```

Sasa fungua browser yako na uende: **`http://127.0.0.1:8000/`**  
Utaona JSON response yako mara moja:
```json
{
  "status": "success",
  "message": "Karibu Nairobi Store API!",
  "developer": "Eminek",
  "country": "Kenya 🇰🇪"
}
```

---

## 5. Maajabu ya Swagger UI (`/docs`) na ReDoc (`/redoc`) ✨

Hapa ndipo FastAPI inapopiga frameworks zingine zote (kama Flask au Express.js) bao!

Bila wewe kuandika documentation yoyote ya ziada, fungua browser yako na uweke URL hii:
👉 **`http://127.0.0.1:8000/docs`**

Utapata **Interactive Swagger UI**:
* Unaona kila endpoint uliyoitengeneza (`GET /`, `GET /health`).
* Unaweza kuclick button ya **"Try it out"** kisha click **"Execute"** ili ku-test API yako live bila kuhitaji Postman!
* Unaona curl command, request URL, response body, na status codes.

Ukitaka layout different uliotulia for read only,go to this link:
👉 **`http://127.0.0.1:8000/redoc`**

---

## Summary ya Module ya 01 📌
* Tumetengeneza virtual environment ili kutenga packages zetu.
* Tume-install `fastapi[standard]` yenye Uvicorn server ndani yake.
* Tumeunda API ya kwanza yenye route decorator `@app.get("/")`.
* Tumegundua kwamba FastAPI inageuza Python dictionaries kuwa JSON automatically.
* Tumeona jinsi ya ku-test endpoints moja kwa moja kupitia **`/docs`**.

Kwenye module inayofuata, tutadelve deeper zaidi na kuona jinsi ya kupokea **Path Parameters** (kama IDs za vitu) na jinsi **Python Type Hints** zinavyozuia makosa ya data kabla hayajatokea!

👉 Endelea na **[Module 02: Path Parameters & Type Hints](02_path_parameters_and_types.md)**.
