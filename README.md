from fastapi import FastAPI, Form, Query
from fastapi.responses import HTMLResponse, RedirectResponse

app = FastAPI()

sklad = {
    "ovocie": {
        "jablko": {"cena": 0.80, "mnozstvo": 3},
        "banan": {"cena": 1.50, "mnozstvo": 2},
        "hruska": {"cena": 1.20, "mnozstvo": 5},
        "marhula": {"cena": 2.00, "mnozstvo": 4},
        "slivka": {"cena": 1.80, "mnozstvo": 6}
    },
    "zelenina": {
        "mrkva": {"cena": 0.60, "mnozstvo": 10},
        "petrzlen": {"cena": 0.90, "mnozstvo": 8},
        "celer": {"cena": 1.10, "mnozstvo": 3},
        "zemiak": {"cena": 0.70, "mnozstvo": 15}
    },
    "sladkosti": {
        "cokolada": {"cena": 1.50, "mnozstvo": 5},
        "cukor": {"cena": 1.20, "mnozstvo": 10}
    },
    "mliecne": {
        "mlieko": {"cena": 1.00, "mnozstvo": 4},
        "jogurt": {"cena": 0.60, "mnozstvo": 6},
        "maslo": {"cena": 2.50, "mnozstvo": 2},
        "syr": {"cena": 2.00, "mnozstvo": 3}
    },
    "pecivo": {
        "chlieb": {"cena": 1.80, "mnozstvo": 3},
        "rohlik": {"cena": 0.15, "mnozstvo": 20},
        "bageta": {"cena": 0.90, "mnozstvo": 5}
    }
}

# Garantované fotky kategórií (čisté jedlo)
kategoria_obrazky = {
    "ovocie": "https://images.unsplash.com/photo-1619566636858-adf3ef46400b?w=600&q=80",
    "zelenina": "https://images.unsplash.com/photo-1540420773420-3366772f4999?w=600&q=80",
    "sladkosti": "https://images.unsplash.com/photo-1582058091505-f87a2e55a40f?w=600&q=80",
    "mliecne": "https://images.unsplash.com/photo-1628088062854-d1870b4553da?w=600&q=80",
    "pecivo": "https://images.unsplash.com/photo-1509440159596-0249088772ff?w=600&q=80"
}

# Garantované fotky produktov
produkt_obrazky = {
    "jablko": "https://images.unsplash.com/photo-1560806887-1e4cd0b6cbd6?w=500&auto=format&fit=crop&q=60",
    "banan": "https://images.unsplash.com/photo-1571771894821-ce9b6c11b08e?w=500&auto=format&fit=crop&q=60",
    "hruska": "https://images.pexels.com/photos/12811976/pexels-photo-12811976.jpeg",
    "marhula": "https://www.zahradnici.sk/image/cache/catalog/marhule/marhula%20paviot-cr-1300x1300.jpg",
    "slivka": "https://www.zahradnici.sk/image/cache/catalog/tophit-prunus-domestica-cr-1300x1300.jpg",
    
    "mrkva": "https://images.unsplash.com/photo-1598170845058-32b9d6a5da37?w=500&auto=format&fit=crop&q=60",
    "petrzlen": "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQs1NS0kVCPopVaysYXQbVQod5fnnq6pDs6KBbA5eOlqUoMSJiJROVB4nM&s=10",
    "celer": "https://images.unsplash.com/photo-1615485290382-441e4d049cb5?w=500&auto=format&fit=crop&q=60",
    "zemiak": "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSzPNVycLi7NtT1Q60GfnOGtlkQQJfq45HVc7lddLy9Q8PIAFxSf9L_n4Tz&s=10",

    
    "cokolada": "https://images.unsplash.com/photo-1549007994-cb92caebd54b?w=500&auto=format&fit=crop&q=60",
    "cukor": "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQzO2EO1Uts0pPOAWRoh_B6wi6s25AwtA5X8W2QJHwEMcCE-VRtSGj2FoU&s=10",
    
    "mlieko": "https://images.unsplash.com/photo-1563636619-e9143da7973b?w=500&auto=format&fit=crop&q=60",
    "jogurt": "https://images.unsplash.com/photo-1488477181946-6428a0291777?w=500&auto=format&fit=crop&q=60",
    "maslo": "https://images.unsplash.com/photo-1589985270826-4b7bb135bc9d?w=500&auto=format&fit=crop&q=60",
    "syr": "https://images.unsplash.com/photo-1486297678162-eb2a19b0a32d?w=500&auto=format&fit=crop&q=60",
    
    "chlieb": "https://images.unsplash.com/photo-1509440159596-0249088772ff?w=500&auto=format&fit=crop&q=60",
    "rohlik": "https://images.unsplash.com/photo-1549931319-a545dcf3bc73?w=500&auto=format&fit=crop&q=60",
    "bageta": "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTrfB-XwHGWgfMNg963KBjQUvHZBXp6fQOVdlNmFTJDFLAK0guGBXEUowXK&s=10"
}

