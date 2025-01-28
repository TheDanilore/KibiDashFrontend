<template>
    <div class="login-container">
        <h2>Inicio de Sesión de Administrador</h2>
        <form @submit.prevent="handleLogin">
            <div class="form-group">
                <label for="email">Correo Electrónico</label>
                <input type="email" id="email" v-model="email" placeholder="Ingresa tu correo" required />
            </div>
            <div class="form-group">
                <label for="password">Contraseña</label>
                <input type="password" id="password" v-model="password" placeholder="Ingresa tu contraseña" required />
            </div>
            <button type="submit" class="login-button">Iniciar Sesión</button>
            <p v-if="error" class="error-message">{{ error }}</p>
        </form>
    </div>
</template>

<script>
import { ref } from 'vue';
import { useRouter } from 'vue-router';

export default {
    name: 'LoginAdmin',
    setup() {
        const email = ref('');
        const password = ref('');
        const error = ref(null);
        const router = useRouter();

        const handleLogin = async () => {
            error.value = null;
            try {
                //const response = await fetch('http://localhost:8000/api/login', {
                const response = await fetch('https://8nkxdpw3-8000.brs.devtunnels.ms/api/login', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                    },
                    body: JSON.stringify({ email: email.value, password: password.value }),
                });

                const data = await response.json();
                console.log(data); // Verifica la respuesta

                if (response.ok) {
                    console.log(data.token); // Verifica el token
                    localStorage.setItem('adminToken', data.token);
                    localStorage.setItem('userRole', data.role);

                    if (data.role === 'admin') {
                        router.push('/productos/admin');
                    } else {
                        router.push('/');
                    }
                } else {
                    error.value = data.message || 'Error en el inicio de sesión';
                }
            } catch (e) {
                error.value = 'Hubo un problema con la conexión. Inténtalo nuevamente.';
            }
        };



        return { email, password, error, handleLogin };
    },
};
</script>

<style scoped>
.login-container {
    max-width: 400px;
    margin: 6rem auto;
    padding: 2rem;
    border-radius: 8px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    background-color: #ffffff;
}

h2 {
    text-align: center;
    color: #333;
    margin-bottom: 1.5rem;
}

.form-group {
    margin-bottom: 1.2rem;
}

label {
    display: block;
    font-weight: bold;
    margin-bottom: 0.5rem;
    color: #555;
}

input[type="email"],
input[type="password"] {
    width: 100%;
    padding: 0.8rem;
    border-radius: 4px;
    border: 1px solid #ccc;
    font-size: 1rem;
    box-sizing: border-box;
}

.login-button {
    width: 100%;
    padding: 0.8rem;
    border: none;
    border-radius: 4px;
    background-color: #ff5c5c;
    color: #fff;
    font-weight: bold;
    font-size: 1rem;
    cursor: pointer;
    transition: background-color 0.3s ease;
}

.login-button:hover {
    background-color: #ff3a3a;
}

.error-message {
    color: red;
    margin-top: 1rem;
    text-align: center;
}
</style>
