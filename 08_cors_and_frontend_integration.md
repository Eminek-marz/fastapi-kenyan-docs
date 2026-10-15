# Moduli 08: CORS & Kuunganisha Frontend (React, Vue, Flutter) 🌐

<span class="kenyan-tag">Moduli 08</span>

Kama umewahi kujenga frontend ya **React**, **Next.js**, **Vue**, au **Vite** na kujaribu kuita backend yako ya FastAPI kupitia `fetch()` au `axios`, kuna uwezekano wa 99% kwamba umewahi kukutana na kosa hili jekundu kwenye browser console:

> 🛑 **Console Error:**  
> *"Access to fetch at 'http://localhost:8000/api/data' from origin 'http://localhost:3000' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource."*

Hili ndilo kosa linalowapa ma-junior developers wengi maumivu ya kichwa! Katika moduli hii, tutaelewa CORS ni nini haswa na jinsi ya kuisuluhisha kwa mistari michache tu ndani ya FastAPI.

---

## 1. CORS Ni Nini na Kwanini Browser Inakulinda? 🛡️

**CORS** inasimamia **Cross-Origin Resource Sharing**.

Kwenye ulimwengu wa web security, **Origin** inafafanuliwa na mchanganyiko wa vitu vitatu:
1. **Scheme/Protocol** (mfano: `http` au `https`)
2. **Domain/Host** (mfano: `localhost` au `duka.co.ke`)
3. **Port** (mfano: `:3000`, `:5173`, au `:8000`)

### Kwanini Kosa Linatokea?
* Frontend yako ya React ina-run kwenye: **`http://localhost:3000`** (Port 3000).
* Backend yako ya FastAPI ina-run kwenye: **`http://localhost:8000`** (Port 8000).

Kwa sababu port zao zinatofautiana (`3000` vs `8000`), browser yako inaziona kama vyanzo viwili tofauti kabisa (Cross-Origin). Ili kulinda akaunti za watumiaji dhidi ya mashambulizi ya wizi wa data (kama Cross-Site Request Forgery), browser inazuia frontend kusoma majibu ya backend **isipokuwa tu kama backend itatangaza rasmi kwamba inamruhusu huyo frontend!**

---

## 2. Suluhu: Kuongeza `CORSMiddleware` Kwenye FastAPI 🛠️

FastAPI inakupa middleware iliyo tayari kwa ajili ya kutatua tatizo hili kwa urahisi wa hali ya juu:

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI(title="Nairobi E-Commerce API")

# Orodha ya domains/origins zote unazoziruhusu kuongea na API yako:
origins = [
    "http://localhost:3000",       # React / Next.js dev server
    "http://localhost:5173",       # Vite / Vue / Svelte dev server
    "http://127.0.0.1:3000",
    "http://127.0.0.1:5173",
    "https://nairobiduka.co.ke",    # Production live website yako
    "https://admin.nairobiduka.co.ke"
]

# Ongeza CORSMiddleware kwenye application yako:
app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,            # Ruhusu vyanzo hivi pekee
    allow_credentials=True,           # Ruhusu cookies na auth headers (Authorization: Bearer ...)
    allow_methods=["*"],              # Ruhusu methods zote (GET, POST, PUT, DELETE, OPTIONS, n.k.)
    allow_headers=["*"],              # Ruhusu headers zote
)

@app.get("/api/products")
def get_products():
    return [
        {"id": 1, "name": "Bata Safari Boots", "price": 4500},
        {"id": 2, "name": "Kikoi ya Pwani", "price": 1200}
    ]
```

### Je, Unapaswa Kutumia `allow_origins=["*"]`?
* **Wakati wa Local Development:** Unaweza kuweka `allow_origins=["*"]` (alama ya nyota inamaanisha ruhusu tovuti yoyote duniani). Hii inafaa sana ukiwa una-test haraka bila vikwazo.
* **Kwenye Production:** **USIWEKE `["*"]` kamwe** ikiwa API yako ina data za siri au inatumia cookies/tokens za watumiaji. Weka orodha kamili ya domains zako halisi kama tulivyofanya hapo juu!

---

## 3. Mfano wa Frontend: Jinsi React/JS Inavyoita API Hii 💻

Sasa kwa sababu umewasha CORS kwenye backend yako, frontend yako inaweza kufanya request bila vikwazo vyovyote:

```javascript
// Mfano wa JavaScript (kwenye React au Frontend yoyote):
async function loadProducts() {
  try {
    const response = await fetch("http://127.0.0.1:8000/api/products");
    
    if (!response.ok) {
      throw new Error(`HTTP error! Status: ${response.status}`);
    }
    
    const products = await response.json();
    console.log("Bidhaa zimepatikana kutoka FastAPI:", products);
  } catch (error) {
    console.error("Hitilafu wakati wa kuita API:", error);
  }
}

loadProducts();
```

---

## Muhtasari wa Moduli ya 08 📌
* CORS ni utaratibu wa usalama unaotekelezwa na **Web Browser** kuzuia tovuti moja kusoma data ya tovuti nyingine bila idhini.
* Ili kuruhusu frontend (React, Vue, mobile apps za Flutter) kuwasiliana na FastAPI, lazima uongeze **`CORSMiddleware`**.
* Weka domains zako ndani ya list ya **`allow_origins`**.
* Kwenye production, orodhesha tu domains zako rasmi ili kulinda usalama wa watumiaji.

Sasa tuko tayari kwa ajili ya mradi wetu mkubwa wa mwisho wa vitendo: Kujenga **M-Pesa Payment Callback Webhook & Products API** kwa kutumia kila kitu tulichojifunza! 🇰🇪🔥

👉 Endelea na **[Moduli 09: Real-World Project: M-Pesa Callback & Store API](09_real_world_mpesa_api_project.md)**.
