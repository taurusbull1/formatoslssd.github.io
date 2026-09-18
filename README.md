<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>LSSD - Generador de Registros</title>
    <style>
        :root {
            --bg-color: #0f0f11;
            --card-bg: #18191c;
            --border-color: #2b2d31;
            --accent-bronze: #c5a059;
            --accent-hover: #e0b86c;
            --text-main: #e0e0e0;
            --text-muted: #a0a0a0;
            --input-bg: #090a0b;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-main);
            max-width: 750px;
            margin: 40px auto;
            padding: 20px;
        }

        .container {
            background-color: var(--card-bg);
            border: 1px solid var(--border-color);
            border-top: 4px solid var(--accent-bronze);
            border-radius: 8px;
            padding: 25px;
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.5);
        }

        h2 {
            text-align: center;
            color: var(--accent-bronze);
            margin-top: 0;
            margin-bottom: 20px;
            font-size: 1.5rem;
            letter-spacing: 1px;
            text-transform: uppercase;
            border-bottom: 1px solid var(--border-color);
            padding-bottom: 12px;
        }

        .form-group {
            margin-bottom: 18px;
        }

        label {
            display: block;
            font-weight: 600;
            margin-bottom: 6px;
            font-size: 0.85rem;
            color: var(--accent-bronze);
            text-transform: uppercase;
            letter-spacing: 0.5px;
        }

        input[type="text"], select, textarea {
            width: 100%;
            padding: 12px;
            background-color: var(--input-bg);
            border: 1px solid var(--border-color);
            border-radius: 5px;
            color: var(--text-main);
            box-sizing: border-box;
            font-size: 14px;
            transition: border-color 0.2s, box-shadow 0.2s;
        }

        select {
            cursor: pointer;
            font-weight: bold;
            color: var(--accent-bronze);
        }

        input[type="text"]:focus, select:focus, textarea:focus {
            outline: none;
            border-color: var(--accent-bronze);
            box-shadow: 0 0 5px rgba(197, 160, 89, 0.3);
        }

        input[type="text"]::placeholder, textarea::placeholder {
            color: var(--text-muted);
            opacity: 0.6;
        }

        .template-selector {
            background-color: #121315;
            padding: 15px;
            border-radius: 6px;
            border: 1px dashed var(--accent-bronze);
            margin-bottom: 25px;
        }

        .output-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-top: 30px;
            margin-bottom: 10px;
        }

        .btn-copy {
            background-color: var(--accent-bronze);
            color: #000;
            border: none;
            padding: 9px 18px;
            font-size: 13px;
            font-weight: bold;
            border-radius: 4px;
            cursor: pointer;
            transition: background-color 0.2s, transform 0.1s;
        }

        .btn-copy:hover {
            background-color: var(--accent-hover);
        }

        .btn-copy:active {
            transform: scale(0.98);
        }

        textarea.output-box {
            height: 220px;
            font-family: 'Consolas', 'Courier New', monospace;
            font-size: 13px;
            color: #4af626;
            resize: vertical;
        }
        
        input[type="text"].title-box {
            font-family: 'Consolas', 'Courier New', monospace;
            font-size: 13px;
            color: #4af626;
            background-color: var(--input-bg);
        }

        .toast {
            visibility: hidden;
            background-color: var(--accent-bronze);
            color: #000;
            font-weight: bold;
            text-align: center;
            border-radius: 4px;
            padding: 5px 10px;
            font-size: 12px;
            margin-right: 10px;
            opacity: 0;
            transition: opacity 0.3s;
        }

        .toast.show {
            visibility: visible;
            opacity: 1;
        }
    </style>
