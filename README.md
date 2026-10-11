# Gerador-app
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistema XIT - Gerador & Proxy</title>
    <style>
        :root { --red: #ff0000; --bg: #050505; --card: #111; }
        body { background: var(--bg); color: #fff; font-family: sans-serif; margin: 0; padding: 20px; display: flex; justify-content: center; }
        .box { width: 100%; max-width: 450px; background: var(--card); border: 1px solid #333; padding: 20px; border-radius: 8px; box-shadow: 0 0 10px rgba(255,0,0,0.2); }
        h2 { text-align: center; color: var(--red); text-transform: uppercase; margin-top: 0; }
        input, select { width: 100%; padding: 10px; margin: 5px 0; background: #222; border: 1px solid #444; color: white; box-sizing: border-box; }
        button { width: 100%; padding: 12px; background: var(--red); color: white; border: none; cursor: pointer; font-weight: bold; margin-top: 5px; }
        button:hover { background: #cc0000; }
        .key-box { background: #000; color: var(--red); font-family: monospace; text-align: center; padding: 15px; word-break: break-all; border: 1px dashed #444; margin: 10px 0; }
        .list-item { display: flex; justify-content: space-between; align-items: center; background: #1a1a1a; padding: 10px; margin-bottom: 5px; font-size: 14px; }
        .btn-small { width: auto; padding: 5px 10px; font-size: 12px; margin: 0; }
        .status-valid { color: #0f0; }
        .status-invalid { color: #f00; }
        .hidden { display: none; }
        #proxyResult { text-align: center; font-weight: bold; margin-top: 10px; min-height: 20px; }
    </style>
</head>
<body>

<div class="box">
    <!-- GERADOR -->
    <h2>Gerador</h2>
    <input type="text" id="prefix" placeholder="Prefixo (ex: VIP)" value="VIP">
    <select id="validity">
        <option value="7">7 Dias</option>
        <option value="30" selected>30 Dias</option>
        <option value="365">1 Ano</option>
    </select>
    <button onclick="gerar()">GERAR KEY</button>
    
    <div id="genArea" class="hidden">
        <div class="key-box" id="displayKey"></div>
        <button class="btn-copy" onclick="copiar()">Copiar</button>
        <button style="background:#333" onclick="salvar()">SALVAR NO SERVIDOR</button>
    </div>

    <hr style="border-color: #333; margin: 20px 0;">

    <!-- VALIDADOR (PROXY) -->
    <h2>Painel Proxy</h2>
    <input type="text" id="checkInput" placeholder="Cole sua Key aqui...">
    <button onclick="verificar()">VALIDAR ACESSO</button>
    <div id="proxyResult"></div>

    <hr style="border-color: #333; margin: 20px 0;">

    <!-- ADMIN -->
    <h2>Admin</h2>
    <input type="password" id="adminPass" placeholder="Senha Admin" onkeyup="if(event.key==='Enter') adminLogin()">
    <button onclick="adminLogin()" style="background:#333">Entrar</button>
    <div id="adminList" class="hidden"></div>
</div>

<!-- FIREBASE CONFIG -->
<script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/9.22.0/firebase-app.js";
    import { getDatabase, ref, set, push, onValue, remove, get } from "https://www.gstatic.com/firebasejs/9.22.0/firebase-database.js";

    // ⚠️ COLE AQUI SUA CONFIGURAÇÃO DO FIREBASE (PAINEL WEB) ⚠️
    const firebaseConfig = {
        apiKey: "COLE_AQUI_API_KEY",
        authDomain: "SEU_PROJETO.firebaseapp.com",
        databaseURL: "https://SEU_PROJETO-default-rtdb.firebaseio.com",
        projectId: "SEU_PROJETO",
        storageBucket: "SEU_PROJETO.appspot.com",
        messagingSenderId: "NUMERO",
        appId: "ID_DO_APP"
    };

    // Inicializa
    const app = initializeApp(firebaseConfig);
    const db = getDatabase(app);

    // --- LÓGICA DO SISTEMA ---

    window.gerar = () => {
        const p = document.getElementById('prefix').value || 'KEY';
        const r = Math.random().toString(36).substring(2, 8).toUpperCase();
        const key = `${p}-${r}`;
        document.getElementById('displayKey').innerText = key;
        document.getElementById('genArea').classList.remove('hidden');
    };

    window.copiar = () => {
        const k = document.getElementById('displayKey').innerText;
        navigator.clipboard.writeText(k);
        alert("Key copiada!");
    };

    window.salvar = async () => {
        const key = document.getElementById('displayKey').innerText;
        const dias = parseInt(document.getElementById('validity').value);
        const dataExpiracao = Date.now() + (dias * 24 * 60 * 60 * 1000);
        
        await set(ref(db, 'keys/' + key), {
            status: 'active',
            expiry: dataExpiracao,
            created: Date.now()
        });
        alert("Key salva com sucesso! Agora vá no painel abaixo para testar.");
    };

    window.verificar = async () => {
        const input = document.getElementById('checkInput').value.trim();
        const res = document.getElementById('proxyResult');
        res.innerText = "Verificando...";

        try {
            const snap = await get(ref(db, 'keys/' + input));
            if (snap.exists()) {
                const d = snap.val();
                if (d.status === 'active' && Date.now() < d.expiry) {
                    res.innerHTML = "<span class='status-valid'>✅ ACEITO! Acesso Liberado.</span>";
                } else {
                    res.innerHTML = "<span class='status-invalid'>❌ CHAVE EXPIRADA OU REVOGADA.</span>";
                }
            } else {
                res.innerHTML = "<span class='status-invalid'>❌ CHAVE NÃO ENCONTRADA.</span>";
            }
        } catch (e) {
            res.innerText = "Erro de conexão.";
        }
    };

    // --- ADMIN ---
    window.adminLogin = () => {
        if(document.getElementById('adminPass').value === '1234') {
            document.getElementById('adminList').classList.remove('hidden');
            carregarLista();
        } else {
            alert("Senha errada");
        }
    };

    function carregarLista() {
        const listDiv = document.getElementById('adminList');
        onValue(ref(db, 'keys'), (snapshot) => {
            listDiv.innerHTML = "";
            snapshot.forEach((child) => {
                const k = child.key;
                const d = child.val();
                const exp = new Date(d.expiry).toLocaleDateString();
                const isRevoked = d.status !== 'active';
                
                const div = document.createElement('div');
                div.className = 'list-item';
                div.innerHTML = `
                    <span>${k} <small>(Exp: ${exp})</small></span>
                    <button class="btn-revoke" onclick="revogar('${k}')">REVOGAR</button>
                `;
                listDiv.appendChild(div);
            });
        });
    }

    window.revogar = (key) => {
        if(confirm("Tem certeza?")) {
            set(ref(db, 'keys/' + key), { status: 'revoked' });
        }
    };
</script>
</body>
</html>
