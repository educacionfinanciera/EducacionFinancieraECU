<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Plataforma de Pagos</title>
    <script src="https://js.stripe.com/v3/"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin: 20px;
            background-color: #f9f9f9;
        }
        .container {
            max-width: 600px;
            margin: auto;
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        button {
            padding: 10px 20px;
            font-size: 16px;
            margin: 10px;
            color: white;
            background: #007BFF;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
        button:hover {
            background: #0056b3;
        }
        input {
            margin: 10px;
            padding: 10px;
            font-size: 16px;
            width: calc(100% - 24px);
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Plataforma de Pagos</h1>
        
        <!-- Registro de Usuarios -->
        <h3>Registro de Usuario</h3>
        <input id="email" type="email" placeholder="Correo electrónico" />
        <input id="password" type="password" placeholder="Contraseña" />
        <button onclick="registerUser()">Registrar</button>

        <!-- Botón de Pago -->
        <h3>Realizar Pago</h3>
        <input id="amount" type="number" placeholder="Cantidad en USD" />
        <button onclick="makePayment()">Pagar</button>

        <!-- Historial de Transacciones -->
        <h3>Historial de Transacciones</h3>
        <div id="transactions"></div>

        <!-- Soporte al Cliente -->
        <h3>Soporte al Cliente</h3>
        <p>En caso de dudas, contacta a: <a href="mailto:soporte@ejemplo.com">soporte@ejemplo.com</a></p>
    </div>

    <script src="https://www.gstatic.com/firebasejs/9.21.0/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.21.0/firebase-auth.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.21.0/firebase-firestore.js"></script>
    <script>
        // Configuración de Firebase
        const firebaseConfig = {
            apiKey: "TU_API_KEY",
            authDomain: "TU_AUTH_DOMAIN",
            projectId: "TU_PROJECT_ID",
            storageBucket: "TU_STORAGE_BUCKET",
            messagingSenderId: "TU_MESSAGING_SENDER_ID",
            appId: "TU_APP_ID"
        };
        const app = firebase.initializeApp(firebaseConfig);
        const auth = firebase.auth();
        const db = firebase.firestore();

        // Registrar Usuario
        function registerUser() {
            const email = document.getElementById('email').value;
            const password = document.getElementById('password').value;
            auth.createUserWithEmailAndPassword(email, password)
                .then(() => alert("Usuario registrado con éxito"))
                .catch(err => alert(err.message));
        }

        // Realizar Pago
        async function makePayment() {
            const amount = document.getElementById('amount').value;
            if (!auth.currentUser) {
                alert("Inicia sesión para realizar un pago");
                return;
            }
            // Aquí iría la lógica para conectarse con Stripe o PayPal.
            // Simularemos el pago:
            const user = auth.currentUser;
            await db.collection("transactions").add({
                userId: user.uid,
                amount: amount,
                timestamp: new Date()
            });
            alert("Pago realizado con éxito");
            loadTransactions();
        }

        // Cargar Historial de Transacciones
        function loadTransactions() {
            const user = auth.currentUser;
            if (!user) {
                document.getElementById('transactions').innerHTML = "Inicia sesión para ver tu historial.";
                return;
            }
            db.collection("transactions")
                .where("userId", "==", user.uid)
                .orderBy("timestamp", "desc")
                .get()
                .then(snapshot => {
                    let html = "<ul>";
                    snapshot.forEach(doc => {
                        const data = doc.data();
                        html += <li>$${data.amount} - ${data.timestamp.toDate()}</li>;
                    });
                    html += "</ul>";
                    document.getElementById('transactions').innerHTML = html;
                });
        }

        // Escuchar cambios de autenticación
        auth.onAuthStateChanged(user => {
            if (user) {
                loadTransactions();
            } else {
                document.getElementById('transactions').innerHTML = "Inicia sesión para ver tu historial.";
            }
        });
    </script>
</body>
</html>
