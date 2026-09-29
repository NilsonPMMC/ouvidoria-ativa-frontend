<template>
    <div class="flex items-center justify-center min-h-screen min-w-screen overflow-hidden">
        <div class="flex flex-col items-center justify-center">
            
            <div class="card">

                <div v-if="loading" class="text-center">
                    <ProgressSpinner style="width: 50px; height: 50px" strokeWidth="8" />
                    <p class="mt-3">Carregando dados da pesquisa...</p>
                </div>

                <div v-else-if="error" class="text-center">
                    <i class="pi pi-times-circle text-red-500" style="font-size: 3rem;"></i>
                    <h3>Link Inválido</h3>
                    <p>{{ error }}</p>
                </div>

                <div v-else-if="enviado" class="text-center">
                    <i class="pi pi-check-circle text=green-500" style="font-size: 3rem"></i>
                    <h3>Obrigado por responder!</h3>
                    <p>Sua avaliação foi registrada com sucesso.</p>
                </div>

                <div v-else>
                    <div class="flex flex-col items-center">
                        <img class="mb-5" src="/layout/images/logoBrasao.png" alt="Brasão da Prefeitura" style="height:40px" />
                        <div class="text-900 text-3xl font-medium mb-3">Pesquisa de Satisfação</div>
                        <Chip class="text-600 font-medium mb-3">Protocolo: <strong>{{ pesquisa.atendimento?.protocolo || '...' }}</strong></Chip>
                    </div>
                    <div>
                        <ul class="list-none p-0 m-0">
                            <li class="flex align-items-center py-2 border-bottom-1 surface-border">
                                <i class="pi pi-bookmark-fill mr-2 text-primary"></i>
                                <span class="text-900 font-medium">Serviço:</span>
                                <span class="text-600 ml-auto">{{ pesquisa.atendimento?.categoria }}</span>
                            </li>
                            <li class="flex align-items-center py-2 border-bottom-1 surface-border">
                                <i class="pi pi-map-marker mr-2 text-primary"></i>
                                <span class="text-900 font-medium">Bairro:</span>
                                <span class="text-600 ml-auto">{{ pesquisa.atendimento?.bairro || 'N/I' }}</span>
                            </li>
                            <li class="flex align-items-center py-2">
                                <i class="pi pi-calendar mr-2 text-primary"></i>
                                <span class="text-900 font-medium">Data Conclusão:</span>
                                <span class="text-600 ml-auto">{{ formatarData(pesquisa.atendimento?.data_conclusao) }}</span>
                            </li>
                        </ul>
                    </div>
                    <Divider />
                    <form @submit.prevent="enviarPesquisa">
                        <div class="flex flex-col gap-4">
                            <div class="flex flex-col gap-2 mb-3">
                                <label class="text-lg">Como você avalia o atendimento recebido?</label>
                                <Rating v-model="nota" :stars="5" :cancel="false" class="mt-3 rating-grande justify-center" />
                                <small v-if="notaInvalida" class="p-error">Por favor, selecione uma nota.</small>
                            </div>
                
                            <div class="flex flex-col gap-2">
                                <label for="comentario">Deixe um comentário (opcional):</label>
                                <Textarea id="comentario" v-model="comentario" rows="4" />
                            </div>
                
                            <Button type="submit" label="Enviar Avaliação" class="w-full h-16 text-xl" :loading="submitting" />
                        </div>
                    </form>
                </div>
            </div>

        </div>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { useRoute } from 'vue-router';
import axios from 'axios'; 


import Card from 'primevue/card';
import Rating from 'primevue/rating';
import Textarea from 'primevue/textarea';
import Button from 'primevue/button';
import ProgressSpinner from 'primevue/progressspinner';
import Chip from 'primevue/chip';
import Divider from 'primevue/divider';

// Reatividade
const route = useRoute();
const token = ref(route.params.token);

const pesquisa = ref({});
const nota = ref(null);
const comentario = ref('');

const loading = ref(true);
const submitting = ref(false);
const enviado = ref(false);
const error = ref(null);
const notaInvalida = ref(false);

// URL da nossa API
const API_URL = `/backend/api/pesquisa/${token.value}/`;

const formatarData = (dataISO) => {
    if (!dataISO) return '';
    const [ano, mes, dia] = dataISO.split('-');
    return `${dia}/${mes}/${ano}`;
};

// Função: Buscar dados da pesquisa quando o componente carregar
onMounted(async () => {
    try {
        const response = await axios.get(API_URL);
        pesquisa.value = response.data;
        loading.value = false;
    } catch (err) {
        error.value = 'Este link de pesquisa é inválido ou já foi respondido.';
        loading.value = false;
    }
});

// Função: Enviar a avaliação
const enviarPesquisa = async () => {
    if (nota.value === null) {
        notaInvalida.value = true;
        return;
    }
    notaInvalida.value = false;
    submitting.value = true;
    
    try {
        await axios.patch(API_URL, {
            nota_geral: nota.value,
            comentario: comentario.value
        });
        enviado.value = true;
    } catch (err) {
        error.value = 'Houve um erro ao enviar sua resposta. Tente novamente.';
    } finally {
        submitting.value = false;
    }
};
</script>

<style scoped>
/* Estilos para centralizar o formulário limpo */
.min-h-screen { min-height: 100vh; }
.min-w-screen { min-width: 100vw; }
body {
    overflow-x: hidden;
}
.rating-grande {
    scale: 1.75;
    gap: 1rem !important;
}
</style>