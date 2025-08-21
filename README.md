import os
import json
from datetime import datetime

DATA_FILE = "shop_data.json"

def load_data():
    if os.path.exists(DATA_FILE):
        with open(DATA_FILE, "r") as f:
            return json.load(f)
    return {"products": {}, "sales": []}

def save_data(data):
    with open(DATA_FILE, "w") as f:
        json.dump(data, f, indent=4)

def add_product(data):
    name = input("Product name: ")
    if name in data["products"]:
        print("⚠️ Product already exists.")
        return
    try:
        price = float(input("Price: "))
        stock = int(input("Stock quantity: "))
    except ValueError:
        print("⚠️ Invalid number.")
        return
    data["products"][name] = {"price": price, "stock": stock}
    save_data(data)
    print(f"✅ Product '{name}' added successfully!")

def view_inventory(data):
    if not data["products"]:
        print("📭 No products yet.")
        return
    print("\n📦 Inventory:")
    for name, info in data["products"].items():
        print(f"- {name}: ${info['price']:.2f}, Stock: {info['stock']}")

def record_sale(data):
    if not data["products"]:
        print("⚠️ No products available.")
        return
    view_inventory(data)
    product = input("Enter product name: ")
    if product not in data["products"]:
        print("⚠️ Product not found.")
        return
    try:
        qty = int(input("Quantity sold: "))
    except ValueError:
        print("⚠️ Invalid quantity.")
        return
    if qty > data["products"][product]["stock"]:
        print("⚠️ Not enough stock!")
        return
    total = qty * data["products"][product]["price"]
    data["products"][product]["stock"] -= qty
    data["sales"].append({
        "product": product,
        "quantity": qty,
        "total": total,
        "date": str(datetime.today().date())
    })
    save_data(data)
    print(f"✅ Sale recorded! Total = ${total:.2f}")

def daily_summary(data):
    today = str(datetime.today().date())
    today_sales = [s for s in data["sales"] if s["date"] == today]
    if not today_sales:
        print("📭 No sales today.")
        return
    total_income = sum(s["total"] for s in today_sales)
    print(f"\n📅 Sales Summary for {today}")
    print(f"💰 Total Income: ${total_income:.2f}")
    product_totals = {}
    for s in today_sales:
        product_totals[s["product"]] = product_totals.get(s["product"], 0) + s["quantity"]
    print("\n🔥 Top Selling Products:")
    for p, q in sorted(product_totals.items(), key=lambda x: x[1], reverse=True):
        print(f"- {p}: {q} sold")

def menu():
    data = load_data()
    while True:
        print("\n==== ONLINE SHOP MANAGER ====")
        print("1. Add Product")
        print("2. View Inventory")
        print("3. Record Sale")
        print("4. Daily Sales Summary")
        print("5. Exit")
        choice = input("Choose an option: ")

        if choice == "1":
            add_product(data)
        elif choice == "2":
            view_inventory(data)
        elif choice == "3":
            record_sale(data)
        elif choice == "4":
            daily_summary(data)
        elif choice == "5":
            break
        else:
            print("⚠️ Invalid choice.")

if __name__ == "__main__":
    menu()
