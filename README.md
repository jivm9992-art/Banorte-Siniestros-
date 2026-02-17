[Codigo Mascarilla.txt](https://github.com/user-attachments/files/25374375/Codigo.Mascarilla.txt)
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simulador Siniestros | Banorte & Pentafón</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>

    <style>
        :root {
            --primary-color: #EB0029;
            --bg-color: #f4f6f9;
            --border-color: #d1d3e2;
            --text-color: #5a5c69;
        }
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg-color);
            margin: 0;
            padding: 20px;
            color: #333;
            font-size: 13px;
        }
        #timer-container {
            position: fixed;
            top: 10px;
            right: 20px;
            background: #333;
            color: #0f0;
            padding: 5px 15px;
            border-radius: 20px;
            font-family: 'Courier New', monospace;
            font-weight: bold;
            font-size: 1.2rem;
            box-shadow: 0 4px 6px rgba(0,0,0,0.2);
            z-index: 1000;
        }
        .container {
            background-color: white;
            max-width: 1000px;
            margin: 0 auto;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 0 15px rgba(0,0,0,0.1);
            border-top: 5px solid var(--primary-color);
        }

        /* --- ESTILOS DE LOGOS --- */
        .header-logos {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 20px;
            padding-bottom: 10px;
            border-bottom: 1px solid #eee;
        }
        .header-logos img {
            max-height: 80px; 
            width: auto;
        }

        h2 { margin-top: 5px; color: var(--primary-color); text-align: center; text-transform: uppercase; }
        
        /* Panel Excel */
        .db-panel {
            background-color: #e8f5e9;
            border: 1px solid #c8e6c9;
            padding: 15px;
            margin-bottom: 20px;
            border-radius: 4px;
        }
        .db-panel summary { font-weight: bold; cursor: pointer; color: #2e7d32; }
        input[type="file"] { margin-top: 10px; font-size: 0.9rem; }

        /* Formularios */
        .section-header {
            background-color: #eaecf4;
            padding: 8px 15px;
            font-weight: 700;
            color: #4e73df;
            border-left: 5px solid #4e73df;
            margin: 15px 0 10px 0;
            display: flex; justify-content: space-between; align-items: center;
        }
        .form-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
            gap: 12px; padding: 5px;
        }
        .form-group { display: flex; flex-direction: column; }
        label { font-size: 0.75rem; font-weight: 700; margin-bottom: 4px; color: var(--text-color); }
        input, select, textarea {
            padding: 6px 10px; border: 1px solid var(--border-color);
            border-radius: 4px; font-size: 0.85rem; text-transform: uppercase;
        }
        input:focus, select:focus, textarea:focus {
            outline: none; border-color: var(--primary-color);
            box-shadow: 0 0 0 3px rgba(235, 0, 41, 0.1);
        }
        .readonly { background-color: #eaecf4; pointer-events: none; color: #6e707e; font-weight: bold; }

        /* Botones */
        .btn-search {
            background-color: #4e73df; color: white; border: none;
            padding: 8px 15px; border-radius: 4px; cursor: pointer;
            font-weight: bold; height: 32px; margin-top: 18px; 
        }
        .btn-search:hover { background-color: #2e59d9; }
        
        .btn-generate {
            background-color: var(--primary-color); color: white; border: none;
            padding: 15px; width: 100%; font-size: 1.1rem; font-weight: bold;
            border-radius: 5px; cursor: pointer; margin-top: 20px;
        }
        .btn-generate:hover { background-color: #c90022; }

        .btn-download-db {
            background-color: #28a745; color: white; border: none;
            padding: 10px; width: 100%; font-size: 1rem; font-weight: bold;
            border-radius: 5px; cursor: pointer; margin-top: 10px;
            display: none;
        }
        .btn-download-db:hover { background-color: #218838; }

        /* BOTON LIMPIAR (NUEVO) */
        .btn-clear {
            background-color: #6c757d; color: white; border: none;
            padding: 10px; width: 100%; font-size: 1rem; font-weight: bold;
            border-radius: 5px; cursor: pointer; margin-top: 10px;
        }
        .btn-clear:hover { background-color: #5a6268; }

        /* Mensajes */
        #sys-msg {
            display: none; padding: 15px; margin-top: 15px;
            border-radius: 5px; text-align: center; font-weight: bold;
        }
        .msg-error { background-color: #f8d7da; color: #721c24; border: 1px solid #f5c6cb; }
        .msg-success { background-color: #d4edda; color: #155724; border: 1px solid #c3e6cb; }
        .msg-warning { background-color: #fff3cd; color: #856404; border: 1px solid #ffeeba; }
    </style>
</head>
<body>

<div id="timer-container">⏱ 00:00</div>

<div class="container">
    
    <div class="header-logos">
        <img src="Logo Banorte.jpg" alt="Logo Banorte">
        <img src="Logo Pentafón.jpg" alt="Logo Pentafón">
    </div>

    <details class="db-panel">
        <summary>📂 Cargar Base de Datos (Excel)</summary>
        <div style="margin-top: 10px;">
            <p style="margin: 0 0 5px 0;">Sube tu archivo <strong>.xlsx</strong>. El sistema buscará columnas como "Póliza", "Asegurado", etc.</p>
            <input type="file" id="excelInput" accept=".xlsx, .xls" onchange="cargarExcel(this)">
            <br><br>
            <span id="db-status" style="font-weight:bold; color: #555;">Estado: Usando Base de Datos de Prueba</span>
        </div>
    </details>

    <h2>Gestión de Reporte de Siniestro</h2>

    <div class="section-header">1. BÚSQUEDA Y CONTACTO INICIAL</div>
    <div class="form-grid">
        <div class="form-group">
            <label>Póliza</label>
            <input type="text" id="b_poliza" maxlength="7" placeholder="7 Dígitos" oninput="validarNum(this)">
        </div>
        <div class="form-group">
            <label>Oficina</label>
            <input type="text" id="b_oficina" maxlength="3" placeholder="3 Caracteres" style="text-transform: uppercase;">
        </div>
        <div class="form-group">
            <label>Inciso</label>
            <input type="text" id="b_inciso" maxlength="3" value="1" oninput="validarNum(this)">
        </div>
        <div class="form-group">
            <label>Serie (VIN)</label>
            <input type="text" id="b_serie" maxlength="17" placeholder="17 Caracteres">
        </div>
        <div class="form-group">
            <button class="btn-search" onclick="buscarPoliza()">🔍 Buscar Póliza</button>
        </div>
    </div>
    
    <div class="form-grid" style="border-top: 1px dashed #ccc; padding-top: 10px;">
        <div class="form-group" style="grid-column: span 2;">
            <label>Nombre Reportante</label>
            <input type="text" id="rep_nombre" placeholder="Nombre completo">
        </div>
        <div class="form-group" style="grid-column: span 2;">
            <label>Teléfono Contacto</label>
            <input type="text" id="rep_telefono" maxlength="10" placeholder="10 Dígitos" oninput="validarNum(this)">
        </div>
    </div>

    <div class="section-header">2. DATOS DE PÓLIZA (SISTEMA)</div>
    <div class="form-grid">
        <div class="form-group" style="grid-column: span 2;">
            <label>Asegurado</label>
            <input type="text" id="res_asegurado" class="readonly">
        </div>
        <div class="form-group">
            <label>Estatus</label>
            <input type="text" id="res_estatus" class="readonly">
        </div>
        <div class="form-group">
            <label>Vigencia</label>
            <input type="text" id="res_vigencia" class="readonly">
        </div>
        <div class="form-group">
            <label>Marca</label>
            <input type="text" id="res_marca" class="readonly">
        </div>
        <div class="form-group">
            <label>Submarca</label>
            <input type="text" id="res_submarca" class="readonly">
        </div>
        <div class="form-group">
            <label>Modelo</label>
            <input type="text" id="res_modelo" class="readonly">
        </div>
    </div>

    <div class="section-header">3. DETALLE DEL EVENTO</div>
    
    <div class="form-grid">
        <div class="form-group">
            <label>1. Fecha Siniestro</label>
            <input type="date" id="sin_fecha">
        </div>
        <div class="form-group">
            <label>Hora Aprox.</label>
            <input type="time" id="sin_hora">
        </div>
    </div>

    <div class="form-grid">
        <div class="form-group">
            <label>2. Estado</label>
            <select id="sin_estado" onchange="cargarMunicipios()">
                <option value="">SELECCIONAR...</option>
            </select>
        </div>
        <div class="form-group">
            <label>3. Municipio</label>
            <select id="sin_municipio">
                <option value="">SELECCIONE ESTADO</option>
            </select>
        </div>
    </div>

    <div class="form-grid">
        <div class="form-group" style="grid-column: span 4;">
            <label>4. Descripción de Vehículos</label>
            <input type="text" id="sin_desc_veh">
        </div>
    </div>

    <div class="form-grid">
        <div class="form-group" style="grid-column: span 4;">
            <label>5. Dirección del Siniestro</label>
            <input type="text" id="sin_direccion">
        </div>
    </div>

    <div class="form-grid">
        <div class="form-group" style="grid-column: span 4;">
            <label>6. Referencias Visuales</label>
            <input type="text" id="sin_referencias">
        </div>
    </div>

    <div class="form-grid">
        <div class="form-group" style="grid-column: span 4;">
            <label>7. Comentarios Adicionales</label>
            <textarea id="sin_comentarios" rows="2"></textarea>
        </div>
    </div>

    <div class="form-grid">
        <div class="form-group" style="grid-column: span 2;">
            <label>8. Causa</label>
            <select id="sin_causa">
                <option value="">SELECCIONAR...</option>
                <option value="COLISION">COLISIÓN</option>
                <option value="ATROPELLO">ATROPELLO</option>
                <option value="ROBO POR ASALTO">ROBO POR ASALTO</option>
                <option value="ROBO ESTACIONADO">ROBO ESTACIONADO</option>
                <option value="ROBO PARCIAL">ROBO PARCIAL</option>
                <option value="ABUSO DE CONFIANZA">ABUSO DE CONFIANZA</option>
                <option value="VOLCADURA">VOLCADURA</option>
                <option value="INCENDIO">INCENDIO</option>
                <option value="INUNDACION">INUNDACIÓN</option>
            </select>
        </div>
    </div>

    <div class="section-header">4. DATOS DEL CONDUCTOR</div>
    <div class="form-grid">
        <div class="form-group" style="grid-column: span 2;">
            <label>Nombre Conductor</label>
            <input type="text" id="cond_nombre">
        </div>
        <div class="form-group" style="grid-column: span 2;">
            <label>Teléfono Conductor</label>
            <input type="text" id="cond_telefono" maxlength="10" oninput="validarNum(this)">
        </div>
    </div>

    <div class="section-header">5. VEHÍCULO ASEGURADO (CONFIRMACIÓN)</div>
    <div class="form-grid">
        <div class="form-group">
            <label>Placas</label>
            <input type="text" id="veh_placas" maxlength="8">
        </div>
        <div class="form-group">
            <label>Color</label>
            <select id="veh_color">
                <option value="">SELECCIONAR...</option>
                <option value="BLANCO">BLANCO</option>
                <option value="NEGRO">NEGRO</option>
                <option value="PLATA">PLATA</option>
                <option value="GRIS">GRIS</option>
                <option value="ROJO">ROJO</option>
                <option value="AZUL">AZUL</option>
                <option value="ARENA">ARENA</option>
                <option value="VINO">VINO</option>
                <option value="OTRO">OTRO</option>
            </select>
        </div>
    </div>

    <button class="btn-generate" onclick="generarReporte()">✅ GENERAR REPORTE</button>
    <button class="btn-clear" onclick="limpiarFormulario()">🧹 LIMPIAR / NUEVO REPORTE</button>
    <button class="btn-download-db" id="btnDescarga" onclick="descargarBaseActualizada()">💾 DESCARGAR EXCEL CON REPORTES</button>
    
    <div id="sys-msg"></div>

</div>

<script>
    // --- VARIABLES GLOBALES ---
    let reportesGenerados = []; 
    let workbookOriginal = null; 

    // --- 1. CRONÓMETRO ---
    let segundos = 0; let timerActivo = false; let intervalo;
    document.body.addEventListener('input', iniciarTimer);
    document.body.addEventListener('click', iniciarTimer);
    function iniciarTimer() {
        if (!timerActivo) {
            timerActivo = true;
            intervalo = setInterval(() => {
                segundos++;
                const min = Math.floor(segundos / 60).toString().padStart(2, '0');
                const sec = (segundos % 60).toString().padStart(2, '0');
                document.getElementById('timer-container').innerText = `⏱ ${min}:${sec}`;
            }, 1000);
        }
    }

    // --- 2. BASE DE DATOS FLEXIBLE ---
    let db = [
        { poliza: "1234567", oficina: "100", inciso: "1", serie: "VINTEST1234567890", asegurado: "JUAN PEREZ (PRUEBA)", estatus: "VIGENTE", vigencia: "31/12/2026", marca: "NISSAN", submarca: "VERSA", modelo: "2024" }
    ];

    function cargarExcel(input) {
        const file = input.files[0];
        if (!file) return;
        const reader = new FileReader();
        reader.onload = function(e) {
            try {
                const data = new Uint8Array(e.target.result);
                workbookOriginal = XLSX.read(data, {type: 'array'});
                const sheet = workbookOriginal.Sheets[workbookOriginal.SheetNames[0]];
                const jsonData = XLSX.utils.sheet_to_json(sheet);

                if (jsonData.length === 0) { alert("Archivo vacío."); return; }

                // Mapeo flexible
                db = jsonData.map(row => {
                    const getVal = (posiblesNombres) => {
                        const keys = Object.keys(row);
                        const normalize = s => s.toLowerCase().normalize("NFD").replace(/[\u0300-\u036f]/g, "").trim();
                        for (let nombreBuscado of posiblesNombres) {
                            const foundKey = keys.find(k => normalize(k) === nombreBuscado);
                            if (foundKey) return row[foundKey];
                        }
                        return "";
                    };

                    return {
                        poliza: String(getVal(["poliza", "num poliza", "policy", "póliza"])).trim(),
                        oficina: String(getVal(["oficina", "sucursal", "ofi"])).trim(),
                        inciso: String(getVal(["inciso", "inc"])).trim(),
                        serie: String(getVal(["serie", "vin", "serial"])).trim(),
                        asegurado: getVal(["asegurado", "nombre", "cliente", "contratante", "titular"]),
                        estatus: getVal(["estatus", "estado", "status", "situacion"]),
                        vigencia: getVal(["vigencia", "fin vigencia", "vencimiento", "termino"]),
                        marca: getVal(["marca", "vehiculo"]),
                        submarca: getVal(["submarca", "linea", "tipo"]),
                        modelo: String(getVal(["modelo", "anio", "año"])).trim()
                    };
                });

                document.getElementById('db-status').innerHTML = `✅ BD Cargada: ${db.length} registros.`;
                document.getElementById('db-status').style.color = "green";
                alert(`Se cargaron ${db.length} registros. Ahora puedes generar reportes.`);
            } catch (error) {
                console.error(error);
                alert("Error al leer Excel. Verifica el formato.");
            }
        };
        reader.readAsArrayBuffer(file);
    }

    // --- 3. GEOGRAFÍA ---
    const geoData = {
        "AGUASCALIENTES": ["AGUASCALIENTES", "JESUS MARIA"], "BAJA CALIFORNIA": ["TIJUANA", "MEXICALI", "ENSENADA"],
        "BAJA CALIFORNIA SUR": ["LA PAZ", "LOS CABOS"], "CAMPECHE": ["CAMPECHE", "CARMEN"],
        "CHIAPAS": ["TUXTLA GUTIERREZ", "TAPACHULA"], "CHIHUAHUA": ["CHIHUAHUA", "JUAREZ"],
        "CIUDAD DE MEXICO": ["AZCAPOTZALCO", "COYOACAN", "IZTAPALAPA", "BENITO JUAREZ", "CUAUHTEMOC", "MIGUEL HIDALGO"],
        "COAHUILA": ["SALTILLO", "TORREON"], "COLIMA": ["COLIMA", "MANZANILLO"],
        "DURANGO": ["DURANGO", "GOMEZ PALACIO"], "GUANAJUATO": ["LEON", "IRAPUATO", "CELAYA"],
        "GUERRERO": ["ACAPULCO", "CHILPANCINGO"], "HIDALGO": ["PACHUCA", "TULANCINGO"],
        "JALISCO": ["GUADALAJARA", "ZAPOPAN", "TLAQUEPAQUE", "PUERTO VALLARTA"],
        "MEXICO (EDOMEX)": ["ECATEPEC", "NAUCALPAN", "TOLUCA", "NEZAHUALCOYOTL"],
        "MICHOACAN": ["MORELIA", "URUAPAN"], "MORELOS": ["CUERNAVACA", "CUAUTLA"],
        "NAYARIT": ["TEPIC", "BAHIA DE BANDERAS"], "NUEVO LEON": ["MONTERREY", "SAN PEDRO", "APODACA"],
        "OAXACA": ["OAXACA", "TUXTEPEC"], "PUEBLA": ["PUEBLA", "CHOLULA"],
        "QUERETARO": ["QUERETARO", "SAN JUAN DEL RIO"], "QUINTANA ROO": ["CANCUN", "PLAYA DEL CARMEN"],
        "SAN LUIS POTOSI": ["SAN LUIS POTOSI", "VALLES"], "SINALOA": ["CULIACAN", "MAZATLAN"],
        "SONORA": ["HERMOSILLO", "OBREGON"], "TABASCO": ["VILLAHERMOSA", "CARDENAS"],
        "TAMAULIPAS": ["REYNOSA", "TAMPICO"], "TLAXCALA": ["TLAXCALA", "APIZACO"],
        "VERACRUZ": ["VERACRUZ", "XALAPA"], "YUCATAN": ["MERIDA", "VALLADOLID"], "ZACATECAS": ["ZACATECAS", "FRESNILLO"]
    };
    const selEdo = document.getElementById('sin_estado');
    const selMun = document.getElementById('sin_municipio');
    Object.keys(geoData).sort().forEach(e => { let op = document.createElement('option'); op.value = e; op.text = e; selEdo.appendChild(op); });
    function cargarMunicipios() {
        const est = selEdo.value; selMun.innerHTML = '<option value="">SELECCIONAR...</option>';
        if(est && geoData[est]) geoData[est].forEach(m => { let op = document.createElement('option'); op.value = m; op.text = m; selMun.appendChild(op); });
    }

    // --- 4. LÓGICA ---
    function validarNum(i) { i.value = i.value.replace(/[^0-9]/g, ''); }

    function buscarPoliza() {
        const p = document.getElementById('b_poliza').value.trim();
        const o = document.getElementById('b_oficina').value.trim().toUpperCase();
        const i = document.getElementById('b_inciso').value.trim();
        const s = document.getElementById('b_serie').value.trim().toUpperCase();

        document.querySelectorAll('.readonly').forEach(el => el.value = "");
        document.getElementById('sys-msg').style.display = 'none';

        let found = null;
        if(s.length > 4) {
            found = db.find(x => x.serie.includes(s));
        } else if (p && o && i) {
            // Comparación estricta con uppercase
            found = db.find(x => x.poliza == p && x.oficina.toUpperCase() == o && x.inciso == i);
        } else {
            alert("Error: Ingresa Póliza+Oficina+Inciso O una Serie válida."); return;
        }

        if(found) {
            document.getElementById('res_asegurado').value = found.asegurado || "SIN DATO EN ARCHIVO";
            document.getElementById('res_estatus').value = found.estatus || "DESCONOCIDO";
            document.getElementById('res_vigencia').value = found.vigencia || "PENDIENTE";
            document.getElementById('res_marca').value = found.marca || "SIN MARCA";
            document.getElementById('res_submarca').value = found.submarca || "SIN SUBMARCA";
            document.getElementById('res_modelo').value = found.modelo || "----";

            if(s.length > 0) {
                document.getElementById('b_poliza').value = found.poliza;
                document.getElementById('b_oficina').value = found.oficina;
                document.getElementById('b_inciso').value = found.inciso;
            }
            alert("✅ Póliza Encontrada");
        } else {
            alert("❌ No encontrada. Verifica los datos.");
        }
    }

    function generarReporte() {
        // --- RECOLECCIÓN Y VALIDACIÓN SUAVE ---
        let faltantes = [];
        
        const nombreRep = document.getElementById('rep_nombre').value;
        const nombreCond = document.getElementById('cond_nombre').value;
        const telContacto = document.getElementById('rep_telefono').value;
        
        // Datos Poliza
        const poliza = document.getElementById('b_poliza').value;
        const oficina = document.getElementById('b_oficina').value;
        const inciso = document.getElementById('b_inciso').value;
        const valAsegurado = document.getElementById('res_asegurado').value;

        // Datos Siniestro
        const fecha = document.getElementById('sin_fecha').value;
        const hora = document.getElementById('sin_hora').value;
        const estado = document.getElementById('sin_estado').value;
        const municipio = document.getElementById('sin_municipio').value;
        const direccion = document.getElementById('sin_direccion').value;
        const descVeh = document.getElementById('sin_desc_veh').value;
        const causa = document.getElementById('sin_causa').value;
        
        // Datos Vehiculo
        const placas = document.getElementById('veh_placas').value;
        const color = document.getElementById('veh_color').value;

        // Check List de Faltantes
        if(!nombreRep) faltantes.push("Nombre Reportante");
        if(telContacto.length < 10) faltantes.push("Teléfono Contacto");
        if(!valAsegurado || valAsegurado === "") faltantes.push("Datos Póliza");
        if(!fecha) faltantes.push("Fecha");
        if(!estado) faltantes.push("Estado");
        if(!municipio) faltantes.push("Municipio");
        if(!descVeh) faltantes.push("Desc. Vehículos");
        if(!direccion) faltantes.push("Dirección");
        if(!causa) faltantes.push("Causa");
        if(!nombreCond) faltantes.push("Conductor");
        if(!placas) faltantes.push("Placas");
        if(!color) faltantes.push("Color");

        clearInterval(intervalo);
        const folio = "SIN-" + Math.floor(Math.random() * 900000 + 100000);
        const tiempo = document.getElementById('timer-container').innerText;
        
        const estatusFinal = faltantes.length === 0 ? "COMPLETO" : "INCOMPLETO";
        const stringFaltantes = faltantes.length === 0 ? "NINGUNO" : faltantes.join(", ");

        // Guardar reporte en memoria
        const nuevoReporte = {
            "Folio": folio,
            "Estatus Reporte": estatusFinal,
            "Datos Faltantes": stringFaltantes,
            "Nombre Reportante": nombreRep,
            "Nombre Conductor": nombreCond,
            "Numero Contacto": telContacto,
            "Poliza": poliza,
            "Oficina": oficina,
            "Inciso": inciso,
            "Estado": estado,
            "Municipio": municipio,
            "Direccion": direccion,
            "Fecha": fecha,
            "Hora": hora,
            "Tiempo Atencion": tiempo
        };
        reportesGenerados.push(nuevoReporte);

        document.getElementById('btnDescarga').style.display = "block";
        const msgDiv = document.getElementById('sys-msg');

        if(faltantes.length > 0) {
            msgDiv.className = "msg-warning";
            msgDiv.innerHTML = `
                ⚠️ <strong>REPORTE GENERADO CON OBSERVACIONES</strong><br>
                Folio: ${folio}<br>
                El reporte se guardó pero faltaron los siguientes datos: <br>
                <em style="font-size:0.9em">${stringFaltantes}</em>
            `;
        } else {
            msgDiv.className = "msg-success";
            msgDiv.innerHTML = `
                ✅ <strong>REPORTE COMPLETADO CON ÉXITO</strong><br>
                Folio: ${folio}<br>
                Tiempo: ${tiempo}<br>
                Todos los campos capturados.
            `;
        }
        msgDiv.style.display = "block";
        msgDiv.scrollIntoView({behavior:"smooth"});
    }

    function limpiarFormulario() {
        if(!confirm("¿Deseas limpiar el formulario para iniciar un nuevo reporte? (Los reportes anteriores siguen guardados)")) return;

        // Limpiar inputs (excepto file)
        document.querySelectorAll("input:not([type='file']), select, textarea").forEach(el => el.value = "");
        
        // Restaurar defaults
        document.getElementById('b_inciso').value = "1";

        // Reset Timer
        clearInterval(intervalo);
        timerActivo = false;
        segundos = 0;
        document.getElementById('timer-container').innerText = "⏱ 00:00";

        // Ocultar mensajes
        document.getElementById('sys-msg').style.display = 'none';

        window.scrollTo({ top: 0, behavior: 'smooth' });
    }

    function descargarBaseActualizada() {
        if (!workbookOriginal) {
            workbookOriginal = XLSX.utils.book_new();
        }
        const hojaReportes = XLSX.utils.json_to_sheet(reportesGenerados);
        const nombreHoja = "HISTORIAL_REPORTES";

        if(workbookOriginal.Sheets[nombreHoja]) {
            workbookOriginal.Sheets[nombreHoja] = hojaReportes;
        } else {
            XLSX.utils.book_append_sheet(workbookOriginal, hojaReportes, nombreHoja);
        }
        XLSX.writeFile(workbookOriginal, "BaseDatos_Actualizada_Reportes.xlsx");
    }
</script>

</body>
</html>
