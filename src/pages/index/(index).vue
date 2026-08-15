<template>
  <div class="page-container">
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
  </div>
</template>

<script setup>
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
  if (carregando.value) return;

  if (!SERVICE_ID || !TEMPLATE_ID || !PUBLIC_KEY) {
    console.error("❌ Variáveis de ambiente não configuradas!");
    $q.notify({
      type: "negative",
      message: "Erro de configuração do servidor de e-mail.",
      position: "top",
      timeout: 5000,
    });
    return;
  }

  carregando.value = true;

  try {
    const templateParams = {
      from_name: nome.value,
      from_email: email.value,
      telefone: telefone.value,
      subject: assunto.value,
      message: mensagem.value,
    };

    console.log("📨 Enviando e-mail:", templateParams);

    const response = await emailjs.send(
      SERVICE_ID,
      TEMPLATE_ID,
      templateParams,
      {
        publicKey: PUBLIC_KEY,
      }
    );

    console.log("✅ E-mail enviado com sucesso!", response);

    $q.notify({
      type: "positive",
      message: "Mensagem enviada com sucesso! 📨",
      position: "top",
      timeout: 4000,
      icon: "check_circle",
    });

    limparFormulario();
  } catch (error) {
    console.error("❌ Erro detalhado ao enviar e-mail:", error);

    let errorMessage = "Erro ao enviar mensagem. Tente novamente!";

    if (error.text) {
      errorMessage = error.text;
    } else if (error.message) {
      errorMessage = error.message;
    }

    $q.notify({
      type: "negative",
      message: errorMessage,
      position: "top",
      timeout: 5000,
      icon: "error",
    });
  } finally {
    carregando.value = false;
  }
};
</script>

<style scoped>
.page-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem 1rem;
}

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

@media (max-width: 600px) {
  .form-header h1 {
    font-size: 1.5rem;
  }

  .form-header p {
    font-size: 1rem;
  }
}
</style>
