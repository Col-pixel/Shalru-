from flask import Flask, render_template, request, redirect, url_for
import sqlite3
import os

app = Flask(__name__)

def init_db():
    with sqlite3.connect('inventory.db') as conn:
        cursor = conn.cursor()
        cursor.execute('''CREATE TABLE IF NOT EXISTS inventory (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            item_name TEXT NOT NULL,
            category TEXT NOT NULL,
            quantity INTEGER NOT NULL,
            price REAL NOT NULL,
            supplier TEXT
        )''')
        conn.commit()

@app.route('/')
def index():
    conn = sqlite3.connect('inventory.db')
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM inventory")
    items = cursor.fetchall()
    conn.close()
    return render_template('index.html', items=items)

@app.route('/add', methods=['POST'])
def add_item():
    item_name = request.form.get('item_name')
    category = request.form.get('category')
    quantity = request.form.get('quantity', type=int)
    price = request.form.get('price', type=float)
    supplier = request.form.get('supplier')

    if item_name and category and quantity is not None and price is not None:
        conn = sqlite3.connect('inventory.db')
        cursor = conn.cursor()
        cursor.execute(
            "INSERT INTO inventory (item_name, category, quantity, price, supplier) VALUES (?, ?, ?, ?, ?)",
            (item_name, category, quantity, price, supplier)
        )
        conn.commit()
        conn.close()
    return redirect(url_for('index'))

@app.route('/delete/<int:item_id>', methods=['POST'])
def delete_item(item_id):
    conn = sqlite3.connect('inventory.db')
    cursor = conn.cursor()
    cursor.execute("DELETE FROM inventory WHERE id=?", (item_id,))
    conn.commit()
    conn.close()
    return redirect(url_for('index'))

if __name__ == '__main__':
    init_db()
    app.run(host='0.0.0.0', port=10000)
