<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simulador de Caja Registradora Pro</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; background-color: #f4f4f9; }
        .container { max-width: 500px; background: white; padding: 20px; border-radius: 8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1); margin: auto; }
        .form-group { display: flex; gap: 8px; margin-bottom: 12px; }
        input, select { padding: 8px; border: 1px solid #ccc; border-radius: 4px; font-size: 14px; }
        input[type="text"].desc-input { width: 40%; }
        input[type="number"].price-input { width: 25%; }
        select.iva-select { width: 20%; }
        .btn-add { background-color: #007bff; color: white; border: none; cursor: pointer; border-radius: 4px; font-weight: bold; width: 15%; }
        .btn-add:hover { background-color: #0056b3; }
        .lista-carrito { background: #f8f9fa; padding: 10px; border-radius: 4px; margin-bottom: 15px; max-height: 120px; overflow-y: auto; font-size: 14px; }
        .item-carrito { display: flex; justify-content: space-between; padding: 4px 0; border-bottom: 1px solid #eee; }
        .grid-botones { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 10px; }
        button.action-btn { color: white; border: none; padding: 10px 15px; cursor: pointer; border-radius: 4px; font-size: 14px; width: 100%; }
        .btn-success { background-color: #28a745; } .btn-success:hover { background-color: #218838; }
        .btn-danger { background-color: #dc3545; } .btn-danger:hover { background-color: #bd2130; }
        .btn-info { background-color: #17a2b8; } .btn-info:hover { background-color: #138496; }
        .btn-warning { background-color: #ffc107; color: #212529 !important; font-weight: bold; } .btn-warning:hover { background-color: #e0a800; }
        .btn-ticket { background-color: #6c757d; margin-top: 8px; display: none; }
        .btn-ticket:hover { background-color: #5a6268; }
        .resultado, .box-info { margin-top: 15px; padding: 12px; border-radius: 4px; display: none; font-size: 14px; }
        .exito { background-color: #d4edda; color: #155724; border: 1px solid #c3e6cb; }
        .error { background-color: #f8d7da; color: #721c24; border: 1px solid #f5c6cb; }
        .box-info { background-color: #e2e3e5; color: #383d41; border: 1px solid #d6d8db; max-height: 160px; overflow-y: auto; }
        .seccion-pago { background: #e9ecef; padding: 12px; border-radius: 4px; margin-bottom: 15px; }
        #moduloCaja { display: none; }
        #moduloLogin { max-width: 350px; margin: 50px auto; padding: 20px; background: white; border-radius: 8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1); text-align: center; }
        footer { margin-top: 25px; font-size: 11px; color: #6c757d; text-align: center; border-top: 1px dashed #dee2e6; padding-top: 10px; }
    </style>
</head>
<body>

    <div id="moduloLogin">
        <h3>Acceso de Cajero</h3>
        <input type="password" id="inputPass" placeholder="Contraseña de Sesión (1234)" style="width: 80%; margin-bottom: 15px; text-align: center;"><br>
        <button class="action-btn btn-success" onclick="procesarLogin()">Ingresar al Sistema</button>
        <p id="errorLogin" style="color: red; font-size: 13px; display: none; margin-top: 10px;"></p>
    </div>

    <div class="container" id="moduloCaja">
        <h2>Caja Registradora Pro</h2>
        
        <div class="form-group">
            <input type="text" id="inputDesc" class="desc-input" placeholder="Artículo (ej. Sabritas)">
            <input type="number" id="inputPrecio" class="price-input" placeholder="Precio ($)" min="0.01" step="0.01">
            <select id="selectIva" class="iva-select">
                <option value="0.16">16%</option>
                <option value="0.08">8%</option>
                <option value="0.00">0%</option>
            </select>
            <button class="btn-add" onclick="agregarAlCarrito()">+</button>
        </div>

        <div class="lista-carrito" id="listaCarrito">
            <p style="color: #888; text-align: center; margin: 5px 0;">El carrito está vacío</p>
        </div>

        <div class="seccion-pago">
            <strong style="font-size:13px; display:block; margin-bottom:5px;">Método y Transacción:</strong>
            <div style="display:flex; gap:10px;">
                <select id="selectMetodo" onchange="alternarCampoEfectivo()" style="width:40%;">
                    <option value="Efectivo">Efectivo</option>
                    <option value="Tarjeta">Tarjeta Bancaria</option>
                </select>
                <input type="number" id="inputEfectivo" placeholder="¿Con cuánto paga? ($)" style="width:60%;" min="0">
            </div>
        </div>
        
        <div class="grid-botones">
            <button class="action-btn btn-success" onclick="enviarCompra()">Procesar Venta</button>
            <button class="action-btn btn-danger" onclick="limpiarCarrito()">Vaciar Carrito</button>
        </div>
        <div class="grid-botones">
            <button class="action-btn btn-info" onclick="consultarHistorial()">Ver Historial</button>
            <button class="action-btn btn-warning" onclick="consultarReporteIva()">Corte de IVA</button>
        </div>
        
        <div id="resultado" class="resultado"></div>
        <button id="btnDescargarTicket" class="action-btn btn-ticket" onclick="descargarTicket()">⬇️ Descargar Ticket (.txt)</button>
        <div id="infoBox" class="box-info"></div>
        
        <footer>
            D.R. © 2026 Juan Valentín García Espinoza<br>
            ID: GAEJ940310HSPRSN02 | Todos los derechos reservados.
        </footer>
    </div>

    <script>
        /**
         * @file index.html
         * @description Cliente Frontend alineado con los endpoints y variables de Flask de backend.
         * @copyright (c) 2026 Juan Valentín García Espinoza (ID: GAEJ940310HSPRSN02)
         */
        let carrito = [];
        let ultimoTicketProcesado = null;
        let tokenSesion = "";
        const URL_BASE = 'http://127.0.0.1:5000';

        async function procesarLogin() {
            const pass = document.getElementById('inputPass').value;
            const errorTxt = document.getElementById('errorLogin');
            try {
                const respuesta = await fetch(`${URL_BASE}/login`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ password: pass })
                });
                const datos = await respuesta.json();
                if (respuesta.ok) {
                    tokenSesion = datos.token;
                    document.getElementById('moduloLogin').style.display = 'none';
                    document.getElementById('moduloCaja').style.display = 'block';
                } else {
                    errorTxt.innerText = datos.mensaje;
                    errorTxt.style.display = 'block';
                }
            } catch (e) {
                errorTxt.innerText = "Servidor backend desconectado.";
                errorTxt.style.display = 'block';
            }
        }

        function alternarCampoEfectivo() {
            const metodo = document.getElementById('selectMetodo').value;
            const inputEfectivo = document.getElementById('inputEfectivo');
            if (metodo === 'Tarjeta') {
                inputEfectivo.value = '';
                inputEfectivo.disabled = true;
            } else {
                inputEfectivo.disabled = false;
            }
        }

        function agregarAlCarrito() {
            const inputPrecio = document.getElementById('inputPrecio');
            const selectIva = document.getElementById('selectIva');
            const inputDesc = document.getElementById('inputDesc');
            
            const precio = parseFloat(inputPrecio.value);
            const tasaIva = parseFloat(selectIva.value);
            const desc = inputDesc.value.trim() || "Artículo genérico";

            if (isNaN(precio) || precio <= 0) {
                alert('Ingrese un precio válido.');
                return;
            }

            carrito.push({ descripcion: desc, precio: precio, tasa_iva: tasaIva });
            inputPrecio.value = '';
            inputDesc.value = '';
            actualizarVistaCarrito();
        }

        function limpiarCarrito() {
            if (carrito.length === 0) return;
            if (confirm('¿Vaciar los artículos del carrito actual?')) {
                carrito = [];
                actualizarVistaCarrito();
                ocultarContenedoresOpcionales();
            }
        }

        function actualizarVistaCarrito() {
            const contenedor = document.getElementById('listaCarrito');
            if (carrito.length === 0) {
                contenedor.innerHTML = '<p style="color: #888; text-align: center; margin: 5px 0;">El carrito está vacío</p>';
                return;
            }
            contenedor.innerHTML = carrito.map((item) => `
                <div class="item-carrito">
                    <span><strong>${item.descripcion}</strong> - $${item.precio.toFixed(2)}</span>
                    <span class="text-muted">(IVA ${(item.tasa_iva * 100).toFixed(0)}%)</span>
                </div>
            `).join('');
        }

        function ocultarContenedoresOpcionales() {
            document.getElementById('btnDescargarTicket').style.display = 'none';
            document.getElementById('resultado').style.display = 'none';
            document.getElementById('infoBox').style.display = 'none';
        }

        async function enviarCompra() {
            const contenedorResultado = document.getElementById('resultado');
            const btnTicket = document.getElementById('btnDescargarTicket');
            const infoBox = document.getElementById('infoBox');
            
            if (carrito.length === 0) {
                alert('El carrito está vacío.');
                return;
            }

            const metodoPago = document.getElementById('selectMetodo').value;
            const montoRecibido = parseFloat(document.getElementById('inputEfectivo').value) || 0;

            if (metodoPago === 'Efectivo' && montoRecibido <= 0) {
                alert('Ingrese el monto en efectivo recibido.');
                return;
            }

            try {
                const respuesta = await fetch(`${URL_BASE}/procesar-compra`, {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                        'Authorization': tokenSesion
                    },
                    body: JSON.stringify({
                        articulos: carrito,
                        metodo_pago: metodoPago,
                        monto_recibido: montoRecibido
                    })
                });

                const datos = await respuesta.json();
                contenedorResultado.style.display = 'block';
                infoBox.style.display = 'none';

                if (respuesta.ok) {
                    contenedorResultado.className = 'resultado exito';
                    contenedorResultado.innerHTML = `
                        <strong>¡Venta Exitosa (Ticket #${datos.id})!</strong><br>
                        Subtotal: $${datos.subtotal.toFixed(2)}<br>
                        IVA Total: $${datos.iva.toFixed(2)}<br>
                        <strong>Total: $${datos.total.toFixed(2)}</strong>
                        <hr style="margin:5px 0; border:0; border-top:1px dashed #155724;">
                        Pago: $${datos.monto_recibido.toFixed(2)} (${datos.metodo_pago})<br>
                        <strong>Cambio: $${datos.cambio.toFixed(2)}</strong>
                    `;
                    
                    // Almacenamos temporalmente una copia de los artículos antes de limpiar el carro para el ticket físico
                    datos.articulos_procesados = [...carrito];
                    ultimoTicketProcesado = datos;
                    
                    btnTicket.style.display = 'block';
                    carrito = [];
                    actualizarVistaCarrito();
                    document.getElementById('inputEfectivo').value = '';
                } else {
                    contenedorResultado.className = 'resultado error';
                    contenedorResultado.innerHTML = `<strong>Error:</strong><br>${datos.mensaje}`;
                    btnTicket.style.display = 'none';
                }
            } catch (error) {
                contenedorResultado.style.display = 'block';
                contenedorResultado.className = 'resultado error';
                contenedorResultado.innerHTML = `<strong>Error de conexión con el servidor.</strong>`;
                btnTicket.style.display = 'none';
            }
        }

        async function consultarHistorial() {
            const infoBox = document.getElementById('infoBox');
            document.getElementById('resultado').style.display = 'none';
            try {
                const respuesta = await fetch(`${URL_BASE}/historial`, {
                    headers: { 'Authorization': tokenSesion }
                });
                const datos = await respuesta.json();
                infoBox.style.display = 'block';
                
                if (respuesta.ok) {
                    if (datos.length === 0) {
                        infoBox.innerHTML = '<strong>Historial:</strong> No hay ventas registradas.';
                        return;
                    }
                    let html = '<strong>Últimas 10 Ventas:</strong><br>';
                    datos.forEach(v => {
                        html += `• Ticket #${v.id} | Total: $${v.total.toFixed(2)} | ${v.metodo_pago} | <small>${v.fecha}</small><br>`;
                    });
                    infoBox.innerHTML = html;
                } else {
                    infoBox.innerHTML = `Error: ${datos.mensaje}`;
                }
            } catch (e) {
                infoBox.style.display = 'block';
                infoBox.innerHTML = 'Error al conectar con la base de datos.';
            }
        }

        async function consultarReporteIva() {
            const infoBox = document.getElementById('infoBox');
            document.getElementById('resultado').style.display = 'none';
            try {
                const respuesta = await fetch(`${URL_BASE}/reporte-iva`, {
                    headers: { 'Authorization': tokenSesion }
                });
                const datos = await respuesta.json();
                infoBox.style.display = 'block';
                
                if (respuesta.ok) {
                    infoBox.innerHTML = `
                        <strong>Corte del Día (Hoy):</strong><br>
                        Transacciones: ${datos.transacciones}<br>
                        Subtotal Neto: $${datos.subtotal_dia.toFixed(2)}<br>
                        IVA Acumulado: $${datos.iva_dia.toFixed(2)}<br>
                        <hr style="margin:5px 0; border:0; border-top:1px solid #383d41;">
                        <strong>Total Bruto: $${datos.total_dia.toFixed(2)}</strong>
                    `;
                } else {
                    infoBox.innerHTML = `Error: ${datos.mensaje}`;
                }
            } catch (e) {
                infoBox.style.display = 'block';
                infoBox.innerHTML = 'Error al generar el reporte de impuestos.';
            }
        }

        function descargarTicket() {
            if (!ultimoTicketProcesado) return;
            
            const t = ultimoTicketProcesado;
            let contenido = `========================================\n`;
            contenido += `         TICKET DE COMPRA VENTA         \n`;
            contenido += `               TICKET #${t.id}          \n`;
            contenido += `========================================\n`;
            if(t.articulos_procesados) {
                t.articulos_procesados.forEach(art => {
                    contenido += `${art.descripcion.padEnd(22)} $${art.precio.toFixed(2).padStart(8)} (IVA ${(art.tasa_iva*100)}%)\n`;
                });
            }
            contenido += `----------------------------------------\n`;
            contenido += `Subtotal:                     $${t.subtotal.toFixed(2).padStart(8)}\n`;
            contenido += `IVA Recaudado:                $${t.iva.toFixed(2).padStart(8)}\n`;
            contenido += `TOTAL:                        $${t.total.toFixed(2).padStart(8)}\n`;
            contenido += `----------------------------------------\n`;
            contenido += `Método de Pago:               ${t.metodo_pago}\n`;
            contenido += `Monto Recibido:               $${t.monto_recibido.toFixed(2).padStart(8)}\n`;
            contenido += `Cambio Entregado:             $${t.cambio.toFixed(2).padStart(8)}\n`;
            contenido += `========================================\n`;
            contenido += `¡Gracias por su preferencia!\n`;
            contenido += `Cajero ID: GAEJ940310HSPRSN02\n`;

            const blob = new Blob([contenido], { type: 'text/plain;charset=utf-8' });
            const enlace = document.createElement('a');
            enlace.href = URL.createObjectURL(blob);
            enlace.download = `ticket_${t.id}.txt`;
            enlace.click();
            URL.revokeObjectURL(enlace.href);
        }
    </script>
</body>
</html>
