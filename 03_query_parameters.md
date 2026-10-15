# Moduli 03: Query Parameters & Filtering 🔎

<span class="kenyan-tag">Moduli 03</span>

Wakati **Path Parameters** zinatumika kuidentify kitu maalum (kama `/products/12`), **Query Parameters** zinatumika kufanya **filtering, searching, sorting, na pagination** (mfano: `/products?category=electronics&limit=10`).

Kwa module hii, tutaona venye FastAPI inavyodifferentiate kirahisi path parameters na query parameters, jinsi ya kuweka default values, na jinsi ya kuongeza validation rules kwa kutumia `Query()`.

---

## 1.Primary Query Parameter  Ni Nini? ❓

Kwa FastAPI, ukiongeza parameter kwa function yako ambayo **haijatajwa kwenye path**, FastAPI inajua moja kwa moja kwamba hiyo ni **Query Parameter** inayotoka kwenye URL baada ya alama ya `?`.

```python
from fastapi import FastAPI

app = FastAPI()

fake_items_db = [
    {"item_name": "Samsung Galaxy A54", "price": 42000},
    {"item_name": "iPhone 15 Pro", "price": 165000},
    {"item_name": "HP EliteBook 840", "price": 55000},
    {"item_name": "Wireless Mouse", "price": 1500},
    {"item_name": "USB-C Fast Charger", "price": 2500},
]

# Hapa 'skip' na 'limit' haziko kwenye path: ni Query Parameters!
@app.get("/items/")
def read_items(skip: int = 0, limit: int = 2):
    return fake_items_db[skip : skip + limit]
```

### Jinsi Inavyofanya Kazi Kwenye Browser:
1. Ukienda: **`http://127.0.0.1:8000/items/`**  
   FastAPI itatumia default values: `skip=0` na `limit=2`. Utapata bidhaa 2 za kwanza.
2. Ukitaka kuruka bidhaa 2 za kwanza na uchukue 2 zinazofuata:  
   👉 **`http://127.0.0.1:8000/items/?skip=2&limit=2`**
3. Ukitaka kuchukua bidhaa 5:  
   👉 **`http://127.0.0.1:8000/items/?limit=5`**

---

## 2. Optional Parameters & Booleans (TRUE  OR FALSE) ⚖️

How about wakati unataka parameter ikuwe optional, yaani user anaweza kuituma au kuiacha?
Kwa Python ya kisasa (3.10+), tunatumia syntax safi ya `type | None = None`:

```python
@app.get("/products/{product_id}")
def get_product(product_id: int, q: str | None = None, available: bool = True):
    result = {"product_id": product_id, "is_available": available}
    
    if q:
        result["search_query"] = q
        
    return result
```

### Tofauti na Frameworks Zingine:
Angalia venye FastAPI inavyo-handle aina ya data ya `bool` (`available: bool`):
URL zote hizi zitatafsiriwa kama `True`:
* `?available=true`
* `?available=1`
* `?available=yes`
* `?available=on`

Na hizi zitatafsiriwa kama `False`:
* `?available=false`
* `?available=0`
* `?available=no`

FastAPI inafanya conversion hii automatically bila wewe kuandika code ya ku-check string!

---

## 3. Kuweka Mipaka ya Kina kwa Kutumia `Query()` 🛡️
 
How about unadai search query  isikuwa fupi sana (mfano: iwe na angalau herufi 3 ili kuzuia database overload), na isizidi herufi 20?

Hapa tuna-import **`Query`** kutoka `fastapi`:

```python
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/search/")
def search_inventory(
    q: str = Query(
        min_length=3,
        max_length=20,
        title="Neno la Kutafuta",
        description="Andika jina la kifaa unachotafuta (herufi 3 hadi 20)",
        examples=["samsung", "charger"]
    )
):
    return {"searching_for": q, "status": "results found"}
```

### Nini Kinatokea User Akituma Herufi 2 Pekee?
Ukienda: `http://127.0.0.1:8000/search/?q=hp`  
FastAPI inakataa mara moja na kurudisha **422 Unprocessable Entity**:
```json
{
  "detail": [
    {
      "type": "string_too_short",
      "loc": ["query", "q"],
      "msg": "String should have at least 3 characters",
      "input": "hp"
    }
  ]
}
```

---

## 4. Kupokea List ya Query Parameters (Multiple Values) 📋

Wakati mwingine unataka kuruhusu user kuchuja vitu kwa kutumia categories au tags nyingi kwa wakati mmoja, mfano:  
`/items/?tag=phone&tag=samsung&tag=electronics`

Unaweza kufanya hivyo kirahisi kwa kutumia `list[str]`:

```python
@app.get("/filter/")
def filter_by_tags(tags: list[str] = Query(default=["general"])):
    return {
        "selected_tags": tags,
        "total_tags_count": len(tags)
    }
```

Ukiita: `http://127.0.0.1:8000/filter/?tags=phones&tags=deals&tags=nairobi`  
FastAPI inakupa Python list kamili:
```json
{
  "selected_tags": ["phones", "deals", "nairobi"],
  "total_tags_count": 3
}
```

---

## Summary ya Module ya 03 📌
* Parameters ambazo haziko kwenye path zinachukuliwa kama **Query Parameters**.
* Tumia **default values** (`skip: int = 0`) kwa ajili ya pagination na filtering.
* Tumia **`str | None = None`** kutengeneza parameters za hiari (optional).
* FastAPI inageuza maneno kama `yes`, `1`, `true` kuwa Python `True` boolean.
* Tumia **`Query(min_length=..., max_length=...)`** kuongeza sheria kali za data validation na documentation nzuri ndani ya `/docs`.

Kwenye moduli inayofuata, tutaingia kwenye moyo halisi wa REST APIs: jinsi ya kupokea data kubwa (JSON payloads) kwa kutumia **Request Body & Pydantic Models**!

👉 Endelea na **[Moduli 04: Request Body & Pydantic Models](04_request_body_and_pydantic.md)**.
