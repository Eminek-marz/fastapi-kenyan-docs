# Moduli 06: Error Handling & HTTPException 🚨

<span class="kenyan-tag">Moduli 06</span>

Kwa real world situation ya programming, mambo huwa hayaendi sawa kila wakati:
* Client anaweza kuomba product ambayo haiko kwa database (haipatikani).
* Mtu anaweza kujaribu kutoa pesa nyingi kuliko enye iko M-Pesa.
* User anaweza kujaribu kuingia kwa page ya admin bila ruhusa.

Ukitegemea Python ya kawaida bila ku-handle errors vizuri, server yako itarusha `IndexError` au `KeyError` na kumrudishia client **500 Internal Server Error** — kitu ambacho kinaonekana vibaya sana na kinaweza kuvujisha siri za server yako (stack trace leakage).

Katika module hii, tutajifunza venye tunaeza handle errors kama experts kwa kutumia **`HTTPException`**.

---

## 1. Jinsi ya Ku-raise `HTTPException` 🛑

FastAPI inatupa class inayoitwa **`HTTPException`**. Unapotaka kusimamisha execution ya code na kumwambia client kosa lililopo, unatumia hii python keyword: **`raise`**.

```python
from fastapi import FastAPI, HTTPException, status

app = FastAPI()

# Database ya mifano:
items_database = {
    1: {"name": "Laptop ya Dell Latitude", "price": 45000, "stock": 4},
    2: {"name": "Mouse ya HP", "price": 1200, "stock": 0},
    3: {"name": "Simu ya Tecno Camon", "price": 28000, "stock": 10},
}

@app.get("/items/{item_id}")
def read_item(item_id: int):
    # Ikiwa item haipo kwa database:
    if item_id not in items_database:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Samahani, product enye ID namba {item_id} haijapatikana kwa mfumo."
        )
        
    return items_database[item_id]
```

### Jaribu Kwenye Browser:
1. Ukienda: `http://127.0.0.1:8000/items/1` $\rightarrow$ Utapata information ya Dell Laptop na status `200 OK`.
2. Ukienda: `http://127.0.0.1:8000/items/99` $\rightarrow$ FastAPI inasimamisha kila kitu na kumrudishia client JSON safi:

```json
{
  "detail": "Samahani, product yenye ID namba 99 haijapatikana kwa mfumo."
}
```
Na status code ya response inakuwa **`404 Not Found`** badala ya crash ya 500!

---

## 2. Kuweka Custom Headers Kwenye Kosa 🏷️

Wakati mwingine (hasa kwenye issue ya authentication na security), unahitaji kuambatanisha custom HTTP header kwa error response ili kumpa client maelekezo zaidi:

```python
@app.get("/premium-content/")
def get_premium(token: str | None = None):
    if token != "nairobi-secret-token":
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Huna idhini ya kutazama maudhui haya. Token yako si sahihi.",
            headers={"WWW-Authenticate": "Bearer", "X-Help-Contact": "support@store.co.ke"}
        )
        
    return {"content": "Siri za mafanikio ya tech Kenya 🇰🇪"}
```

---

## 3. Practical example: Kosa la Salio la Malipo/Payment Balance Error, (Custom Business Logic) 💳

Hebu tuone mfano halisi wa API ya kukata payment ya product:

```python
from fastapi import FastAPI, HTTPException, status
from pydantic import BaseModel

app = FastAPI()

class PaymentRequest(BaseModel):
    account_number: str
    amount: float

# Salio la mfano kwenye akaunti:(Payment balance example)
USER_WALLET_BALANCE = 500.0

@app.post("/checkout/")
def process_checkout(payment: PaymentRequest):
    # Kosa 1: Kiasi hakikubaliki (Chini ya 10 KES):
    if payment.amount < 10.0:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Kiasi cha chini kabisa cha malipo ni KES 10.00."
        )

    # Kosa 2: Salio halitoshi:
    if payment.amount > USER_WALLET_BALANCE:
        raise HTTPException(
            status_code=status.HTTP_402_PAYMENT_REQUIRED,
            detail={
                "error": "INSUFFICIENT_FUNDS",
                "message": "Salio lako halitoshi kukamilisha muamala huu.",
                "current_balance": USER_WALLET_BALANCE,
                "attempted_amount": payment.amount,
                "deficit": payment.amount - USER_WALLET_BALANCE
            }
        )

    return {
        "status": "SUCCESS",
        "message": f"Malipo ya KES {payment.amount} yamekamilika!",
        "new_balance": USER_WALLET_BALANCE - payment.amount
    }
```

> [!TIP]
> Angalia jinsi tulivyoweza kuweka **dictionary nzima** ndani ya parameter ya `detail=`! FastAPI inaiweka dictionary hiyo moja kwa moja kwenye JSON ya error, ikimwezesha frontend developer kuonyesha ujumbe wa kina kwa mtumiaji.

---

## 4. Custom Global Exception Handlers 🌐

Ikiwa project yako inatumia custom Python exceptions (mfano: `MpesaConnectionError` au `ItemOutOfStockException`), unaweza kuziambia FastAPI jinsi ya kuzigeuza kuwa HTTP responses safi kwa kutumia `@app.exception_handler()`:

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI()

# 1. Custom Exception yetu ya Python:
class ItemOutOfStockException(Exception):
    def __init__(self, item_name: str):
        self.item_name = item_name

# 2. Handler inayoikamata exception hiyo kila inapotokea popote kwenye app:
@app.exception_handler(ItemOutOfStockException)
def out_of_stock_handler(request: Request, exc: ItemOutOfStockException):
    return JSONResponse(
        status_code=400,
        content={
            "error_type": "OUT_OF_STOCK",
            "message": f"Poleni sana! Bidhaa ya '{exc.item_name}' imeisha stoo kwa sasa."
        }
    )

# 3. Kwenye endpoint yoyote, una-raise tu hiyo exception:
@app.get("/buy/{name}")
def buy_item(name: str):
    if name.lower() == "iphone":
        raise ItemOutOfStockException(item_name="iPhone 15 Pro")
        
    return {"message": f"Umenunua {name}!"}
```

---

## Summary ya Module ya 06 📌
* Usiruhusu server yako kurusha raw Python exceptions zinazosababisha **500 Internal Server Error**.
* Tumia **`raise HTTPException(status_code=..., detail=...)`** kuterminate process na kurudisha ujumbe uliopangwa(pre-configured message).
* Parameter ya `detail=` inaweza kubeba string rahisi au hata dictionary kamili yenye error statistics.
* Tumia **`@app.exception_handler`** kubadilisha custom Python exceptions kuwa JSON responses safi kwa entire system.

Kwa module inayofuata, tutaingia kwa most powerful feature katika FastAPI: **Dependency Injection (`Depends`)**!

👉 Endelea na **[Moduli 07: Dependency Injection (`Depends`)](07_dependency_injection.md)**.
