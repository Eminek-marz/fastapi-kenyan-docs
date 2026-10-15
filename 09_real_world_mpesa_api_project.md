# Moduli 09: Mradi wa Vitendo — M-Pesa Callback & Store API 🇰🇪📱

<span class="kenyan-tag">Moduli 09: Capstone Project</span>

Hongera sana kwa kufika moduli ya mwisho ya mwongozo huu! Sasa tutaweka maarifa yote tuliyojifunza pamoja (Path parameters, Query filtering, Pydantic validation, Status codes, Error handling, na CORS) ili kujenga **Mradi Kamili wa Vitendo**.

Tutajenga backend ya duka la kisasa lenye:
1. **CRUD kamili ya Bidhaa za Duka (Products API)**.
2. **Safaricom Daraja M-Pesa Callback Webhook Endpoint** — jinsi ya kupokea, kuthibitisha, na kusoma taarifa za malipo ya STK Push kiotomatiki wakati mteja anapolipia bidhaa kwa M-Pesa!

---

## 1. Muundo wa Code Kamili (`main.py`) 📜

Tengeneza file lako la `main.py` na uweke code hii kamili. Ni runnable na haina external database dependencies:

```python
from enum import Enum
from typing import Any
from fastapi import FastAPI, HTTPException, Query, status
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel, Field

# ==========================================
# 1. SETUP YA FASTAPI & CORS
# ==========================================
app = FastAPI(
    title="Nairobi Hub & M-Pesa Store API 🇰🇪",
    description="Backend kamili iliyojengwa kwa ajili ya ma-developer wa Kenya bila deep translation.",
    version="1.0.0"
)

# Washa CORS ili frontend zote ziwasiliane na API hii:
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# ==========================================
# 2. MODELS & SCHEMAS ZA PYDANTIC
# ==========================================
class ProductCategory(str, Enum):
    electronics = "electronics"
    clothing = "clothing"
    food = "food"
    accessories = "accessories"

class ProductCreate(BaseModel):
    name: str = Field(min_length=2, max_length=50, examples=["Simu ya Infinix Hot 40"])
    description: str | None = Field(default=None, examples=["Simu safi ya 128GB"])
    price: float = Field(gt=0, description="Bei lazima izidi sifuri", examples=[18500.0])
    category: ProductCategory
    in_stock: bool = True

class ProductResponse(ProductCreate):
    id: int

# Schema za Kusoma Safaricom M-Pesa Callback JSON:
class CallbackMetadataItem(BaseModel):
    Name: str
    Value: Any = None

class CallbackMetadata(BaseModel):
    Item: list[CallbackMetadataItem]

class StkCallback(BaseModel):
    MerchantRequestID: str
    CheckoutRequestID: str
    ResultCode: int
    ResultDesc: str
    CallbackMetadata: CallbackMetadata | None = None

class MpesaCallbackPayload(BaseModel):
    Body: dict[str, StkCallback]

# ==========================================
# 3. DATABASE YA MFANO (IN-MEMORY STORAGE)
# ==========================================
products_db: dict[int, dict] = {
    1: {"id": 1, "name": "Bata Safari Boots", "description": "Classic Kenyan leather", "price": 4200.0, "category": "clothing", "in_stock": True},
    2: {"id": 2, "name": "Oraimo 20000mAh Powerbank", "description": "Fast charging", "price": 2800.0, "category": "electronics", "in_stock": True},
    3: {"id": 3, "name": "Kahawa ya Dormans 500g", "description": "Original Kenyan Coffee", "price": 850.0, "category": "food", "in_stock": True},
}

mpesa_transactions_db: list[dict] = []

# ==========================================
# 4. CRUD ENDPOINTS ZA BIDHAA (PRODUCTS)
# ==========================================

@app.get("/api/products/", response_model=list[ProductResponse], tags=["Products"])
def list_products(
    skip: int = Query(default=0, ge=0, description="Idadi ya kuruka"),
    limit: int = Query(default=10, le=50, description="Kiwango cha juu cha bidhaa kwa ukurasa"),
    category: ProductCategory | None = Query(default=None, description="Chuja kwa category")
):
    """
    Soma orodha ya bidhaa zote kwa pagination na category filtering.
    """
    all_items = list(products_db.values())
    if category:
        all_items = [item for item in all_items if item["category"] == category.value]
    return all_items[skip : skip + limit]

@app.get("/api/products/{product_id}", response_model=ProductResponse, tags=["Products"])
def get_product(product_id: int):
    """
    Pata taarifa za bidhaa maalum kulingana na ID yake.
    """
    if product_id not in products_db:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Bidhaa namba {product_id} haijapatikana."
        )
    return products_db[product_id]

@app.post("/api/products/", response_model=ProductResponse, status_code=status.HTTP_201_CREATED, tags=["Products"])
def create_product(product: ProductCreate):
    """
    Unda bidhaa mpya kwenye duka.
    """
    new_id = max(products_db.keys(), default=0) + 1
    product_data = product.model_dump()
    product_data["id"] = new_id
    products_db[new_id] = product_data
    return product_data

@app.delete("/api/products/{product_id}", status_code=status.HTTP_204_NO_CONTENT, tags=["Products"])
def delete_product(product_id: int):
    """
    Futa bidhaa kutoka kwenye duka.
    """
    if product_id not in products_db:
        raise HTTPException(status_code=404, detail="Bidhaa haipo")
    del products_db[product_id]
    return None

# ==========================================
# 5. M-PESA DARAJA WEBHOOK CALLBACK ENDPOINT
# ==========================================

@app.post("/api/v1/payments/mpesa-callback", tags=["M-Pesa Webhook"])
def mpesa_payment_callback(payload: dict):
    """
    Endpoint hii inaitwa na Safaricom Daraja API mara tu mteja anapoweka PIN yake ya M-Pesa.
    """
    print("\n🟢 [M-PESA WEBHOOK RECEIVED] Safaricom imetuma taarifa za muamala:")
    
    stk_callback = payload.get("Body", {}).get("stkCallback", {})
    result_code = stk_callback.get("ResultCode")
    result_desc = stk_callback.get("ResultDesc")
    checkout_id = stk_callback.get("CheckoutRequestID")

    # ResultCode 0 inamaanisha mteja alilipa kwa mafanikio:
    if result_code == 0:
        meta_items = stk_callback.get("CallbackMetadata", {}).get("Item", [])
        
        # Geuza items za Safaricom kuwa dictionary safi:
        payment_info = {item["Name"]: item.get("Value") for item in meta_items}
        
        receipt_number = payment_info.get("MpesaReceiptNumber")
        amount = payment_info.get("Amount")
        phone = payment_info.get("PhoneNumber")
        
        record = {
            "status": "COMPLETED",
            "receipt": receipt_number,
            "amount": amount,
            "phone": phone,
            "checkout_id": checkout_id
        }
        mpesa_transactions_db.append(record)
        
        print(f"💰 Malipo Yamethibitishwa! Receipt: {receipt_number} | KES {amount} kutoka {phone}")
        
    else:
        # ResultCode != 0 inamaanisha mteja ali-cancel au salio halitoshi:
        print(f"❌ Muamala Umefeli/Kukataliwa: {result_desc} (Code: {result_code})")
        mpesa_transactions_db.append({
            "status": "FAILED",
            "reason": result_desc,
            "checkout_id": checkout_id
        })

    # Safaricom inahitaji majibu ya haraka ya 'ResultCode: 0' ili ijue umepokea taarifa:
    return {
        "ResultCode": 0,
        "ResultDesc": "Callback processed successfully by Nairobi Hub"
    }

@app.get("/api/v1/payments/transactions", tags=["M-Pesa Webhook"])
def get_recorded_transactions():
    """
    Onyesha miamala yote ya M-Pesa iliyorekodiwa live.
    """
    return {
        "total_transactions": len(mpesa_transactions_db),
        "transactions": mpesa_transactions_db
    }
```

