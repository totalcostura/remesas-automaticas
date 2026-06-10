import requests
import json
from datetime import datetime

# Configuración de ganancia (restar 5%)
GANANCIA = 0.95

def obtener_precio_p2p(fiat, trade_type, pay_types=None, trans_amount=10):
    """
    trade_type: 'BUY' (comprar USDT con fiat) o 'SELL' (vender USDT por fiat)
    pay_types: lista de strings, ej. ['pago movil']
    """
    url = "https://p2p.binance.com/bapi/c2c/v2/friendly/c2c/adv/search"
    headers = {
        "Content-Type": "application/json",
        "User-Agent": "Mozilla/5.0"
    }
    payload = {
        "asset": "USDT",
        "fiat": fiat,
        "tradeType": trade_type,
        "page": 1,
        "rows": 1,
        "payTypes": pay_types if pay_types else [],
        "transAmount": str(trans_amount)
    }
    try:
        response = requests.post(url, headers=headers, json=payload, timeout=15)
        data = response.json()
        if data.get("code") == "000000" and data.get("data"):
            precio = float(data["data"][0]["adv"]["price"])
            return precio
        else:
            return None
    except Exception as e:
        print(f"Error con {fiat} {trade_type}: {e}")
        return None

def main():
    print("Obteniendo precios P2P de Binance...")
    
    # Venezuela: venta USDT con pago móvil, monto 10 USDT
    p_ve_venta = obtener_precio_p2p("VES", "SELL", ["pago movil"], 10)
    # Chile: compra USDT sin filtro, monto 10 USDT
    p_cl_compra = obtener_precio_p2p("CLP", "BUY", None, 10)
    # USA: compra USD sin filtro
    p_usa_compra = obtener_precio_p2p("USD", "BUY", None, 10)
    # España: compra EUR con BBVA, monto 30
    p_esp_compra = obtener_precio_p2p("EUR", "BUY", ["BBVA"], 30)
    # México: compra MXN con Santander, monto 30
    p_mex_compra = obtener_precio_p2p("MXN", "BUY", ["Santander"], 30)
    
    resultados = []
    fecha_actual = datetime.now().strftime("%A %d de %B %Y")
    
    if p_ve_venta and p_cl_compra:
        tasa_chile_ve = (p_ve_venta / p_cl_compra) * GANANCIA
        resultados.append(f"Chile → Venezuela: {tasa_chile_ve:.4f} VES/CLP")
    
    if p_ve_venta and p_usa_compra:
        tasa_usa_ve = (p_ve_venta / p_usa_compra) * GANANCIA
        resultados.append(f"EE. UU. → Venezuela: {tasa_usa_ve:.0f} VES/USD")
    
    if p_ve_venta and p_esp_compra:
        tasa_esp_ve = (p_ve_venta / p_esp_compra) * GANANCIA
        resultados.append(f"España → Venezuela: {tasa_esp_ve:.0f} VES/EUR")
    
    if p_ve_venta and p_mex_compra:
        tasa_mex_ve = (p_ve_venta / p_mex_compra) * GANANCIA
        resultados.append(f"México → Venezuela: {tasa_mex_ve:.0f} VES/MXN")
    
    # Escribir resultados a un archivo
    with open("tasas.txt", "w", encoding="utf-8") as f:
        f.write("# REMESAS VINOTINTO\n")
        f.write("# +56 9 3657 2481\n\n")
        for linea in resultados:
            f.write(linea + "\n")
        f.write("\n# ENVÍO DE DINERO\n")
        f.write("# TASAS DE CAMBIO ACTUALIZADAS\n")
        f.write(f"# {fecha_actual}\n")
    
    print("Archivo tasas.txt generado correctamente.")

if __name__ == "__main__":
    main()
