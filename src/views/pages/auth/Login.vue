<script setup>
// Importações do Sakai (existentes)
import FloatingConfigurator from '@/components/FloatingConfigurator.vue';

// --- NOSSAS NOVAS IMPORTAÇÕES ---
import { ref } from 'vue';
import { useRouter } from 'vue-router';
import axios from 'axios';

// Componentes PrimeVue que vamos usar (Sakai pode importá-los globalmente, mas é bom ser explícito)
import InputText from 'primevue/inputtext';
import Password from 'primevue/password';
import Checkbox from 'primevue/checkbox';
import Button from 'primevue/button';
import Message from 'primevue/message';
// ---------------------------------

// --- NOSSAS VARIÁVEIS DE ESTADO ---
// A API de token do Django espera 'username', não 'email'
const username = ref(''); // MUDANÇA: de 'email' para 'username'
const password = ref('');
const checked = ref(false); // (Para o "Remember me", podemos ver a lógica depois)

const router = useRouter(); // Para redirecionar após o login
const loading = ref(false); // Para o <Button>
const error = ref(null);   // Para a <Message>
// ---------------------------------

// --- NOSSA FUNÇÃO DE LOGIN ---
const handleLogin = async () => {
    loading.value = true;
    error.value = null;

    try {
        // 1. Chame a API de token que criamos no Django
        const response = await axios.post('/backend/api/token/', {
            username: username.value,
            password: password.value
        });

        // 2. Salve os tokens no Local Storage para usar depois
        localStorage.setItem('accessToken', response.data.access);
        localStorage.setItem('refreshToken', response.data.refresh);

        // 3. Redirecione para o Dashboard (rota '/')
        router.push('/'); 

    } catch (err) {
        // 4. Trate o erro (usuário/senha errados darão 401)
        console.error("Erro no login:", err);
        error.value = 'Usuário ou senha inválidos.';
    } finally {
        loading.value = false;
    }
};
// -----------------------------
</script>

<template>
    <FloatingConfigurator />
    <div class="bg-surface-50 dark:bg-surface-950 flex items-center justify-center min-h-screen min-w-[100vw] overflow-hidden">
        <div class="flex flex-col items-center justify-center">
            <div style="border-radius: 56px; padding: 0.3rem; background: linear-gradient(180deg, var(--primary-color) 10%, rgba(33, 150, 243, 0) 30%)">
                <div class="w-full bg-surface-0 dark:bg-surface-900 py-20 px-8 sm:px-20" style="border-radius: 53px">
                    <div class="text-center mb-8">
                        <img class="mx-auto mb-5" src="/layout/images/logoBrasao.png" alt="Brasão da Prefeitura" style="height:60px" />
                        <div class="text-surface-900 dark:text-surface-0 text-3xl font-medium mb-4">Ouvidoria Ativa</div>
                        <span class="text-muted-color font-medium">Acesso ao Painel de Gestão</span>
                    </div>

                    <form @submit.prevent="handleLogin">
                        <label for="username" class="block text-surface-900 dark:text-surface-0 text-xl font-medium mb-2">Usuário</label>
                        <InputText id="username" type="text" placeholder="Usuário" class="w-full md:w-[30rem] mb-8" v-model="username" />

                        <label for="password" class="block text-surface-900 dark:text-surface-0 font-medium text-xl mb-2">Senha</label>
                        <Password id="password" v-model="password" placeholder="Senha" :toggleMask="true" class="mb-4" fluid :feedback="false"></Password>

                        <div class="flex items-center justify-between mt-2 mb-8 gap-8">
                            <div class="flex items-center">
                                <Checkbox v-model="checked" id="rememberme1" binary class="mr-2"></Checkbox>
                                <label for="rememberme1">Lembrar-me</label>
                            </div>
                            <span class="font-medium no-underline ml-2 text-right cursor-pointer text-primary">Esqueceu a Senha?</span>
                        </div>
                        <Button label="Entrar" class="w-full" as="router-link" to="/"></Button>
                    </form>
                </div>
            </div>
        </div>
    </div>
</template>

<style scoped>
.pi-eye {
    transform: scale(1.6);
    margin-right: 1rem;
}

.pi-eye-slash {
    transform: scale(1.6);
    margin-right: 1rem;
}
</style>
