<!-- GERADOR DE KEYS -->
<div style="background:#000; padding:20px; border:1px solid #333; font-family:sans-serif;">
    <h3 style="color:red;">Gerador de Key</h3>
    
    <!-- Configuração -->
    <input type="text" id="genPrefix" placeholder="Prefixo (ex: VIP)" value="VIP" style="width:95%; padding:10px; background:#222; color:white; border:1px solid #444; margin-bottom:10px;">
    
    <select id="genDays" style="width:95%; padding:10px; background:#222; color:white; border:1px solid #444; margin-bottom:10px;">
        <option value="7">7 Dias</option>
        <option value="30" selected>30 Dias</option>
        <option value="365">1 Ano</option>
    </select>

    <button onclick="gerarKey()" style="width:100%; padding:15px; background:red; color:white; font-weight:bold; cursor:pointer;">GERAR KEY</button>

    <!-- Resultado -->
    <div id="resultArea" class="hidden" style="margin-top:15px; text-align:center;">
        <div id="keyDisplay" style="background:#111; color:red; font-family:monospace; padding:15px; font-size:18px; word-break:break-all; border:1px dashed #444;"></div>
        <button onclick="copiarKey()" style="margin-top:10px; width:100%; background:#333; color:white; padding:10px; cursor:pointer;">COPIAR E SALVAR</button>
    </div>
</div>

<script>
// CONFIGURAÇÃO DO FIREBASE (Cole aqui suas chaves do Firebase)
const firebaseConfig = {
    apiKey: "SUA_API_KEY",
    authDomain: "SEU_PROJETO.firebaseapp.com",
    databaseURL: "https://SEU_PROJETO-default-rtdb.firebaseio.com",
    projectId: "SEU_PROJETO",
    storageBucket: "SEU_PROJETO.appspot.com",
    messagingSenderId: "NUMERO",
    appId: "ID_APP"
};

import { initializeApp } from "https://www.gstatic.com/firebasejs/9.22.0/firebase-app.js";
import { getDatabase, ref, set } from "https://www.gstatic.com/firebasejs/9.22.0/firebase-database.js";

const app = initializeApp(firebaseConfig);
const db = getDatabase(app);

function gerarKey() {
    const prefix = document.getElementById('genPrefix').value || 'KEY';
    const random = Math.random().toString(36).substring(2, 8).toUpperCase();
    const keyFinal = `${prefix}-${random}`;
    
    // Mostra na tela
    document.getElementById('keyDisplay').innerText = keyFinal;
    document.getElementById('resultArea').classList.remove('hidden');
    
    // Prepara dados
    const dias = parseInt(document.getElementById('genDays').value);
    const expiry = Date.now() + (dias * 24 * 60 * 60 * 1000);
    
    // Salva no Servidor (Firebase)
    set(ref(db, 'keys/' + keyFinal), {
        status: 'active',
        expiry: expiry,
        created: Date.now()
    }).then(() => {
        alert("Key gerada e salva com sucesso!");
    });
}

function copiarKey() {
    const k = document.getElementById('keyDisplay').innerText;
    navigator.clipboard.writeText(k);
    alert("Copiada! Agora cole no site do Proxy.");
}
</script>
