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
    />

    <q-input
      label="Email"
      v-model="email"
      type="email"
      :rules="[
        val => !!val || 'Informe seu e-mail',
        val => /.+@.+\..+/.test(val) || 'E-mail inválido'
      ]"
    />

    <q-input
      label="Telefone"
      v-model="telefone"
      mask="(##) #####-####"
      unmasked-value
      :rules="[val => !!val || 'Informe seu telefone']"
    />

    <q-input
      label="Assunto"
      v-model="assunto"
      :rules="[val => !!val || 'Informe o assunto']"
    />

    <q-input
      label="Mensagem"
      type="textarea"
      v-model="mensagem"
      :rules="[val => !!val || 'Digite a mensagem']"
    />

    <div class="flex q-gutter-md q-mt-md">
      <q-btn
        label="Enviar"
        color="primary"
        type="submit"
        :loading="carregando"
      />
      <q-btn label="Cancelar" color="red" type="reset" :disable="carregando" />
    </div>
  </q-form>
</template>

<script setup>
import "../../css/formLogin.scss";
import { ref } from "vue";
import { useQuasar } from "quasar";
import emailjs from "@emailjs/browser";

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

// Debug - verifica se as variáveis estão carregando
console.log("🔍 Configuração do EmailJS:");
console.log("✅ Service ID:", SERVICE_ID);
console.log("✅ Template ID:", TEMPLATE_ID);
console.log("✅ Public Key:", PUBLIC_KEY);

// ============================================
// INICIALIZA O EMAILJS
// ============================================
if (PUBLIC_KEY) {
  try {
    emailjs.init(PUBLIC_KEY);
    console.log("✅ EmailJS inicializado com sucesso!");
  } catch (error) {
    console.error("❌ Erro ao inicializar EmailJS:", error);
  }
} else {
  console.warn("⚠️ Public Key não encontrada! Verifique seu arquivo .env");
}

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
// FUNÇÃO PARA ENVIAR O EMAIL
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
      timeout: 5000
    });
    return;
  }

  // Ativa o loading
  carregando.value = true;

  try {
    // Parâmetros do template do EmailJS
    const templateParams = {
      from_name: nome.value,
      from_email: email.value,
      telefone: telefone.value,
      subject: assunto.value,
      message: mensagem.value
    };

    console.log("📨 Enviando e-mail com os parâmetros:", templateParams);

    // Envia o e-mail usando o EmailJS
    const response = await emailjs.send(
      SERVICE_ID,
      TEMPLATE_ID,
      templateParams,
      {
        publicKey: PUBLIC_KEY
      }
    );

    console.log("✅ E-mail enviado com sucesso!", response);

    // Notificação de sucesso
    $q.notify({
      type: "positive",
      message: "Mensagem enviada com sucesso! 📨",
      position: "top",
      timeout: 4000
    });

    // Limpa o formulário após o envio bem-sucedido
    limparFormulario();
  } catch (error) {
    console.error("❌ Erro detalhado ao enviar e-mail:", error);

    // Mensagem de erro mais específica
    let errorMessage = "Erro ao enviar mensagem. Tente novamente!";

    if (error.text) {
      errorMessage = `Erro: ${error.text}`;
    } else if (error.message) {
      errorMessage = `Erro: ${error.message}`;
    }

    // Notificação de erro
    $q.notify({
      type: "negative",
      message: errorMessage,
      position: "top",
      timeout: 5000
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
}
</style>
