<template>
  <div class="form-header">
    <h1>Formulário de contato</h1>
    <p>Para saber mais faça contato conosco</p>
  </div>

  <q-form
    class="form"
    @submit.prevent="validarMensagem"
    @reset="limparFormulario"
    greedy
  >
    <q-input
      label="Nome"
      v-model="nome"
      :rules="[val => !!val || 'Informe seu nome']"
      outlined
      dense
    />

    <q-input
      label="Email"
      v-model="email"
      type="email"
      :rules="[
        val => !!val || 'Informe seu e-mail',
        val => /.+@.+\..+/.test(val) || 'E-mail inválido'
      ]"
      outlined
      dense
    />

    <q-input
      label="Telefone"
      v-model="telefone"
      mask="(##) #####-####"
      unmasked-value
      :rules="[val => !!val || 'Informe seu telefone']"
      outlined
      dense
    />

    <q-input
      label="Assunto"
      v-model="assunto"
      :rules="[val => !!val || 'Informe o assunto']"
      outlined
      dense
    />

    <q-input
      label="Mensagem"
      type="textarea"
      v-model="mensagem"
      :rules="[val => !!val || 'Digite a mensagem']"
      outlined
      dense
      rows="4"
    />

    <div class="flex q-gutter-md q-mt-md">
      <q-btn
        label="Enviar"
        color="primary"
        type="submit"
        :loading="carregando"
        :disable="carregando"
      />
      <q-btn
        label="Cancelar"
        color="red"
        type="reset"
        :disable="carregando"
        flat
      />
    </div>
  </q-form>
</template>

<script setup>
import "../../css/formLogin.scss";
import { ref } from "vue";
import { useQuasar } from "quasar";
import axios from "axios";

// ============================================
// VARIÁVEIS REATIVAS DO FORMULÁRIO
// ============================================
const nome = ref("");
const email = ref("");
const telefone = ref("");
const assunto = ref("");
const mensagem = ref("");
const carregando = ref(false);

// ============================================
// INSTÂNCIA DO QUASAR PARA NOTIFICAÇÕES
// ============================================
const $q = useQuasar();

// ============================================
// VARIÁVEIS DE AMBIENTE
// ============================================
const SERVICE_ID = import.meta.env.VITE_EMAILJS_SERVICE_ID || "";
const TEMPLATE_ID = import.meta.env.VITE_EMAILJS_TEMPLATE_ID || "";
const PUBLIC_KEY = import.meta.env.VITE_EMAILJS_PUBLIC_KEY || "";


// ============================================
// FUNÇÃO PARA LIMPAR O FORMULÁRIO
// ============================================
const limparFormulario = () => {
  nome.value = "";
  email.value = "";
  telefone.value = "";
  assunto.value = "";
  mensagem.value = "";
};

