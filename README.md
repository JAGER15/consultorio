# consultorio
<html>
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Sistema IMC</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
            background-color: #f5f5f5;
        }
        .container {
            background-color: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
        }
        .form {
            margin-bottom: 30px;
            padding: 20px;
            background-color: #f9f9f9;
            border-radius: 8px;
        }
        .form_label {
            display: block;
            margin-top: 10px;
            font-weight: bold;
        }
        .form_input {
            width: 100%;
            padding: 8px;
            margin-top: 5px;
            border: 1px solid #ddd;
            border-radius: 4px;
            box-sizing: border-box;
        }
        button {
            margin-top: 15px;
            padding: 10px 20px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            margin-right: 10px;
        }
        button:hover {
            background-color: #45a049;
        }
        h1 {
            color: #333;
            border-bottom: 2px solid #4CAF50;
            padding-bottom: 10px;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 12px;
            text-align: left;
        }
        th {
            background-color: #4CAF50;
            color: white;
        }
        tr:nth-child(even) {
            background-color: #f2f2f2;
        }
        .imc-low { background-color: #ffeb3b; }
        .imc-normal { background-color: #8bc34a; }
        .imc-overweight { background-color: #ff9800; }
        .imc-obesity { background-color: #f44336; color: white; }
        .btn-modify { 
            background-color: #2196F3;
            padding: 5px 10px;
            color: white;
            text-decoration: none;
            border-radius: 4px;
            margin-right: 5px;
            display: inline-block;
            border: none;
            cursor: pointer;
        }
        .btn-delete { 
            background-color: #f44336;
            padding: 5px 10px;
            color: white;
            text-decoration: none;
            border-radius: 4px;
            margin-right: 5px;
            display: inline-block;
            border: none;
            cursor: pointer;
        }
        .alert {
            padding: 15px;
            margin-bottom: 20px;
            border: 1px solid transparent;
            border-radius: 4px;
            display: none;
        }
        .alert-success {
            color: #3c763d;
            background-color: #dff0d8;
            border-color: #d6e9c6;
            display: block;
        }
        .alert-error {
            color: #a94442;
            background-color: #f2dede;
            border-color: #ebccd1;
            display: block;
        }
        .form-row {
            display: flex;
            gap: 20px;
            margin-bottom: 15px;
        }
        .form-group {
            flex: 1;
        }
        .actions-cell {
            min-width: 150px;
        }
        .empty-message {
            text-align: center;
            color: #666;
            padding: 20px;
            font-style: italic;
        }
    </style>
</head>
<body>
    <div class="container">
        <!-- Mensajes -->
        <div id="message" class="alert"></div>
        
        <!-- Formulario de datos -->
        <div class="form">
            <h1 id="form-title">Datos de la Persona</h1>
            <form id="imc-form">
                <input type="hidden" id="edit-id" value="" />
                
                <div class="form-row">
                    <div class="form-group">
                        <label class="form_label">ID</label>
                        <input class="form_input" type="text" id="id" required="" />
                    </div>
                    <div class="form-group">
                        <label class="form_label">Nombre</label>
                        <input class="form_input" type="text" id="nombre" required="" />
                    </div>
                </div>
                
                <div class="form-row">
                    <div class="form-group">
                        <label class="form_label">Peso (kg)</label>
                        <input class="form_input" type="number" step="0.01" id="peso" required="" />
                    </div>
                    <div class="form-group">
                        <label class="form_label">Altura (m)</label>
                        <input class="form_input" type="number" step="0.01" id="altura" required="" />
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


      
    </div>
   
  </body>
</html>
