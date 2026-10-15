# Moduli 04: Request Body & Pydantic Schemas 📦

<span class="kenyan-tag">Moduli 04</span>

Unapotaka kutuma data kutoka kwa frontend (au mobile app) kwenda kwenye server yako ili kuunda kitu kipya (kwa mfano: kujiandikisha kama user mpya, kuweka oda ya bidhaa, au kutuma malipo), huwezi kuweka data yote hiyo kwenye URL. 

Unaituma ndani ya **Request Body** kwa kutumia HTTP Methods kama **`POST`**, **`PUT`**, au **`PATCH`**.

Kwa module hii, tutaona venye FastAPI inavyoshirikiana na library maarufu ya **Pydantic** kufanya data validation na serialization kwa kiwango cha juu sana.

---

## 1. Pydantic Ni Nini na Kwanini Ni Muhimu? 🧠

**Pydantic** ni library ya Python inayotumia standard Python type hints kuthibitisha (validate) kwamba data inayoingia au kutoka inalingana na mfumo unaoutaka.

Badala ya kupokea raw dictionary na kuanza kuandika:
```python
# ❌ NJIA YA ZAMANI (Stress tupu):
data = request.json()
if "price" not in data or data["price"] < 0:
    return "Error: Bei haiwezi kuwa negative"
```

Unaunda **Pydantic Model** mara moja tu, na FastAPI inashughulikia mengine yote!

---

## 2. Kuunda Model ya Kwanza kwa `BaseModel` 🏗️

Tengeneza schema ya product mpya:

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

# Tunainherite kutoka kwa BaseModel ya Pydantic:
class Product(BaseModel):
    name: str
    description: str | None = None
    price: float = Field(gt=0, description="Bei lazima iwe kubwa kuliko zero")
    tax: float | None = None
    in_stock: bool = True

@app.post("/products/")
def create_product(product: Product):
    # 'product' hapa ni instance halisi ya Pydantic Model!
    total_price = product.price
    if product.tax:
        total_price += product.tax
        
    return {
        "message": f"Bidhaa ya {product.name} imeundwa kikamilifu!",
        "item_data": product.model_dump(), # model_dump() inageuza model kuwa Python dict
        "final_price_kes": total_price
    }
```

### Nini Kinafanyika Hapa?
1. Client akituma JSON payload kama hii:
   ```json
   {
     "name": "Sony WH-1000XM5 Headphones",
     "description": "Noise cancelling headphones",
     "price": 45000.0,
     "tax": 7200.0
   }
   ```
2. FastAPI inachukua JSON hiyo na kuisoma kama `Product`.
3. Inathibitisha kwamba `price` ni namba na iko juu ya `0` (`gt=0`).
4. Kwa function yako, unapata autocomplete ya VS Code ukiandika `product.name`, `product.price`, n.k.!

### Ikiwa Client Akituma Data Mbaya?
Mteja akijaribu kutuma `price: -500` au `price: "bure"`, FastAPI inamrudishia mara moja **422 Unprocessable Entity**:
```json
{
  "detail": [
    {
      "type": "greater_than",
      "loc": ["body", "price"],
      "msg": "Input should be greater than 0",
      "input": -500
    }
  ]
}
```

---

## 3. Kuchanganya Path, Query, na Request Body kwa Pamoja 🔀

FastAPI ni framework proficient sana. Inaweza kutofautisha kati ya hizi data sources tatu ndani ya function moja:

```python
@app.put("/items/{item_id}")
def update_item(
    item_id: int,               # 1. Iko kwenye path URL -> Path Parameter
    item: Product,              # 2. Ni Pydantic Model -> Request Body (JSON)
    notify_users: bool = False  # 3. Parameter ya kawaida -> Query Parameter
):
    results = {"item_id": item_id, "updated_item": item.model_dump()}
    if notify_users:
        results["notification"] = "Users notified via SMS/Email"
    return results
```

* **Path Parameter (`item_id`)**: Inasomwa kutoka kwa URL path: `/items/42`.
* **Query Parameter (`notify_users`)**: Inasomwa kutoka kwa query string: `?notify_users=true`.
* **Request Body (`item`)**: Inasomwa kutoka kwenye JSON body ya request.

---

## 4. Nested Models (Model Ndani ya Model Nyingine) 🪆

Kwa real world situation, data haikuagi flat. Mfano: product inaweza kuwa na list ya picha (`Image`), na duka inaweza kuwa na  (`Address`).

Pydantic inakuruhusu kuweka models(nested models) ndani ya models zingine beautifully:

```python
from pydantic import BaseModel, HttpUrl

class ProductImage(BaseModel):
    url: HttpUrl
    caption: str

class StoreInventory(BaseModel):
    product_name: str
    tags: set[str] = set() # Set inazuia duplicate tags automatically
    images: list[ProductImage] # List ya images zenye validation kamili ya URL

@app.post("/inventory/")
def add_inventory(item: StoreInventory):
    return {
        "status": "success",
        "total_images": len(item.images),
        "data": item.model_dump()
    }
```

User akituma unvalid URL  (kama `"url": "unvalid URL"`), Pydantic itakataa na kusema lazima iwe valid HTTP/HTTPS URL!

---

## 5. Swagger UI Inavyorahisisha Testing 🎮

Ukifungua **`http://127.0.0.1:8000/docs`**, utaona kwamba kwa kila endpoint ya `POST` au `PUT`, Swagger inakupa **JSON Editor iliyo na example data tayari**!

Huna haja ya kuinvent JSON structure kwenye kichwa chako au kufungua Postman:
1. Click endpoint ya `/products/`.
2. Click **"Try it out"**.
3. Badilisha values kwenye JSON text box.
4. Click **"Execute"** ndio uget responses immediately!

---

## System ya Module ya 04 📌
* **Request Body** hutumika kusafirisha JSON payloads kubwa kwa njia ya `POST`, `PUT`, na `PATCH`.
* **Pydantic `BaseModel`** hutumika kuillustrate muundo (schema) na sheria za data validation.
* **`Field(gt=0, ...)`** inawezesha kuweka constraints za ziada kama namba kuwa negative au urefu wa string.
* FastAPI inatambua automatically kama variable ni Path Parameter, Query Parameter, au Request Body.
* **Nested Models** zinawezesha kuwakilisha mifumo changamano ya data kwa usafi.

Kwa module inayofuata, tutaangalia vile ya kurudisha **HTTP Status Codes sahihi** na kutumia **Response Models** for data security zako!

👉 Endelea na **[Module 05: Status Codes & Response Models](05_status_codes_and_response_models.md)**.