// ============================================
// FUNÇÃO PARA ENVIAR O EMAIL COM AXIOS
// ============================================
const validarMensagem = async () => {
  // Evita múltiplos envios
  if (carregando.value) return;

  // Valida se as variáveis de ambiente estão configuradas
  if (!SERVICE_ID || !TEMPLATE_ID || !PUBLIC_KEY) {
    console.error("❌ Variáveis de ambiente do EmailJS não configuradas!");
    $q.notify({
      type: "negative",
      message: "Erro de configuração do servidor de e-mail.",
      position: "top",
      timeout: 5000,
      actions: [
        {
          label: "Detalhes",
          color: "white",
          handler: () => {
            console.log("Service ID:", SERVICE_ID);
            console.log("Template ID:", TEMPLATE_ID);
            console.log("Public Key:", PUBLIC_KEY);
          }
        }
      ]
    });
    return;
  }

  // Ativa o loading
  carregando.value = true;

  try {
    // Parâmetros do template do EmailJS via API REST
    const payload = {
      service_id: SERVICE_ID,
      template_id: TEMPLATE_ID,
      user_id: PUBLIC_KEY,
      template_params: {
        from_name: nome.value,
        from_email: email.value,
        telefone: telefone.value,
        subject: assunto.value,
        message: mensagem.value,
        to_email: "seu-email@exemplo.com" // Opcional: e-mail de destino
      }
    };

    console.log("📨 Enviando e-mail com os parâmetros:", payload);

    // Envia o e-mail usando AXIOS
    const response = await axios.post(
      "https://api.emailjs.com/api/v1.0/email/send",
      payload,
      {
        headers: {
          "Content-Type": "application/json",
        },
        timeout: 15000, // 15 segundos de timeout
      }
    );

    console.log("✅ E-mail enviado com sucesso!", response.data);

    // Notificação de sucesso
    $q.notify({
      type: "positive",
      message: "Mensagem enviada com sucesso! 📨",
      position: "top",
      timeout: 4000,
      icon: "check_circle",
    });

    // Limpa o formulário após o envio bem-sucedido
    limparFormulario();

  } catch (error) {
    console.error("❌ Erro detalhado ao enviar e-mail:", error);

    // Mensagem de erro mais específica
    let errorMessage = "Erro ao enviar mensagem. Tente novamente!";
    let errorDetails = "";

    if (error.response) {
      // O servidor respondeu com status diferente de 2xx
      console.error("📦 Dados do erro:", error.response.data);
      console.error("📊 Status:", error.response.status);
      console.error("📋 Headers:", error.response.headers);

      errorDetails = error.response.data?.message || error.response.statusText;
      errorMessage = `Erro ${error.response.status}: ${errorDetails}`;

      // Tratamento específico para erros comuns
      if (error.response.status === 400) {
        errorMessage = "❌ Requisição inválida. Verifique os dados do formulário.";
      } else if (error.response.status === 401 || error.response.status === 403) {
        errorMessage = "🔒 Erro de autenticação. Verifique sua Public Key.";
      } else if (error.response.status === 404) {
        errorMessage = "❌ Service ID ou Template ID não encontrado.";
      } else if (error.response.status === 429) {
        errorMessage = "⏳ Muitas requisições. Aguarde um momento e tente novamente.";
      }

    } else if (error.request) {
      // A requisição foi feita mas não houve resposta
      console.error("📡 Sem resposta do servidor:", error.request);
      errorMessage = "⏰ Servidor não respondeu. Verifique sua conexão com a internet.";

    } else if (error.code === "ECONNABORTED") {
      errorMessage = "⏱️ Tempo limite excedido. O servidor demorou para responder.";

    } else {
      // Algo aconteceu na configuração da requisição
      console.error("⚙️ Erro na configuração:", error.message);
      errorMessage = `⚠️ Erro: ${error.message}`;
    }

    // Notificação de erro
    $q.notify({
      type: "negative",
      message: errorMessage,
      position: "top",
      timeout: 6000,
      icon: "error",
      actions: [
        {
          label: "Detalhes",
          color: "white",
          handler: () => {
            console.log("🔍 Detalhes do erro completo:", error);
            if (error.response) {
              console.log("📦 Dados:", error.response.data);
            }
          }
        }
      ]
    });

  } finally {
    // Desativa o loading
    carregando.value = false;
  }
};
</script>

<style scoped>
.form-header {
  margin-bottom: 2rem;
  text-align: center;
}

.form-header h1 {
  font-size: 2rem;
  color: #1a1a1a;
  margin-bottom: 0.5rem;
}

.form-header p {
  color: #666;
  font-size: 1.1rem;
}

.form {
  max-width: 600px;
  margin: 0 auto;
}

/* Estilos para melhorar a experiência mobile */
@media (max-width: 600px) {
  .form {
    max-width: 100%;
    padding: 0 1rem;
  }

  .form-header h1 {
    font-size: 1.5rem;
  }

  .form-header p {
    font-size: 1rem;
  }
}
</style>