---

## 2. Jinsi ya Ku-run na Ku-test Project Hii 🚀

### Hatua ya 1: Washa Server
Kwenye terminal yako:
```bash
uvicorn main:app --reload
```

### Hatua ya 2: Fungua Swagger UI
Fungua: **`http://127.0.0.1:8000/docs`**

Utapata dashboard safi sana yenye makundi mawili ya tags:
* **`Products`**: Ambapo unaweza ku-test `GET`, `POST`, na `DELETE` za bidhaa.
* **`M-Pesa Webhook`**: Ambapo unaweza ku-simulate jinsi Safaricom inavyotuma taarifa!

---

## 3. Ku-simulate M-Pesa STK Push Callback 📱

Ili kuona jinsi webhook inavyofanya kazi, chagua endpoint ya `POST /api/v1/payments/mpesa-callback`, bonyeza **"Try it out"**, na ubandike mfano halisi wa JSON inayotumwa na Safaricom Daraja:

```json
{
  "Body": {
    "stkCallback": {
      "MerchantRequestID": "29115-34620561-1",
      "CheckoutRequestID": "ws_CO_19122026102511234567",
      "ResultCode": 0,
      "ResultDesc": "The service request is processed successfully.",
      "CallbackMetadata": {
        "Item": [
          { "Name": "Amount", "Value": 4200.0 },
          { "Name": "MpesaReceiptNumber", "Value": "SDT45KL89Q" },
          { "Name": "TransactionDate", "Value": 20260924012015 },
          { "Name": "PhoneNumber", "Value": 254712345678 }
        ]
      }
    }
  }
}
```

Bonyeza **"Execute"**:
1. Server yako itajibu mara moja na:
   ```json
   {
     "ResultCode": 0,
     "ResultDesc": "Callback processed successfully by Nairobi Hub"
   }
   ```
2. Angalia terminal yako ya Uvicorn, utaona log ya moja kwa moja:
   ```text
   🟢 [M-PESA WEBHOOK RECEIVED] Safaricom imetuma taarifa za muamala:
   💰 Malipo Yamethibitishwa! Receipt: SDT45KL89Q | KES 4200.0 kutoka 254712345678
   ```
3. Sasa nenda kwa `GET /api/v1/payments/transactions`, utaona muamala huo ukiwa umehifadhiwa kikamilifu!

---

## Hongera Sana! 🎓🇰🇪

Umefanikiwa kukamilisha mwongozo mzima wa **FastAPI (Toleo la Kiswahili cha Kenya bila Deep Translation)**! 

Sasa una ujuzi thabiti wa:
- Kuunda scalable REST APIs kwa kutumia Python.
- Kutumia Pydantic kwa ajili ya data integrity na automatic error reporting.
- Kuzuia usalama wa data kwa kutumia `response_model` na `HTTPException`.
- Kuunganisha frontend kupitia CORS.
- Kujenga production-ready webhooks za mifumo halisi ya kifedha kama Safaricom M-Pesa.

Uko tayari kujenga backends kubwa na zenye tija kwa jamii yetu na ulimwengu mzima! 🚀🔥
