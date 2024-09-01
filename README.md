from flask import Flask, request, render_template_string
import sqlite3
from datetime import datetime

app = Flask(__name__)

# 데이터베이스 초기화
def init_db():
    with sqlite3.connect('database.db') as conn:
        c = conn.cursor()
        c.execute('''
            CREATE TABLE IF NOT EXISTS visits (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                ip_address TEXT,
                user_agent TEXT,
                visit_date TEXT
            )
        ''')
        conn.commit()

# 데이터베이스에 방문 기록 저장
def log_visit(ip_address, user_agent, visit_date):
    with sqlite3.connect('database.db') as conn:
        c = conn.cursor()
        c.execute('''
            INSERT INTO visits (ip_address, user_agent, visit_date)
            VALUES (?, ?, ?)
        ''', (ip_address, user_agent, visit_date))
        conn.commit()

# 데이터베이스의 모든 방문 기록을 가져오는 함수
def get_all_visits():
    with sqlite3.connect('database.db') as conn:
        c = conn.cursor()
        c.execute('SELECT * FROM visits')
        return c.fetchall()

@app.route('/')
def home():
    ip_address = request.remote_addr
    user_agent = request.headers.get('User-Agent')
    visit_date = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
    
    log_visit(ip_address, user_agent, visit_date)
    
    return render_template_string('''
        <!doctype html>
        <title>Welcome</title>
        <h1>Welcome to the Flask App</h1>
        <p>Your visit has been recorded.</p>
        <p>IP Address: {{ ip_address }}</p>
        <p>User Agent: {{ user_agent }}</p>
        <p>Visit Date: {{ visit_date }}</p>
    ''', ip_address=ip_address, user_agent=user_agent, visit_date=visit_date)

@app.route('/visits')
def view_visits():
    visits = get_all_visits()
    return render_template_string('''
        <!doctype html>
        <title>Visit Records</title>
        <h1>Visit Records</h1>
        <table border="1">
            <tr>
                <th>ID</th>
                <th>IP Address</th>
                <th>User Agent</th>
                <th>Visit Date</th>
            </tr>
            {% for visit in visits %}
            <tr>
                <td>{{ visit[0] }}</td>
                <td>{{ visit[1] }}</td>
                <td>{{ visit[2] }}</td>
                <td>{{ visit[3] }}</td>
            </tr>
            {% endfor %}
        </table>
        <a href="/">Back to Home</a>
    ''', visits=visits)

if __name__ == '__main__':
    init_db()
    app.run(debug=True)
