@app.route('/')
def index():
    menu = '''
    <h1>brukerinformasjon</h1>
    <ul>
        <li><a href="/vis_klasse">Vis klasser</a></li>
        <li><a href="/leggtil_klasse">Legg til klasse</a></li>
        <li><a href="/slett_klasse">Slett klasse</a></li>
        <li><br></li>
        <li><a href="/vis_student">Vis studenter</a></li>
        <li><a href="/leggtil_student">Legg til student</a></li>
        <li><a href="/slett_student">Slett student</a></li>
    </ul>
    '''
    return render_template_string(menu)

# --- VIS KLASSE ---
@app.route('/vis_klasse')
def vis_klasse():
    con = sqlite3.connect("skole.db")
    cur = con.cursor()
    cur.execute("SELECT * FROM klasse")
    klasser = cur.fetchall()
    con.close()

    html = "<h2>Klasser</h2><table border=1><tr><th>Kode</th><th>Navn</th><th>Studium</th></tr>"
    for k in klasser:
        html += f"<tr><td>{k[0]}</td><td>{k[1]}</td><td>{k[2]}</td></tr>"
    html += "</table><a href='/'>Tilbake</a>"
    return render_template_string(html)

# --- LEGG TIL KLASSE ---
@app.route('/leggtil_klasse', methods=['GET', 'POST'])
def leggtil_klasse():
    if request.method == 'POST':
        kode = request.form['klassekode']
        navn = request.form['klassenavn']
        studie = request.form['studiumkode']
        con = sqlite3.connect("skole.db")
        cur = con.cursor()
        cur.execute("INSERT INTO klasse VALUES (?, ?, ?)", (kode, navn, studie))
        con.commit()
        con.close()
        return redirect('/')
    form = '''
    <h2>Legg til klasse</h2>
    <form method="post">
        Kode: <input name="klassekode"><br>
        Navn: <input name="klassenavn"><br>
        Studiumkode: <input name="studiumkode"><br>
        <input type="submit" value="Lagre">
    </form>
    <a href="/">Tilbake</a>
    '''
    return render_template_string(form)

# --- SLETT KLASSE ---
@app.route('/slett_klasse', methods=['GET','POST'])
def slett_klasse():
    con = sqlite3.connect("skole.db")
    cur = con.cursor()
    cur.execute("SELECT klassekode FROM klasse")
    klasser = cur.fetchall()
    if request.method == 'POST':
        kode = request.form['klassekode']
        cur.execute("DELETE FROM klasse WHERE klassekode=?", (kode,))
        con.commit()
        con.close()
        return redirect('/')
    con.close()
    options = "".join([f"<option value='{k[0]}'>{k[0]}</option>" for k in klasser])
    form = f'''
    <h2>Slett klasse</h2>
    <form method="post">
        <select name="klassekode">{options}</select>
        <input type="submit" value="Slett">
    </form>
    <a href="/">Tilbake</a>
    '''
    return render_template_string(form)

# --- VIS STUDENT ---
@app.route('/vis_student')
def vis_student():
    con = sqlite3.connect("skole.db")
    cur = con.cursor()
    cur.execute("SELECT * FROM student")
    studenter = cur.fetchall()
    con.close()
    html = "<h2>Studenter</h2><table border=1><tr><th>Brukernavn</th><th>Fornavn</th><th>Etternavn</th><th>Klasse</th></tr>"
    for s in studenter:
        html += f"<tr><td>{s[0]}</td><td>{s[1]}</td><td>{s[2]}</td><td>{s[3]}</td></tr>"
    html += "</table><a href='/'>Tilbake</a>"
    return render_template_string(html)

# --- START APP ---
if __name__ == '__main__':
    app.run(debug=True)
