# consultorio

<html>
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Sistema IMC</title>
 * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            
        }
        .container {
            max-width: 1200px;
            margin: 0 auto;
        }
        body {
            background: linear-gradient(135deg, #ff9800, #ffc107);
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 20px;
        }
        
        .form {
            background-color: white;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
            padding: 40px;
            width: 100%;
            max-width: 500px;
            transition: transform 0.3s ease;
        }
        
        .form:hover {
            transform: translateY(-5px);
        }
        
        h1 {
            color: #ff6600;
            text-align: center;
            margin-bottom: 30px;
            font-size: 28px;
            position: relative;
            padding-bottom: 15px;
        }
        
        h1:after {
            content: '';
            position: absolute;
            bottom: 0;
            left: 50%;
            transform: translateX(-50%);
            width: 80px;
            height: 4px;
            background: linear-gradient(to right, #ff9800, #ffc107);
            border-radius: 2px;
        }
        
        .form_label {
            display: block;
            margin-bottom: 8px;
            font-weight: 600;
            color: #333;
            font-size: 16px;
        }
        
        .form_input {
            width: 100%;
            padding: 14px 16px;
            margin-bottom: 25px;
            border: 2px solid #ffc107;
            border-radius: 8px;
            font-size: 16px;
            transition: all 0.3s;
            background-color: #fffdf5;
        }
        
        .form_input:focus {
            outline: none;
            border-color: #ff9800;
            box-shadow: 0 0 0 3px rgba(255, 152, 0, 0.2);
            background-color: white;
        }
        
        button {
            width: 100%;
            padding: 15px;
            border: none;
            border-radius: 8px;
            font-size: 18px;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.3s;
            margin-top: 10px;
        }
        
        button[type="submit"] {
            background: linear-gradient(to right, #ff9800, #ff6600);
            color: white;
            box-shadow: 0 4px 15px rgba(255, 152, 0, 0.4);
        }
        
        button[type="submit"]:hover {
            background: linear-gradient(to right, #ff6600, #ff9800);
            box-shadow: 0 6px 20px rgba(255, 152, 0, 0.6);
            transform: translateY(-2px);
        }
        
        button[type="button"] {
            background: linear-gradient(to right, #ffc107, #ff9800);
            color: white;
            box-shadow: 0 4px 15px rgba(255, 193, 7, 0.4);
        }
        
        button[type="button"]:hover {
            background: linear-gradient(to right, #ff9800, #ffc107);
            box-shadow: 0 6px 20px rgba(255, 193, 7, 0.6);
            transform: translateY(-2px);
        }
        
        .form-footer {
            text-align: center;
            margin-top: 25px;
            color: #666;
            font-size: 14px;
        }
        
        @media (max-width: 600px) {
            .form {
                padding: 30px 20px;
            }
            
            h1 {
                font-size: 24px;
            }
        }
</head>
<body>
    <div class="container">
        <!-- Mensajes -->
        <div id="message" class="alert"></div>
        
        <!-- Formulario de datos -->
        <div class="form">
            <h1 id="form-title">Datos de la Persona</h1>
            <form id="imc-form">
                <input type="hidden" id="edit-id" value="">
                
                <div class="form-row">
                    <div class="form-group">
                        <label class="form_label">ID</label>
                        <input class="form_input" type="text" id="id" required>
                    </div>
                    <div class="form-group">
                        <label class="form_label">Nombre</label>
                        <input class="form_input" type="text" id="nombre" required>
                    </div>
                </div>
                
                <div class="form-row">
                    <div class="form-group">
                        <label class="form_label">Peso (kg)</label>
                        <input class="form_input" type="number" step="0.01" id="peso" required>
                    </div>
                    <div class="form-group">
                        <label class="form_label">Altura (m)</label>
                        <input class="form_input" type="number" step="0.01" id="altura" required>
                    </div>
                </div>
                
                <button type="submit" id="submit-btn">GUARDAR</button>
                <button type="button" id="cancel-btn" style="display: none;">CANCELAR</button>
            </form>
        </div>
        
        <!-- Tabla de datos -->
        <h1>Registros de Personas</h1>
        <div id="table-container">
            <table>
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Nombre</th>
                        <th>Peso (kg)</th>
                        <th>Altura (m)</th>
                        <th>IMC</th>
                        <th>Estado</th>
                        <th>Consejo</th>
                        <th class="actions-cell">Acciones</th>
                    </tr>
                </thead>
                <tbody id="table-body">
                    <!-- Los datos se cargarán aquí con JavaScript -->
                </tbody>
            </table>
            <p id="empty-message" class="empty-message">No hay datos registrados. Agrega una persona usando el formulario arriba.</p>
        </div>
    </div>

    <script>
        // Sistema de almacenamiento
        const STORAGE_KEY = 'imc_data';
        
        // Cargar datos al iniciar
        let personas = JSON.parse(localStorage.getItem(STORAGE_KEY)) || [];
        let editMode = false;
        let currentEditId = '';
        
        // Mostrar mensaje
        function showMessage(text, type) {
            const messageDiv = document.getElementById('message');
            messageDiv.textContent = text;
            messageDiv.className = `alert alert-${type}`;
            messageDiv.style.display = 'block';
            
            setTimeout(() => {
                messageDiv.style.display = 'none';
            }, 3000);
        }
        
        // Calcular IMC y determinar estado
        function calcularIMC(peso, altura) {
            return peso / (altura * altura);
        }
        
        function determinarEstado(imc) {
            if (imc < 18.5) {
                return {
                    estado: "Bajo peso",
                    consejo: "Debes aumentar de peso. Consulta a un nutricionista.",
                    clase: "imc-low"
                };
            } else if (imc >= 18.5 && imc < 25) {
                return {
                    estado: "Peso normal",
                    consejo: "¡Excelente! Mantén tus hábitos saludables.",
                    clase: "imc-normal"
                };
            } else if (imc >= 25 && imc < 30) {
                return {
                    estado: "Sobrepeso",
                    consejo: "Necesitas hacer ejercicio y mejorar tu alimentación.",
                    clase: "imc-overweight"
                };
            } else {
                return {
                    estado: "Obesidad",
                    consejo: "Consulta a un médico y nutricionista urgentemente.",
                    clase: "imc-obesity"
                };
            }
        }
        
        // Renderizar tabla
        function renderTable() {
            const tableBody = document.getElementById('table-body');
            const emptyMessage = document.getElementById('empty-message');
            
            tableBody.innerHTML = '';
            
            if (personas.length === 0) {
                emptyMessage.style.display = 'block';
                return;
            }
            
            emptyMessage.style.display = 'none';
            
            personas.forEach(persona => {
                const imc = calcularIMC(persona.peso, persona.altura);
                const estadoInfo = determinarEstado(imc);
                
                const row = document.createElement('tr');
                row.innerHTML = `
                    <td>${persona.id}</td>
                    <td>${persona.nombre}</td>
                    <td>${persona.peso.toFixed(2)} kg</td>
                    <td>${persona.altura.toFixed(2)} m</td>
                    <td class="${estadoInfo.clase}">${imc.toFixed(2)}</td>
                    <td class="${estadoInfo.clase}">${estadoInfo.estado}</td>
                    <td>${estadoInfo.consejo}</td>
                    <td>
                        <button class="btn-modify" onclick="editarPersona('${persona.id}')">Modificar</button>
                        <button class="btn-delete" onclick="eliminarPersona('${persona.id}')">Eliminar</button>
                    </td>
                `;
                tableBody.appendChild(row);
            });
        }
        
        // Manejar envío del formulario
        document.getElementById('imc-form').addEventListener('submit', function(e) {
            e.preventDefault();
            
            const id = document.getElementById('id').value;
            const nombre = document.getElementById('nombre').value;
            const peso = parseFloat(document.getElementById('peso').value);
            const altura = parseFloat(document.getElementById('altura').value);
            
            if (editMode) {
                // Modo edición
                const index = personas.findIndex(p => p.id === currentEditId);
                if (index !== -1) {
                    personas[index] = { id, nombre, peso, altura };
                    localStorage.setItem(STORAGE_KEY, JSON.stringify(personas));
                    showMessage('Datos actualizados correctamente!', 'success');
                    
                    // Salir del modo edición
                    cancelarEdicion();
                }
            } else {
                // Modo guardar nuevo
                // Verificar si el ID ya existe
                if (personas.some(p => p.id === id)) {
                    showMessage('Error: El ID ya existe. Use otro ID.', 'error');
                    return;
                }
                
                personas.push({ id, nombre, peso, altura });
                localStorage.setItem(STORAGE_KEY, JSON.stringify(personas));
                showMessage('Datos guardados correctamente!', 'success');
                
                // Limpiar formulario
                document.getElementById('imc-form').reset();
            }
            
            renderTable();
        });
        
        // Editar persona
        function editarPersona(id) {
            const persona = personas.find(p => p.id === id);
            if (!persona) return;
            
            editMode = true;
            currentEditId = id;
            
            // Llenar formulario con datos
            document.getElementById('edit-id').value = id;
            document.getElementById('id').value = persona.id;
            document.getElementById('nombre').value = persona.nombre;
            document.getElementById('peso').value = persona.peso;
            document.getElementById('altura').value = persona.altura;
            
            // Cambiar interfaz
            document.getElementById('form-title').textContent = 'Editar Persona';
            document.getElementById('submit-btn').textContent = 'ACTUALIZAR';
            document.getElementById('cancel-btn').style.display = 'inline-block';
        }
        
        // Cancelar edición
        function cancelarEdicion() {
            editMode = false;
            currentEditId = '';
            
            // Restaurar interfaz
            document.getElementById('form-title').textContent = 'Datos de la Persona';
            document.getElementById('submit-btn').textContent = 'GUARDAR';
            document.getElementById('cancel-btn').style.display = 'none';
            
            // Limpiar formulario
            document.getElementById('imc-form').reset();
            document.getElementById('edit-id').value = '';
        }
        
        // Eliminar persona
        function eliminarPersona(id) {
            if (confirm('¿Seguro que deseas eliminar a esta persona?')) {
                personas = personas.filter(p => p.id !== id);
                localStorage.setItem(STORAGE_KEY, JSON.stringify(personas));
                renderTable();
                showMessage('Persona eliminada correctamente!', 'success');
                
                // Si estábamos editando esta persona, cancelar edición
                if (editMode && currentEditId === id) {
                    cancelarEdicion();
                }
            }
        }
        
        // Botón cancelar
        document.getElementById('cancel-btn').addEventListener('click', cancelarEdicion);
        
        // Inicializar
        renderTable();
    </script>
</body>
</html>
