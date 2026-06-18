<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Horarios Polanco</title>
    
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="theme-color" content="#1b4f72">
    
    <link rel="manifest" href='data:application/manifest+json,{"name":"Control de Horarios","short_name":"Horarios","start_url":".","display":"standalone","background_color":"#ffffff","theme_color":"#1b4f72","orientation":"landscape","icons":[{"src":"https://cdn-icons-png.flaticon.com/512/3652/3652191.png","sizes":"512x512","type":"image/png"}]}'>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    
    <style>
        :root {
            --primary-blue: #1b4f72;
            --secondary-blue: #2e86c1;
            --accent-yellow: #ffff00;
            --border-color: #a6a6a6;
            --whatsapp-green: #25d366;
        }
        * { box-sizing: border-box; font-family: 'Segoe UI', Helvetica, Arial, sans-serif; margin: 0; padding: 0; }
        body { background-color: #f7f9fa; color: #333; padding: 10px; }
        header { background-color: var(--primary-blue); color: white; padding: 12px; border-radius: 8px 8px 0 0; display: flex; flex-direction: column; gap: 10px; align-items: center; }
        @media(min-width: 600px) { header { flex-direction: row; justify-content: space-between; } }
        .nav-tabs { display: flex; gap: 5px; width: 100%; justify-content: space-around; }
        @media(min-width: 600px) { .nav-tabs { width: auto; } }
        .tab-btn { background-color: transparent; color: white; border: 1.5px solid white; padding: 6px 12px; cursor: pointer; border-radius: 4px; font-weight: bold; font-size: 13px; }
        .tab-btn.active { background-color: white; color: var(--primary-blue); }
        .container { background: white; padding: 15px; border: 1px solid var(--border-color); border-top: none; border-radius: 0 0 8px 8px; box-shadow: 0 4px 6px rgba(0,0,0,0.05); min-height: 400px; }
        .page { display: none; }
        .page.active { display: block; }
        .form-group { margin-bottom: 12px; width: 100%; }
        label { display: block; margin-bottom: 4px; font-weight: bold; font-size: 14px; }
        input[type="text"], input[type="date"], select { width: 100%; padding: 8px; border: 1px solid var(--border-color); border-radius: 4px; font-size: 14px; background-color: white; }
        .btn { background-color: var(--secondary-blue); color: white; border: none; padding: 10px 15px; cursor: pointer; border-radius: 4px; font-weight: bold; font-size: 13px; }
        .btn-danger { background-color: #922b21; }
        .btn-success { background-color: #1e8449; }
        .btn-secondary { background-color: #7f8c8d; }
        .btn-whatsapp { background-color: var(--whatsapp-green); color: white; }
        .table-wrapper { overflow-x: auto; margin-top: 15px; background: white; border: 1px solid #000; }
        .schedule-table { width: 100%; border-collapse: collapse; font-size: 11px; text-align: center; }
        .schedule-table th, .schedule-table td { border: 1px solid #000; padding: 4px; min-width: 65px; height: 24px; }
        .th-title { background-color: var(--primary-blue); color: white; font-weight: bold; text-align: left; font-size: 13px; }
        .th-day { background-color: var(--secondary-blue); color: white; font-weight: bold; text-transform: capitalize; }
        .th-sub { background-color: #d9e1f2; font-weight: bold; }
        .special-cell { background-color: var(--accent-yellow) !important; font-weight: bold; }
        .empty-row { background-color: #a6a6a6; }
        .grid-assignment { display: grid; grid-template-columns: repeat(auto-fit, minmax(140px, 1fr)); gap: 10px; margin-top: 15px; }
        .day-box { border: 1px solid var(--border-color); padding: 8px; background: #fcfcfc; border-radius: 4px; }
        .day-box h4 { margin-bottom: 5px; color: var(--primary-blue); border-bottom: 1px solid var(--border-color); font-size: 13px; }
        .employee-list { margin-top: 20px; width: 100%; border-collapse: collapse; font-size: 14px; }
        .employee-list th, .employee-list td { border: 1px solid var(--border-color); padding: 8px; }
        .employee-list th { background-color: #eaeded; }
        .food-table { margin-top: 15px; width: 260px; border-collapse: collapse; font-size: 11px; }
        .food-table th, .food-table td { border: 1px solid #000; padding: 3px; }
        .food-header { background-color: var(--primary-blue); color: white; font-weight: bold; }
    </style>
</head>
<body>

    <header>
        <h3>Horarios de Personal</h3>
        <div class="nav-tabs">
            <button class="tab-btn active" onclick="switchPage('consulta', this)">Consulta</button>
            <button class="tab-btn" onclick="switchPage('asignacion', this)">Asignar</button>
            <button class="tab-btn" onclick="switchPage('captura', this)">Personal</button>
        </div>
    </header>

    <div class="container">
        
        <div id="page-consulta" class="page active">
            <div style="display: flex; flex-direction: column; gap: 10px; margin-bottom: 10px;">
                <div class="form-group" style="margin: 0; display: flex; align-items: center; gap: 8px;">
                    <label style="white-space: nowrap; font-size:13px;">Semana (Lunes):</label>
                    <input type="date" id="search-week-date" onchange="syncDatesFromConsulta()">
                </div>
                <div style="display: flex; flex-wrap: wrap; gap: 8px; width: 100%;">
                    <button class="btn btn-success" style="flex: 1; min-width: 120px;" onclick="exportToImage()">Guardar Imagen</button>
                    <button class="btn btn-whatsapp" style="flex: 1; min-width: 150px;" onclick="shareToWhatsApp()">Compartir en WhatsApp</button>
                    <button class="btn btn-secondary" onclick="clearWeekData()">Limpiar</button>
                </div>
            </div>

            <div id="capture-area" class="table-wrapper">
                <table class="schedule-table">
                    <thead>
                        <tr>
                            <th class="th-title" colspan="2">Tienda: Polanco</th>
                            <th class="th-day" colspan="2" id="head-lunes">lunes --</th>
                            <th class="th-day" colspan="2" id="head-martes">martes --</th>
                            <th class="th-day" colspan="2" id="head-miercoles">miércoles --</th>
                            <th class="th-day" colspan="2" id="head-jueves">jueves --</th>
                            <th class="th-day" colspan="2" id="head-viernes">viernes --</th>
                            <th class="th-day" colspan="2" id="head-sabado">sábado --</th>
                            <th class="th-day" colspan="2" id="head-domingo">domingo --</th>
                        </tr>
                        <tr>
                            <th class="th-sub" style="text-align: left;">Especialista</th>
                            <th class="th-sub">Posicion</th>
                            <th class="th-sub">Entrada</th><th class="th-sub">Salida</th>
                            <th class="th-sub">Entrada</th><th class="th-sub">Salida</th>
                            <th class="th-sub">Entrada</th><th class="th-sub">Salida</th>
                            <th class="th-sub">Entrada</th><th class="th-sub">Salida</th>
                            <th class="th-sub">Entrada</th><th class="th-sub">Salida</th>
                            <th class="th-sub">Entrada</th><th class="th-sub">Salida</th>
                            <th class="th-sub">Entrada</th><th class="th-sub">Salida</th>
                        </tr>
                    </thead>
                    <tbody id="schedule-rows"></tbody>
                </table>

                <table class="food-table">
                    <tr>
                        <th rowspan="3" style="border: none; font-weight: bold; font-size: 12px; text-align: left; vertical-align: middle; width: 70px;">Facialista</th>
                        <th class="food-header" colspan="2">Horarios de Comida</th>
                    </tr>
                    <tr><td>Apertura</td><td>2:00-3:00</td></tr>
                    <tr><td>Apertura</td><td>2:00-3:00</td></tr>
                    <tr><th style="border: none; font-weight: bold; font-size: 12px; text-align: left;">Vendedor</th><td>Cierre</td><td>3:00-4:00</td></tr>
                </table>
            </div>
        </div>

        <div id="page-asignacion" class="page">
            <div class="form-group">
                <label>1. Semana (Lunes):</label>
                <input type="date" id="assign-week-date" onchange="syncDatesFromAssign()">
            </div>
            <div class="form-group">
                <label>2. Personal:</label>
                <select id="assign-employee" onchange="loadEmployeeSchedule()"></select>
            </div>

            <div class="grid-assignment">
                <script>
                    const dias = ['Lunes', 'Martes', 'Miércoles', 'Jueves', 'Viernes', 'Sábado', 'Domingo'];
                    dias.forEach((dia, idx) => {
                        document.write(`
                            <div class="day-box">
                                <h4>${dia}</h4>
                                <div style="margin-bottom: 4px;">
                                    <select id="day-entry-${idx}" onchange="calculateSingleExit(${idx})" style="font-size:12px; padding:4px;">
                                        <option value="">-- Vacío --</option>
                                        <option value="09:00 a.m.">09:00 a.m.</option>
                                        <option value="10:00 a.m.">10:00 a.m.</option>
                                        <option value="11:00 a.m.">11:00 a.m.</option>
                                        <option value="12:00 p.m.">12:00 p.m.</option>
                                        <option value="01:00 p.m.">01:00 p.m.</option>
                                        <option value="Descanso">"Descanso"</option>
                                        <option value="Vacaciones">"Vacaciones"</option>
                                        <option value="Incapacidad">"Incapacidad"</option>
                                        <option value="Capacitación">"Capacitación"</option>
                                        <option value="Corporativo">"Corporativo"</option>
                                        <option value="Otra Tienda">"Otra Tienda"</option>
                                    </select>
                                </div>
                                <input type="text" id="day-exit-${idx}" readonly style="background: #eaeded; font-weight: bold; font-size:12px; padding:4px;" placeholder="Salida">
                            </div>
                        `);
                    });
                </script>
            </div>

            <div style="margin-top: 15px; display: flex; gap: 10px;">
                <button class="btn btn-success" style="flex: 1;" onclick="saveSchedule()">Guardar Cambios</button>
                <button class="btn btn-danger" onclick="loadEmployeeSchedule()">Descartar</button>
            </div>
        </div>

        <div id="page-captura" class="page">
            <h3>Nuevo Empleado</h3>
            <div class="form-group" style="margin-top:10px;">
                <label>Nombre Completo:</label>
                <input type="text" id="emp-name" placeholder="Ej. Nancy Ramos">
            </div>
            <div class="form-group">
                <label>Puesto:</label>
                <select id="emp-position">
                    <option value="Gerente">Gerente (8 hrs)</option>
                    <option value="Vendedora">Vendedora (8 hrs)</option>
                    <option value="Cosmetóloga">Cosmetóloga (9 hrs)</option>
                </select>
            </div>
            <button class="btn btn-success" style="width:100%;" onclick="addEmployee()">Registrar</button>

            <table class="employee-list">
                <thead><tr><th>Nombre</th><th>Puesto</th><th>Acción</th></tr></thead>
                <tbody id="employee-table-rows"></tbody>
            </table>
        </div>

    </div>

    <script>
        let empleados = [];
        let horarios = {};

        function cargarDeLocalStorage() {
            try {
                const empLocal = localStorage.getItem('h_empleados');
                const horLocal = localStorage.getItem('h_horarios');
                empleados = empLocal ? JSON.parse(empLocal) : [];
                horarios = horLocal ? JSON.parse(horLocal) : {};
            } catch (e) {
                console.error("Error cargando localStorage", e);
                empleados = [];
                horarios = {};
            }
        }

        function guardarEnLocalStorage() {
            localStorage.setItem('h_empleados', JSON.stringify(empleados));
            localStorage.setItem('h_horarios', JSON.stringify(horarios));
        }

        window.onload = function() {
            cargarDeLocalStorage();
            
            const hoy = new Date();
            const deToMonday = hoy.getDay() === 0 ? 6 : hoy.getDay() - 1;
            const lunes = new Date(hoy.setDate(hoy.getDate() - deToMonday));
            
            const año = lunes.getFullYear();
            const mes = String(lunes.getMonth() + 1).padStart(2, '0');
            const dia = String(lunes.getDate()).padStart(2, '0');
            const fechaStr = `${año}-${mes}-${dia}`;
            
            document.getElementById('search-week-date').value = fechaStr;
            document.getElementById('assign-week-date').value = fechaStr;

            renderEmployees();
            populateEmployeeSelects();
            renderScheduleTable();
        };

        function switchPage(pageId, btn) {
            document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
            document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
            
            document.getElementById(`page-${pageId}`).classList.add('active');
            btn.classList.add('active');

            cargarDeLocalStorage();

            if(pageId === 'consulta') {
                renderScheduleTable();
            }
            if(pageId === 'asignacion') {
                populateEmployeeSelects();
                loadEmployeeSchedule();
            }
        }

        function syncDatesFromConsulta() {
            document.getElementById('assign-week-date').value = document.getElementById('search-week-date').value;
            renderScheduleTable();
        }

        function syncDatesFromAssign() {
            document.getElementById('search-week-date').value = document.getElementById('assign-week-date').value;
            loadEmployeeSchedule();
        }

        function addEmployee() {
            const name = document.getElementById('emp-name').value.trim();
            const position = document.getElementById('emp-position').value;
            if(!name) return alert('Introduce un nombre.');

            cargarDeLocalStorage();
            empleados.push({ id: Date.now().toString(), name, position });
            guardarEnLocalStorage();
            
            document.getElementById('emp-name').value = '';
            renderEmployees();
            populateEmployeeSelects();
        }

        function renderEmployees() {
            const tbody = document.getElementById('employee-table-rows');
            tbody.innerHTML = '';
            if(empleados.length === 0) {
                tbody.innerHTML = `<tr><td colspan="3" style="text-align:center; color:gray;">Sin empleados.</td></tr>`;
                return;
            }
            empleados.forEach(e => {
                tbody.innerHTML += `<tr>
                    <td><strong>${e.name}</strong></td>
                    <td>${e.position}</td>
                    <td><button class="btn btn-danger" style="padding:2px 6px; font-size:11px;" onclick="deleteEmployee('${e.id}')">X</button></td>
                </tr>`;
            });
        }

        function deleteEmployee(id) {
            if(confirm('¿Eliminar empleado?')) {
                cargarDeLocalStorage();
                empleados = empleados.filter(e => e.id !== id);
                guardarEnLocalStorage();
                renderEmployees();
                populateEmployeeSelects();
            }
        }

        function populateEmployeeSelects() {
            const select = document.getElementById('assign-employee');
            const valAnterior = select.value;
            select.innerHTML = '';
            if(empleados.length === 0) {
                select.innerHTML = '<option value="">-- Crea empleados primero --</option>';
                return;
            }
            empleados.forEach(e => {
                select.innerHTML += `<option value="${e.id}">${e.name}</option>`;
            });
            if(valAnterior && empleados.some(e => e.id === valAnterior)) {
                select.value = valAnterior;
            }
        }

        function calculateSingleExit(dayIndex) {
            const entryVal = document.getElementById(`day-entry-${dayIndex}`).value;
            const exitInput = document.getElementById(`day-exit-${dayIndex}`);
            if(!entryVal) { exitInput.value = ''; return; }

            const especiales = ["Descanso", "Vacaciones", "Incapacidad", "Capacitación", "Corporativo", "Otra Tienda"];
            if(especiales.includes(entryVal)) { exitInput.value = `"${entryVal}"`; return; }

            const empId = document.getElementById('assign-employee').value;
            const emp = empleados.find(e => e.id === empId);
            if(!emp) return;

            const horasAAnadir = emp.position === 'Cosmetóloga' ? 9 : 8;
            let [horaMilitar, ampm] = entryVal.split(' ');
            let [horas, minutos] = horaMilitar.split(':').map(Number);

            if(ampm === 'p.m.' && horas !== 12) horas += 12;
            if(ampm === 'a.m.' && horas === 12) horas = 0;

            let totalHoras = horas + horasAAnadir;
            let resultHoras = totalHoras % 12;
            if(resultHoras === 0) resultHoras = 12;
            
            exitInput.value = `${String(resultHoras).padStart(2, '0')}:${String(minutos).padStart(2, '0')}`;
        }

        function saveSchedule() {
            const empId = document.getElementById('assign-employee').value;
            const weekDate = document.getElementById('assign-week-date').value;
            if(!empId || !weekDate) return alert('Selecciona un empleado y una fecha.');

            cargarDeLocalStorage();

            if(!horarios[weekDate]) horarios[weekDate] = {};
            horarios[weekDate][empId] = [];

            for(let i=0; i<7; i++) {
                const entrada = document.getElementById(`day-entry-${i}`).value;
                const salida = document.getElementById(`day-exit-${i}`).value;
                horarios[weekDate][empId].push({ entrada, salida });
            }

            guardarEnLocalStorage();
            alert('Horario guardado con éxito para esta semana.');
        }

        function loadEmployeeSchedule() {
            const empId = document.getElementById('assign-employee').value;
            const weekDate = document.getElementById('assign-week-date').value;

            for(let i=0; i<7; i++) {
                document.getElementById(`day-entry-${i}`).value = '';
                document.getElementById(`day-exit-${i}`).value = '';
            }

            if(weekDate && empId && horarios[weekDate] && horarios[weekDate][empId]) {
                const empData = horarios[weekDate][empId];
                for(let i=0; i<7; i++) {
                    if(empData[i]) {
                        document.getElementById(`day-entry-${i}`).value = empData[i].entrada;
                        document.getElementById(`day-exit-${i}`).value = empData[i].salida;
                    }
                }
            }
        }

        function clearWeekData() {
            const weekDate = document.getElementById('search-week-date').value;
            if(weekDate && confirm('¿Borrar cuadrícula de esta semana?')) {
                cargarDeLocalStorage();
                delete horarios[weekDate];
                guardarEnLocalStorage();
                renderScheduleTable();
            }
        }

        function renderScheduleTable() {
            const weekDate = document.getElementById('search-week-date').value;
            const tbody = document.getElementById('schedule-rows');
            tbody.innerHTML = '';
            if(!weekDate) return;

            cargarDeLocalStorage();

            let partes = weekDate.split('-');
            let baseTime = new Date(Number(partes[0]), Number(partes[1]) - 1, Number(partes[2])).getTime();
            
            const idsDias = ['lunes', 'martes', 'miercoles', 'jueves', 'viernes', 'sabado', 'domingo'];
            const nombresDias = ['lunes', 'martes', 'miércoles', 'jueves', 'viernes', 'sábado', 'domingo'];
            
            idsDias.forEach((idDia, idx) => {
                let unDiaEnMs = idx * 24 * 60 * 60 * 1000;
                let fechaCalculada = new Date(baseTime + unDiaEnMs);
                let numeroDia = String(fechaCalculada.getDate()).padStart(2, '0');
                
                document.getElementById(`head-${idDia}`).innerText = `${nombresDias[idx]} ${numeroDia}`;
            });

            if(empleados.length === 0) {
                tbody.innerHTML = `<tr><td colspan="16" style="padding:15px; color:gray;">Agrega personal en la pestaña Personal.</td></tr>`;
                return;
            }

            const weekData = horarios[weekDate] || {};

            empleados.forEach(emp => {
                const empSched = weekData[emp.id] || Array(7).fill({ entrada: '', salida: '' });
                let rowHtml = `<tr><td style="text-align:left; font-weight:bold;">${emp.name}</td><td>${emp.position}</td>`;

                for(let i=0; i<7; i++) {
                    const cellData = empSched[i] || { entrada: '', salida: '' };
                    let ent = cellData.entrada || '';
                    let sal = cellData.salida || '';
                    
                    if(ent && ent.includes(' ') && !["Descanso", "Vacaciones", "Incapacidad", "Capacitación", "Corporativo", "Otra Tienda"].includes(ent)) {
                        ent = ent.split(' ')[0]; 
                    }

                    const especiales = ["Descanso", "Vacaciones", "Incapacidad", "Capacitación", "Corporativo", "Otra Tienda"];
                    if(especiales.includes(cellData.entrada)) {
                        rowHtml += `<td class="special-cell">"${cellData.entrada}"</td><td class="special-cell">"${cellData.entrada}"</td>`;
                    } else {
                        rowHtml += `<td>${ent}</td><td>${sal}</td>`;
                    }
                }
                rowHtml += `</tr>`;
                tbody.innerHTML += rowHtml;
            });

            for(let k=0; k<2; k++) {
                tbody.innerHTML += `<tr class="empty-row"><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>`;
            }
        }

        // --- SOLUCIÓN DE FONDO: EXTRACCIÓN DE LIENZO INVISIBLE (100% LIMPIO) ---
        function renderizarImagenLimpia(callback) {
            const areaOriginal = document.getElementById('capture-area');
            
            // 1. Creamos un contenedor temporal completamente plano y desenrollado
            const clonOculto = areaOriginal.cloneNode(true);
            
            // 2. Forzamos estilos absolutos de escritorio ancho para eliminar barras de scroll y saltos
            clonOculto.style.position = 'fixed';
            clonOculto.style.top = '0';
            clonOculto.style.left = '-9999px'; // Lo mandamos fuera de la pantalla visible
            clonOculto.style.width = '1300px';  // Tamaño ideal para que quepa todo plano y estirado como en tu ejemplo
            clonOculto.style.overflow = 'visible';
            clonOculto.style.background = '#ffffff';
            clonOculto.style.padding = '15px';
            clonOculto.style.boxSizing = 'content-box';

            // Forzar a la tabla interna del clon a ocupar todo el espacio plano
            const tablaClonada = clonOculto.querySelector('.schedule-table');
            if(tablaClonada) {
                tablaClonada.style.width = '100%';
            }

            document.body.appendChild(clonOculto);

            // 3. Tomamos la foto directo a este clon perfecto
            html2canvas(clonOculto, { 
                scale: 2, // Excelente definición (Retina)
                backgroundColor: "#ffffff",
                windowWidth: 1350 // Emulamos una ventana gigante
            }).then(canvas => {
                // 4. Eliminamos el clon de la memoria inmediatamente
                document.body.removeChild(clonOculto);
                callback(canvas);
            }).catch(err => {
                if(clonOculto.parentNode) document.body.removeChild(clonOculto);
                console.error(err);
                alert('Ocurrió un detalle al procesar la imagen.');
            });
        }

        function exportToImage() {
            const dateVal = document.getElementById('search-week-date').value;
            if(!dateVal) return alert('Selecciona una semana.');

            renderizarImagenLimpia(canvas => {
                const link = document.createElement('a');
                link.download = `Horario_Semana_${dateVal}.png`;
                link.href = canvas.toDataURL('image/png');
                link.click();
            });
        }

        function shareToWhatsApp() {
            const dateVal = document.getElementById('search-week-date').value;
            if(!dateVal) return alert('Selecciona una semana antes de compartir.');

            renderizarImagenLimpia(canvas => {
                canvas.toBlob(blob => {
                    if (!blob) return alert('Error al empaquetar la imagen.');
                    
                    const nombreArchivo = `Horario_Semana_${dateVal}.png`;
                    const archivoCompartir = new File([blob], nombreArchivo, { type: 'image/png' });

                    if (navigator.canShare && navigator.canShare({ files: [archivoCompartir] })) {
                        navigator.share({
                            files: [archivoCompartir],
                            title: `Horario Polanco - Semana ${dateVal}`,
                            text: `Comparto la cuadrícula completa de horarios para la semana ${dateVal}.`
                        })
                        .catch(err => console.log('Envío cancelado.'));
                    } else {
                        alert('Tu dispositivo no soporta envío directo automático. Se descargará la imagen completa y abriremos WhatsApp Web para que la adjuntes manualmente.');
                        
                        const link = document.createElement('a');
                        link.download = nombreArchivo;
                        link.href = canvas.toDataURL('image/png');
                        link.click();
                        
                        window.open(`https://web.whatsapp.com/`, '_blank');
                    }
                }, 'image/png');
            });
        }

        if ('serviceWorker' in navigator) {
            navigator.serviceWorker.register('./service-worker.js').catch(err => {
                console.log("Modo Web Local Activado (Sin SW offline)");
            });
        }
    </script>
</body>
</html>
