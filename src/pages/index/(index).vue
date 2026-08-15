<template>
  <div class="form-header">
    <h1>Formulário de contato</h1>
    <p>Para saber mais faça contato conosco</p>
  </div>

  <q-form class="form" @submit="validarMensagem" greedy>
    <q-input
      label="Nome"
      v-model="nome"
      :rules="[(val) => !!val || 'Informe seu nome']"
    />
    <q-input
      label="Email"
      v-model="email"
      :rules="[
        (val) => !!val || 'Informe seu e-mail',
        (val) => /.+@.+\..+/.test(val) || 'E-mail inválido',
      ]"
    />
    <q-input
      label="Assunto"
      v-model="assunto"
      :rules="[(val) => !!val || 'Informe o assunto']"
    />
    <q-input
      label="Mensagem"
      type="textarea"
      class=""
      v-model="mensagem"
      :rules="[(val) => !!val || 'Digite a mensagem']"
    />

    <div class="flex q-gutter-md">
      <q-btn label="Enviar" color="primary" type="submit" />
      <q-btn label="Cancelar" color="red" type="reset" />
    </div>
  </q-form>
</template>

<script setup>
import "../../css/formLogin.scss";
import { ref } from "vue";
const nome = ref("");
const email = ref("");
const assunto = ref("");
const mensagem = ref("");

const validarMensagem = () => {
  console.log({
    Nome: nome.value,
    email: email.value,
    assunto: assunto.value,
    mensagem: mensagem.value,
  });

  nome.value = "";
  email.value = "";
  assunto.value = "";
  mensagem.value = "";
};
</script>