</head>
<body>

    <div class="container">
        <h2>LSSD — Generador de Registros</h2>

        <!-- Selector de Plantilla -->
        <div class="template-selector">
            <label for="templateSelect">SELECCIONAR FORMATO DE REGISTRO:</label>
            <select id="templateSelect" onchange="cambiarPlantilla()">
                <option value="historical_record">New Historical Record</option>
                <option value="eow">End of Watch (EOW)</option>
                <option value="exec_mention">Executive Mention</option>
                <option value="personnel_change">Personnel Change With Immediate Effect</option>
                <option value="disciplinary_action">Personnel Disciplinary Action</option>
                <option value="tactical_debrief">Personnel Tactical Debrief</option>
                <option value="completed_tactical_debrief">Completed Personnel Tactical Debrief</option>
            </select>
        </div>

        <!-- Contenedor dinámico de formularios -->
        <div id="dynamicForm"></div>

        <!-- Contenedor del Código del Título (Solo visible en Historical Record) -->
        <div id="titleContainer" style="display: none;">
            <div class="output-header" style="margin-top: 20px;">
                <label for="outputTitle" style="margin: 0;">CÓDIGO DEL TÍTULO:</label>
                <div style="display: flex; align-items: center;">
                    <span id="toastTitle" class="toast">¡COPIADO!</span>
                    <button class="btn-copy" onclick="copiarTitulo()">Copiar Título</button>
                </div>
            </div>
            <input type="text" id="outputTitle" class="title-box" readonly>
        </div>

        <!-- Encabezado con Botón de Copiar Principal -->
        <div class="output-header">
            <label for="outputBbcode" style="margin: 0;">CÓDIGO BBCODE GENERADO:</label>
            <div style="display: flex; align-items: center;">
                <span id="toast" class="toast">¡COPIADO!</span>
                <button class="btn-copy" onclick="copiarAlPortapapeles()">Copiar BBCode</button>
            </div>
        </div>

        <!-- Salida BBCode -->
        <textarea id="outputBbcode" class="output-box" readonly></textarea>
    </div>

    <script>
        const plantillas = {
            historical_record: {
                nombre: "New Historical Record",
                campos: [
                    { id: "empNameRank", label: "Employee Name & Rank", placeholder: "Nombre Apellido — Rango", default: "" },
                    { id: "serialNumber", label: "Serial Number", placeholder: "Placa", default: "" },
                    { id: "assignment", label: "Assignment", placeholder: "Asignación", default: "" },
                    { id: "lastName", label: "Apellido (Para título)", placeholder: "Ej: Pérez", default: "" },
                    { id: "firstName", label: "Nombre (Para título)", placeholder: "Ej: Juan", default: "" }
                ],
                bbcode: (datos) => {
                    const empNameRank = datos.empNameRank || "{Nombre Apellido — Rango}";
                    const serialNumber = datos.serialNumber || "{Placa}";
                    const assignment = datos.assignment || "{Asignación}";

                    return `[divboxlssd=white]
[font=Arial]
[center]
[img]https://i.imgur.com/otXEWuZ.png[/img]

[size=125][b]EMPLOYEE RECORD[/b][/size]
LOS SANTOS SHERIFF'S DEPARTMENT[/center][/font]
[divider]
[font=Arial]
[b]EMPLOYEE NAME & RANK:[/b] ${empNameRank}
[b]SERIAL NUMBER:[/b] ${serialNumber}
[b]ASSIGNMENT:[/b] ${assignment}
[color=#FFFFFF]SPACER[/color]
[color=#FFFFFF]SPACER[/color]
[center][img]https://i.imgur.com/vggSUzN.png[/img][/center][/font][/divboxlssd]`;
                }
            },
            eow: {
                nombre: "End of Watch",
                campos: [
                    { id: "empName", label: "Employee Name & Rank", placeholder: "Nombre Apellido — Rango", default: "" },
                    { id: "compDate", label: "Completion Date", placeholder: "DD/MM/AAAA", default: "" },
                    { id: "reason", label: "Reason", placeholder: "MOTIVO", default: "TERMINATION" }
                ],
                bbcode: (datos) => {
                    const empName = datos.empName || "{Nombre Apellido y Rango}";
                    const compDate = datos.compDate || "{DD/MM/AAAA}";
                    const reason = datos.reason || "{RAZON}";

                    return `[divboxlssd=white]
[font=Arial]
[center]
[img]https://i.imgur.com/otXEWuZ.png[/img]

[size=125][b]END OF WATCH[/b][/size]
LOS SANTOS SHERIFF'S DEPARTMENT[/center][/font]
[divider]
[font=Arial]
[b]EMPLOYEE NAME & RANK:[/b] ${empName}
[b]COMPLETION DATE[/b] ${compDate}
[b]REASON:[/b] ${reason}
[/font][/divboxlssd]`;
                }
            },
            exec_mention: {
                nombre: "Executive Mention",
                campos: [
                    { id: "empNameRank", label: "Employee Name & Rank", placeholder: "Nombre Apellido — Rango", default: "" },
                    { id: "assignment", label: "Assignment", placeholder: "ASIGNACION", default: "" },
                    { id: "supNameRank", label: "Supervisor Name & Rank", placeholder: "Nombre Apellido — Rango del supervisor", default: "" },
                    { id: "supAssignment", label: "Supervisor Assignment", placeholder: "ASIGNACION_SUPERVISOR", default: "" },
                    { id: "mentionedBy", label: "Mentioned By (Name & Rank)", placeholder: "Nombre Apellido — Rango del que menciona", default: "" },
                    { id: "mentionedAssignment", label: "Mentioned By Assignment", placeholder: "ASIGNACION DEL QUE MENCIONA", default: "" },
                    { id: "reason", label: "Reason", placeholder: "RAZON DE QUIEN MENCIONA", default: "" }
                ],
                bbcode: (datos) => {
                    const empNameRank = datos.empNameRank || "{Nombre Apellido — Rango}";
                    const assignment = datos.assignment || "{ASIGNACION}";
                    const supNameRank = datos.supNameRank || "{Nombre Apellido — Rango del supervisor}";
                    const supAssignment = datos.supAssignment || "{ASIGNACION_SUPERVISOR}";
                    const mentionedBy = datos.mentionedBy || "{Nombre Apellido — Rango del que menciona}";
                    const mentionedAssignment = datos.mentionedAssignment || "{ASIGNACION DEL QUE MENCIONA}";
                    const reason = datos.reason || "{RAZON DE QUIEN MENCIONA}";

                    return `[divboxlssd=white]
[font=Arial]
[center]
[img]https://i.imgur.com/otXEWuZ.png[/img]

[size=125][b]EXECUTIVE MENTION[/b][/size]
LOS SANTOS SHERIFF'S DEPARTMENT[/center][/font]
[divider]
[font=Arial]
[b]EMPLOYEE NAME & RANK:[/b] ${empNameRank}
[b]ASSIGNMENT:[/b] ${assignment}
[color=#FFFFFF].[/color]
[b]SUPERVISOR:[/b] ${supNameRank}
[b]ASSIGNMENT:[/b] ${supAssignment}
[color=#FFFFFF].[/color]
[b]MENTIONED BY:[/b] ${mentionedBy}
[b]ASSIGNMENT:[/b] ${mentionedAssignment}
[b]REASON:[/b] ${reason}
[/font][/divboxlssd]`;
                }
            },
            personnel_change: {
                nombre: "Personnel Change With Immediate Effect",
                campos: [
                    { id: "empNameRank", label: "Employee Name & Rank", placeholder: "Nombre Apellido — Rango", default: "" },
                    { id: "oldAssignment", label: "Old Assignment", placeholder: "Asignación, estación, p.ej:  Patrol Deputy, Davis Sheriff's Station", default: "" },
                    { id: "newAssignment", label: "New Assignment", placeholder: "Asignación, estación, p.ej:  Patrol Deputy, Davis Sheriff's Station", default: "" },
                    { id: "supNameRank", label: "Supervisor Name & Rank", placeholder: "Nombre Apellido — Rango del supervisor", default: "" },
                    { id: "supAssignment", label: "Supervisor Assignment", placeholder: "Asignación, estación, p.ej:  Watch Commander, Davis Sheriff's Station", default: "" },
                    { id: "type", label: "Type", placeholder: "TIPO DE CAMBIO (Ej: REASIGNADO, PROMOVIDO, DEGRADADO)", default: "" }
                ],
                bbcode: (datos) => {
                    const empNameRank = datos.empNameRank || "{Nombre Apellido — Rango}";
                    const oldAssignment = datos.oldAssignment || "{Asignación, estación, p.ej:  Patrol Deputy, Davis Sheriff's Station}";
                    const newAssignment = datos.newAssignment || "{Asignación, estación, p.ej:  Patrol Deputy, Davis Sheriff's Station}";
                    const supNameRank = datos.supNameRank || "{Nombre Apellido — Rango del supervisor}";
                    const supAssignment = datos.supAssignment || "{Asignación, estación, p.ej:  Watch Commander, Davis Sheriff's Station}";
                    const type = datos.type || "{TIPO DE CAMBIO}";

                    return `[divboxlssd=white]
[font=Arial]
[center]
[img]https://i.imgur.com/otXEWuZ.png[/img]

[size=125][b]PERSONNEL CHANGE WITH IMMEDIATE EFFECT[/b][/size]
LOS SANTOS SHERIFF'S DEPARTMENT[/center][/font]
[divider]
[font=Arial]
[b]EMPLOYEE NAME & RANK:[/b] ${empNameRank}
[b]OLD ASSIGNMENT:[/b] ${oldAssignment}
[b]NEW ASSIGNMENT:[/b] ${newAssignment}
[color=#FFFFFF].[/color]
[b]SUPERVISOR NAME & RANK:[/b] ${supNameRank}
[b]ASSIGNMENT:[/b] ${supAssignment}
[b]TYPE:[/b] ${type}
[/font][/divboxlssd]`;
                }
            },
            disciplinary_action: {
                nombre: "Personnel Disciplinary Action",
                campos: [
                    { id: "empNameRank", label: "Employee Name & Rank", placeholder: "Nombre Apellido — Rango", default: "" },
                    { 
                        id: "type", 
                        label: "Type", 
                        type: "select", 
                        options: ["AMONESTACIÓN", "SUSPENSIÓN", "DEGRADO", "ADVERTENCIA FINAL", "EXPULSIÓN", "(( STRIKE X/3 ))"],
                        default: "AMONESTACIÓN" 
                    },
                    { id: "strikeNumber", label: "¿Número de strike?", placeholder: "Ej: 1, 2 o 3", default: "", hidden: true },
                    { id: "reason", label: "Reason", type: "textarea", placeholder: "Motivos aquí", default: "" }
                ],
                bbcode: (datos) => {
                    const empNameRank = datos.empNameRank || "{Nombre Apellido — Rango}";
                    let type = datos.type || "{AMONESTACIÓN / SUSPENSIÓN / DEGRADO / ADVERTENCIA FINAL / EXPULSIÓN / (( STRIKE X/3 ))}";
                    
                    if (datos.type === "(( STRIKE X/3 ))") {
                        const strike = datos.strikeNumber || "X";
                        type = `(( STRIKE ${strike}/3 ))`;
                    }

                    const reason = datos.reason || "{Motivos aqui}";

                    return `[divboxlssd=white]
[font=Arial]
[center]
[img]https://i.imgur.com/otXEWuZ.png[/img]

[size=125][b]PERSONNEL DISCIPLINARY ACTION[/b][/size]
LOS SANTOS SHERIFF'S DEPARTMENT[/center][/font]
[divider]
[font=Arial]
[b]EMPLOYEE NAME & RANK:[/b] ${empNameRank}
[color=#FFFFFF].[/color]
[b]TYPE:[/b] ${type}
[b]REASON:[/b] ${reason}
[/font][/divboxlssd]`;
                }
            },
            tactical_debrief: {
                nombre: "Personnel Tactical Debrief",
                campos: [
                    { id: "resolverNameRank", label: "Resolver Name & Rank", placeholder: "Nombre Apellido — Rango", default: "" },
                    { id: "reason", label: "Reason", type: "textarea", placeholder: "Motivos aquí", default: "" }
                ],
                bbcode: (datos) => {
                    const resolverNameRank = datos.resolverNameRank || "{Nombre Apellido — Rango}";
                    const reason = datos.reason || "{Motivos aqui}";

                    return `[divboxlssd=white]
[font=Arial]
[center]
[img]https://i.imgur.com/otXEWuZ.png[/img]

[size=125][b]PERSONNEL TACTICAL DEBRIEF[/b][/size]
LOS SANTOS SHERIFF'S DEPARTMENT[/center][/font]
[divider]
[font=Arial]
[b]RESOLVER NAME & RANK:[/b] ${resolverNameRank}
[color=#FFFFFF].[/color]
[b]REASON:[/b] ${reason}
[/font][/divboxlssd]`;
                }
            },
            completed_tactical_debrief: {
                nombre: "Completed Personnel Tactical Debrief",
                campos: [
                    { id: "instructorNameRank", label: "Instructor Name & Rank", placeholder: "Nombre Apellido — Rango", default: "" },
                    { id: "date", label: "Date", placeholder: "DD/MM/AAAA", default: "" }
                ],
                bbcode: (datos) => {
                    const instructorNameRank = datos.instructorNameRank || "{Nombre Apellido — Rango}";
                    const date = datos.date || "{DD/MM/AAAA}";

                    return `[divboxlssd=white]
[font=Arial]
[center]
[img]https://i.imgur.com/otXEWuZ.png[/img]

[size=125][b]COMPLETED PERSONNEL TACTICAL DEBRIEF[/b][/size]
LOS SANTOS SHERIFF'S DEPARTMENT[/center][/font]
[divider]
[font=Arial]
[b]INSTRUCTOR NAME & RANK:[/b] ${instructorNameRank}
[b]DATE:[/b] ${date}
[/font][/divboxlssd]`;
                }
            }
        };

        let plantillaActual = "historical_record";

        function cambiarPlantilla() {
            plantillaActual = document.getElementById('templateSelect').value;
            const container = document.getElementById('dynamicForm');
            container.innerHTML = "";

            const config = plantillas[plantillaActual];

            config.campos.forEach(campo => {
                const group = document.createElement('div');
                group.className = 'form-group';
                group.id = 'group_' + campo.id; 

                if (campo.hidden) {
                    group.style.display = 'none';
                }

                const label = document.createElement('label');
                label.setAttribute('for', campo.id);
                label.innerText = campo.label + ":";
                group.appendChild(label);

                if (campo.type === "select") {
                    const select = document.createElement('select');
                    select.id = campo.id;
                    
                    select.onchange = (e) => {
                        if (campo.id === 'type' && plantillaActual === 'disciplinary_action') {
                            const strikeGroup = document.getElementById('group_strikeNumber');
                            if (strikeGroup) {
                                strikeGroup.style.display = e.target.value === '(( STRIKE X/3 ))' ? 'block' : 'none';
                            }
                        }
                        generarBBCode();
                    };

                    campo.options.forEach(opt => {
                        const option = document.createElement('option');
                        option.value = opt;
                        option.innerText = opt;
                        if (opt === campo.default) option.selected = true;
                        select.appendChild(option);
                    });
                    group.appendChild(select);
                } else if (campo.type === "textarea") {
                    const textarea = document.createElement('textarea');
                    textarea.id = campo.id;
                    textarea.placeholder = campo.placeholder;
                    textarea.value = campo.default;
                    textarea.style.height = "90px";
                    textarea.oninput = generarBBCode;
                    group.appendChild(textarea);
                } else {
                    const input = document.createElement('input');
                    input.type = 'text';
                    input.id = campo.id;
                    input.placeholder = campo.placeholder;
                    input.value = campo.default;
                    input.oninput = generarBBCode;
                    group.appendChild(input);
                }

                container.appendChild(group);
            });

            generarBBCode();
        }

        function generarBBCode() {
            const config = plantillas[plantillaActual];
            const datos = {};

            config.campos.forEach(campo => {
                const el = document.getElementById(campo.id);
                datos[campo.id] = el ? el.value.trim() : "";
            });

            const resultado = config.bbcode(datos);
            document.getElementById('outputBbcode').value = resultado;

            // Gestión del Código del Título (Solo aplicable a New Historical Record)
            const titleContainer = document.getElementById('titleContainer');
            if (plantillaActual === 'historical_record') {
                titleContainer.style.display = 'block';
                const apellido = datos.lastName || "Apellido";
                const nombre = datos.firstName || "Nombre";
                document.getElementById('outputTitle').value = `[EMPLOYEE RECORD] ${apellido}, ${nombre}`;
            } else {
                titleContainer.style.display = 'none';
            }
        }

        function copiarAlPortapapeles() {
            const outputArea = document.getElementById('outputBbcode');
            outputArea.select();
            outputArea.setSelectionRange(0, 99999);
            navigator.clipboard.writeText(outputArea.value);

            const toast = document.getElementById('toast');
            toast.classList.add('show');
            setTimeout(() => {
                toast.classList.remove('show');
            }, 1500);
        }

        function copiarTitulo() {
            const outputTitle = document.getElementById('outputTitle');
            outputTitle.select();
            outputTitle.setSelectionRange(0, 99999);
            navigator.clipboard.writeText(outputTitle.value);

            const toastTitle = document.getElementById('toastTitle');
            toastTitle.classList.add('show');
            setTimeout(() => {
                toastTitle.classList.remove('show');
            }, 1500);
        }

        cambiarPlantilla();
    </script>

</body>
</html>
