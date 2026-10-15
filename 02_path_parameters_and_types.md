# Moduli 02: Path Parameters & Type Hints 🎯

<span class="kenyan-tag">Moduli 02</span>

Kwa REST APIs, mara nyingi unahitaji kupata information ya kitu maalum kulingana na ID Yake (ID). Mfano: unataka kuget information ya product yenye ID namba `5`, au kusearch user mwenye username `"eminek"`.

Kwa URL, hiyo ID inaitwa **Path Parameter**. Kwa module hii, tutaona venye FastAPI inavyotumia **Python Type Hints** kufanya data validation automatically na kuzuia bugs!

---

## 1.Primary Path Parameter Ni Nini? 🔍

Hebu tuongeze endpoint ya kutafuta product kulingana na ID yake:

```python
from fastapi import FastAPI

app = FastAPI()

# {item_id} hapa ni path parameter:
@app.get("/items/{item_id}")
def get_item(item_id: int):
    return {
        "item_id": item_id,
        "type_of_id": str(type(item_id)), # Tuthibitishe aina ya data
        "message": f"Taarifa za bidhaa namba {item_id} zimepatikana."
    }
```

### Angalia Maajabu ya Hapa:
* Kwa route, tunaweka curly braces ya `{item_id}`.
* Kwa parameter ya function, tunaandika: `item_id: int`.
* Hiyo `: int` inaitwa **Python Type Hint**. Inaambia FastAPI: *"Huyu mtu anayeweka hii ID, hakikisha ni integer."*

Sasa jaribu kufungua URL hii kwenye browser yako:  
👉 **`http://127.0.0.1:8000/items/254`**

Utapata:
```json
{
  "item_id": 254,
  "type_of_id": "<class 'int'>",
  "message": "Taarifa za bidhaa namba 254 zimepatikana."
}
```

> [!NOTE]
> Ingawa URL kwa web huwa zinasafiri kama **text/string** pekee, FastAPI imechukua ile `"254"` na kuigeuza (type conversion) kuwa Python `int` automatically kabla hata haijafika ndani ya function yako!

---

## 2. Nini Kinatokea Mtu Akituma Data Mbaya? (Automatic 422 Error) 🛑

Sasa jaribu kuweka neno badala ya namba:  
👉 **`http://127.0.0.1:8000/items/nairobi`**

Kwenye frameworks zingine za zamani, server ingeweza ku-crash na kutoa `ValueError` au `500 Internal Server Error`. Lakini angalia FastAPI inavyokataa mara moja na response ya **HTTP 422 (Unprocessable Entity)**:

```json
{
  "detail": [
    {
      "type": "int_parsing",
      "loc": [
        "path",
        "item_id"
      ],
      "msg": "Input should be a valid integer, unable to parse string as an integer",
      "input": "nairobi"
    }
  ]
}
```

### Kwanini Hii Ni Ushindi Mkubwa Kwako?
1. Hukuhitaji kuandika `if not isinstance(item_id, int): ...`.
2. Error message inamwambia client waziwazi: *"Kwenye path, parameter ya item_id haikuweza kugeuzwa kuwa integer kwa sababu ulituma 'nairobi'."*
3. Database yako inalindwa kutokana na data chafu kabla hata code yako haijafanya query yoyote!

---

## 3. Mpangilio wa Routes Ni Muhimu (Order Matters!) ⚠️

Fikiria una endpoint mbili: moja inaleta information za profile ya mtu mwenye ako logged in (`/users/me`), na nyingine inaleta user yeyote kwa ID yake (`/users/{user_id}`).

**Angalia hii example:**

```python
from fastapi import FastAPI

app = FastAPI()

# ❌ KOSA LA KAWAIDA:
@app.get("/users/{user_id}")
def get_user_by_id(user_id: str):
    return {"user_id": user_id, "mode": "dynamic"}

@app.get("/users/me")
def get_current_user():
    return {"username": "current_user", "mode": "static"}
```

Ukipanga hivi, ukienda kwa **`/users/me`**, FastAPI itasoma route ya kwanza (`/users/{user_id}`) na kudhani neno `"me"` ni `user_id` ya mtu! Kwa hivyo `get_current_user` haitawahi .

### ✅ Better way: Weka Static Routes Kwanza, Dynamic Baadaye:
```python
# 1. Weka static route kwanza:
@app.get("/users/me")
def get_current_user():
    return {"username": "current_user", "mode": "static"}

# 2. Kisha weka dynamic route yenye parameter:
@app.get("/users/{user_id}")
def get_user_by_id(user_id: str):
    return {"user_id": user_id, "mode": "dynamic"}
```

---

## 4. Kudhibiti Chaguo Maalum kwa Kutumia `Enum` 🎛️
What if unataka path parameter ikubali tu specific list of words (mfano: payment methods kama `mpesa`, `airtel`, au `card`), na ireject other words entirely?

Hapa tunatumia standard Python **`Enum`**:

```python
from enum import Enum
from fastapi import FastAPI

class PaymentMethod(str, Enum):
    mpesa = "mpesa"
    airtel = "airtel"
    card = "card"

app = FastAPI()

@app.get("/payments/{method}")
def process_payment(method: PaymentMethod):
    if method is PaymentMethod.mpesa:
        return {"payment_gateway": "Daraja API", "provider": "Safaricom M-Pesa 🟢"}
    
    if method == PaymentMethod.airtel:
        return {"payment_gateway": "Airtel Money API", "provider": "Airtel 🔴"}
        
    return {"payment_gateway": "Stripe/Visa", "provider": "Card Payment 💳"}
```

### Importance Zake Kwenye Swagger UI (`/docs`):
FastAPI inajua ulichotumia ni `Enum`! Kwa hivyo ukifungua **/docs**, badala ya kuonyesha sanduku tupu la kuandika, inampa user **Dropdown Menu** enye choices tatu tu (`mpesa`, `airtel`, `card`). So user hawezi kuandika neno lililo nje ya menu hiyo!

---

## Summary ya Module ya 02 📌
* **Path Parameters** huwekwa ndani ya alama za `{param_name}` kwenye route URL.
* **Type Hints** (`item_id: int`) huwezesha FastAPI kugeuza aina ya data (type conversion) na kukataa data mbaya automatically (validation).
* Data ikiwa na kosa, user anapata **HTTP 422 Unprocessable Entity** yenye maelezo kamili ya kosa lilipo.
* Weka **static routes** (kama `/users/me`) kabla ya **parameterized routes** (kama `/users/{user_id}`).
* Tumia **Enum** kuunda predefined valid options.

Kwa module inayofuata, tutaangalia venye tutashughulikia **Query Parameters** (kama vile kufanya pagination, searches, na sorting za data)!

👉 Endelea na **[Moduli 03: Query Parameters & Filtering](03_query_parameters.md)**.