kosik = {}
applied_coupon = None

zlavove_kupony = {
    "lowzlava": 0.20,
    "midzlava": 0.10,
    "peekzlava": 0.50,
    }

def cash_round(amount: float) -> float:
    return round(round(amount * 20) / 20, 2)

@app.get("/", response_class=HTMLResponse)
def home(kategoria: str = None):
    kosik_html = ""
    celkova_suma = 0.0
    for polozka, mnozstvo in kosik.items():
        cena = 0.0
        found_cat = ""
        for cat, produkty in sklad.items():
            if polozka in produkty:
                cena = produkty[polozka]["cena"]
                found_cat = cat
                break
        
        item_total = cena * mnozstvo
        celkova_suma += item_total
        kosik_html += f"<li style='margin-bottom: 6px; color: #333;'><b>{polozka}</b> <span style='color: #666; font-size: 13px;'>({found_cat})</span> × {mnozstvo} ks = <b>{item_total:.2f} €</b></li>"

    zlava = 0.0
    if applied_coupon and applied_coupon in zlavove_kupony:
        zlava = celkova_suma * zlavove_kupony[applied_coupon]

    cena_po_zlave = celkova_suma - zlava
    final_total = cash_round(cena_po_zlave)

    obsah_html = ""
    
    if not kategoria:
        obsah_html += '<h2 style="color: #1e293b; margin-bottom: 20px; font-weight: 700; font-size: 24px;">Vyberte si kategóriu tovaru:</h2>'
        obsah_html += '<div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(240px, 1fr)); gap: 20px; margin-top: 15px;">'
        for kat, img_url in kategoria_obrazky.items():
            obsah_html += f"""
            <a href="/?kategoria={kat}" style="text-decoration: none;">
                <div style="position: relative; height: 180px; border-radius: 14px; overflow: hidden; background-image: url('{img_url}'); background-size: cover; background-position: center; box-shadow: 0 6px 15px rgba(0,0,0,0.08); transition: transform 0.2s ease, box-shadow 0.2s ease;" onmouseover="this.style.transform='translateY(-6px)'; this.style.boxShadow='0 12px 24px rgba(0,0,0,0.15)';" onmouseout="this.style.transform='translateY(0)'; this.style.boxShadow='0 6px 15px rgba(0,0,0,0.08)';">
                    <div style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: linear-gradient(to bottom, rgba(0,0,0,0.15), rgba(0,0,0,0.75)); display: flex; align-items: flex-end; padding: 20px;">
                        <span style="color: white; font-size: 22px; font-weight: 800; text-transform: uppercase; text-shadow: 1px 2px 5px rgba(0,0,0,0.6); letter-spacing: 0.5px;">{kat}</span>
                    </div>
                </div>
            </a>
            """
        obsah_html += '</div>'
    else:
        if kategoria in sklad:
            banner_img = kategoria_obrazky.get(kategoria, "")
            obsah_html += f'<div style="margin-bottom: 20px;"><a href="/" style="background: #e2e8f0; color: #334155; padding: 10px 18px; text-decoration: none; border-radius: 8px; font-weight: bold; display: inline-block; transition: background 0.2s;" onmouseover="this.style.background=\'#cbd5e1\'" onmouseout="this.style.background=\'#e2e8f0\'">← Späť na výber kategórií</a></div>'
            
            obsah_html += f"""
            <div style="position: relative; height: 200px; border-radius: 14px; overflow: hidden; background-image: url('{banner_img}'); background-size: cover; background-position: center; margin-bottom: 25px; box-shadow: 0 6px 15px rgba(0,0,0,0.1);">
                <div style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: linear-gradient(to right, rgba(15,23,42,0.85), rgba(15,23,42,0.3)); display: flex; align-items: center; padding: 30px;">
                    <h2 style="color: white; text-transform: uppercase; margin: 0; font-size: 32px; font-weight: 800; text-shadow: 2px 2px 6px rgba(0,0,0,0.5); letter-spacing: 1px;">{kategoria}</h2>
                </div>
            </div>
            """
            
            obsah_html += '<div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 20px; margin-top: 15px;">'
            
            for nazov, info in sklad[kategoria].items():
                bg_img = produkt_obrazky.get(nazov, "")
                obsah_html += f"""
                <div style="position: relative; border-radius: 14px; overflow: hidden; background-image: url('{bg_img}'); background-size: cover; background-position: center; background-color: #334155; box-shadow: 0 4px 12px rgba(0,0,0,0.08); display: flex; flex-direction: column; justify-content: space-between; min-height: 220px; transition: transform 0.2s, box-shadow 0.2s;" onmouseover="this.style.transform='translateY(-4px)'; this.style.boxShadow='0 10px 22px rgba(0,0,0,0.15)';" onmouseout="this.style.transform='translateY(0)'; this.style.boxShadow='0 4px 12px rgba(0,0,0,0.08)'">
                    <div style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(15, 23, 42, 0.78); z-index: 1;"></div>
                    
                    <div style="position: relative; z-index: 2; padding: 20px; display: flex; flex-direction: column; justify-content: space-between; height: 100%;">
                        <div>
                            <div style="font-weight: 800; font-size: 20px; color: #ffffff; margin-bottom: 6px; text-transform: capitalize; text-shadow: 1px 1px 3px rgba(0,0,0,0.5);">{nazov}</div>
                            <div style="color: #38bdf8; font-weight: 900; font-size: 22px; margin-bottom: 6px; text-shadow: 1px 1px 3px rgba(0,0,0,0.5);">{info['cena']:.2f} €</div>
                            <div style="font-size: 13px; color: #cbd5e1; margin-bottom: 16px;">Na sklade: <b style="color: #ffffff;">{info['mnozstvo']} ks</b></div>
                        </div>
                        <form action="/add" method="post" style="display: flex; gap: 8px;">
                            <input type="hidden" name="item" value="{nazov}">
                            <input type="hidden" name="kategoria" value="{kategoria}">
                            <input type="number" name="qty" value="1" min="1" max="{info['mnozstvo']}" style="width: 50px; padding: 8px; border: 1px solid rgba(255,255,255,0.3); border-radius: 8px; text-align: center; font-weight: bold; font-size: 14px; background: rgba(255,255,255,0.9); color: #0f172a;">
                            <button type="submit" style="background: #10b981; color: white; border: none; padding: 8px 12px; border-radius: 8px; cursor: pointer; flex-grow: 1; font-weight: bold; font-size: 14px; box-shadow: 0 2px 5px rgba(0,0,0,0.2); transition: background 0.2s;" onmouseover="this.style.background='#059669'" onmouseout="this.style.background='#10b981'">Pridať</button>
                        </form>
                    </div>
                </div>
                """
            obsah_html += '</div>'

    return f"""
    <html>
        <head><title>Matuskoov obchod</title><meta charset="utf-8"></head>
        <body style="font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; margin: 0; padding: 40px 20px; background-color: #f1f5f9;">
            <div style="max-width: 1000px; margin: auto; background: white; padding: 40px; border-radius: 20px; box-shadow: 0 15px 35px rgba(0,0,0,0.06);">
                <h1 style="color: #0f172a; text-align: center; margin-top: 0; font-size: 36px; font-weight: 900; letter-spacing: -0.5px;">🛒 Matuskoov obchod</h1>
                <hr style="border: 0; height: 1px; background: #e2e8f0; margin: 30px 0;">
                
                {obsah_html}

                <div style="background: #f8fafc; border: 2px dashed #cbd5e1; border-radius: 14px; padding: 25px; margin-top: 40px;">
                    <h3 style="color: #1e293b; margin-top: 0; font-size: 20px; font-weight: 700;">🛍️ Váš nákupný košík:</h3>
                    <ul style="padding-left: 20px; margin-bottom: 20px;">{kosik_html if kosik_html else "<li style='color: #94a3b8;'>Košík je zatiaľ prázdny</li>"}</ul>
                    <div style="font-size: 15px; color: #475569; margin-bottom: 6px;">Medzisúčet: <b>{celkova_suma:.2f} €</b></div>
                    <div style="font-size: 15px; color: #475569; margin-bottom: 6px;">Aktívny kupón: <b>{applied_coupon if applied_coupon else 'Žiadny'}</b></div>
                    <div style="font-size: 15px; color: #475569; margin-bottom: 12px;">Ušetrená zľava: <b style="color: #10b981;">-{zlava:.2f} €</b></div>
                    <div style="font-size: 24px; color: #e11d48; margin-bottom: 25px; font-weight: 800;">Zaplať: {final_total:.2f} €</div>
                    
                    <div style="display: flex; gap: 15px; flex-wrap: wrap; align-items: center;">
                        <form action="/apply-coupon" method="post" style="display: flex; gap: 8px;">
                            <input type="text" name="coupon" placeholder="Zadaj kupón (napr. LETOM20)" style="padding: 10px 14px; border: 1px solid #cbd5e1; border-radius: 8px; width: 200px; font-size: 14px;">
                            <button type="submit" style="background: #3b82f6; color: white; border: none; padding: 10px 18px; border-radius: 8px; cursor: pointer; font-weight: bold; transition: background 0.2s;" onmouseover="this.style.background='#2563eb'" onmouseout="this.style.background='#3b82f6'">Použiť kupón</button>
                        </form>
                        <form action="/clear" method="post">
                            <button type="submit" style="background-color: #ef4444; color: white; border: none; padding: 10px 18px; cursor: pointer; border-radius: 8px; font-weight: bold; transition: background 0.2s;" onmouseover="this.style.background='#dc2626'" onmouseout="this.style.background='#ef4444'">Vyprázdniť košík</button>
                        </form>
                    </div>
                </div>
            </div>
        </body>
    </html>
    """

@app.post("/add")
def add_to_cart(item: str = Form(...), kategoria: str = Form(...), qty: int = Form(...)):
    if kategoria in sklad and item in sklad[kategoria]:
        if sklad[kategoria][item]["mnozstvo"] >= qty:
            kosik[item] = kosik.get(item, 0) + qty
            sklad[kategoria][item]["mnozstvo"] -= qty
    return RedirectResponse(url=f"/?kategoria={kategoria}", status_code=303)

@app.post("/apply-coupon")
def apply_coupon(coupon: str = Form(...)):
    global applied_coupon
    kod = coupon.strip().upper()
    if kod in zlavove_kupony:
        applied_coupon = kod
    return RedirectResponse(url="/", status_code=303)

@app.post("/clear")
def clear_cart():
    global kosik, applied_coupon
    for polozka, mnozstvo in kosik.items():
        for kategoria, produkty in sklad.items():
            if polozka in produkty:
                produkty[polozka]["mnozstvo"] += mnozstvo
                break
    kosik = {}
    applied_coupon = None
    return RedirectResponse(url="/", status_code=303)
