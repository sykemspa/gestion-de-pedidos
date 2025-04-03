<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gestión de Pedidos</title>
    <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-firestore-compat.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
            background-color: #f5f5f5;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        .container {
            width: 90%;
            max-width: 600px;
            padding: 20px;
            background: white;
            border-radius: 10px;
            box-shadow: 2px 2px 10px rgba(0,0,0,0.3);
            margin-bottom: 20px;
        }
        input, button {
            margin: 10px 0;
            padding: 10px;
            width: 100%;
        }
        button {
            background-color: #28a745;
            color: white;
            border: none;
            cursor: pointer;
        }
        button:hover {
            background-color: #218838;
        }
        .calendar {
            width: 100%;
            max-width: 1200px;
            padding: 20px;
            background: white;
            border-radius: 10px;
            box-shadow: 2px 2px 10px rgba(0,0,0,0.3);
            display: flex;
            flex-wrap: wrap;
        }
        .calendar-day {
            flex: 1;
            min-width: 120px;
            padding: 10px;
            margin: 5px;
            background: #ddd;
            text-align: center;
            border-radius: 5px;
        }
        .calendar-day.pendiente {
            background-color: #ffcc00;
        }
        .calendar-day.entregado {
            background-color: #28a745;
            color: white;
        }
        .calendar-day.sin-pedido {
            background-color: #f5f5f5;
        }
    </style>
</head>
<body>
    <div class="container">
        <h2>Gestión de Pedidos</h2>
        <h3>Nuevo Pedido</h3>
        <input type="text" id="cliente" placeholder="Nombre del Cliente">
        <input type="date" id="fechaPedido" readonly>
        <input type="text" id="descripcion" placeholder="Descripción del Pedido">
        <input type="number" id="montoTotal" placeholder="Monto Total" oninput="calcularSaldo()">
        <input type="number" id="abono" placeholder="Abono" oninput="calcularSaldo()">
        <input type="number" id="saldo" placeholder="Saldo" readonly>
        <input type="date" id="diaEntrega">
        <button onclick="agregarPedido()">Agregar Pedido</button>
        <h3>Pedidos Registrados</h3>
        <div id="listaPedidos"></div>
    </div>
    <div class="calendar" id="calendario"></div>
    <script>
        const firebaseConfig = {
            apiKey: "TU_API_KEY",
            authDomain: "TU_AUTH_DOMAIN",
            projectId: "TU_PROJECT_ID",
            storageBucket: "TU_STORAGE_BUCKET",
            messagingSenderId: "TU_MESSAGING_SENDER_ID",
            appId: "TU_APP_ID"
        };
        firebase.initializeApp(firebaseConfig);
        const db = firebase.firestore();

        // Inicializar fecha de pedido al día actual
        document.getElementById("fechaPedido").value = new Date().toISOString().split("T")[0];

        function calcularSaldo() {
            let montoTotal = parseFloat(document.getElementById("montoTotal").value) || 0;
            let abono = parseFloat(document.getElementById("abono").value) || 0;
            document.getElementById("saldo").value = montoTotal - abono;
        }

        function agregarPedido() {
            let cliente = document.getElementById("cliente").value;
            let descripcion = document.getElementById("descripcion").value;
            let montoTotal = parseFloat(document.getElementById("montoTotal").value) || 0;
            let abono = parseFloat(document.getElementById("abono").value) || 0;
            let saldo = montoTotal - abono;
            let diaEntrega = document.getElementById("diaEntrega").value;

            // Validación de campos
            if (!cliente || !descripcion || !montoTotal || !diaEntrega) {
                alert("Por favor, completa todos los campos.");
                return;
            }

            let pedido = {
                cliente: cliente,
                fechaPedido: new Date().toISOString().split("T")[0],
                descripcion: descripcion,
                montoTotal: montoTotal,
                abono: abono,
                saldo: saldo,
                diaEntrega: diaEntrega,
                estado: saldo > 0 ? "Pendiente" : "Entregado"
            };

            // Guardar el pedido en Firestore
            db.collection("pedidos").add(pedido).then(() => {
                actualizarCalendario();
                // Limpiar los campos después de agregar el pedido
                limpiarCampos();
            });
        }

        function limpiarCampos() {
            document.getElementById("cliente").value = "";
            document.getElementById("descripcion").value = "";
            document.getElementById("montoTotal").value = "";
            document.getElementById("abono").value = "";
            document.getElementById("saldo").value = "";
            document.getElementById("diaEntrega").value = "";
        }

        function actualizarCalendario() {
            db.collection("pedidos").onSnapshot((querySnapshot) => {
                let calendario = document.getElementById("calendario");
                calendario.innerHTML = "";
                let fechas = {};

                querySnapshot.forEach((doc) => {
                    let pedido = doc.data();
                    let fecha = pedido.diaEntrega;
                    if (!fechas[fecha]) {
                        fechas[fecha] = [];
                    }
                    fechas[fecha].push({
                        cliente: pedido.cliente,
                        descripcion: pedido.descripcion,
                        estado: pedido.estado
                    });
                });

                let hoy = new Date();
                for (let i = 0; i < 14; i++) {
                    let dia = new Date(hoy);
                    dia.setDate(hoy.getDate() + i);
                    let fechaStr = dia.toISOString().split("T")[0];
                    let div = document.createElement("div");
                    div.className = "calendar-day";
                    let estado = fechas[fechaStr] ? fechas[fechaStr][0].estado : "Sin pedido";
                    div.classList.add(estado === "Pendiente" ? "pendiente" : estado === "Entregado" ? "entregado" : "sin-pedido");
                    div.innerHTML = `<strong>${fechaStr}</strong><br>${fechas[fechaStr] ? fechas[fechaStr].map(p => `${p.cliente}: ${p.descripcion}`).join("<br>") : "Sin pedidos"}`;
                    calendario.appendChild(div);
                }
            });
        }

        // Cargar los pedidos al iniciar la página
        actualizarCalendario();
    </script>
</body>
</html>
