<script setup lang="ts">
import { ref } from 'vue'

const secretCode = ref('')
const correctCode = '1234'
const accessGranted = ref(true)

const checkCode = () => {
  if (secretCode.value === correctCode) {
    accessGranted.value = true
  }
}
</script>

<template>
  <div v-if="accessGranted" class="vault">
    <h2 class="text-2xl font-bold mb-4">Cofre do Banco</h2>
    <p class="mb-4">Parabéns! Você encontrou o cofre secreto.</p>
    <p class="mb-4">A palavra-chave é: <strong>QA-SECRET</strong></p>
    <p>Envie-a no início do email para que saibamos que você a encontrou.</p>
  </div>
  <div v-else class="vault-access">
    <h2 class="text-2xl font-bold mb-4">Acesso ao Cofre</h2>
    <input
      v-model="secretCode"
      type="password"
      placeholder="Digite o código secreto"
      class="w-full p-2 border rounded mb-4"
    />
    <button
      @click="checkCode"
      class="w-full bg-blue-500 text-white py-2 rounded hover:bg-blue-600"
    >
      Acessar
    </button>
  </div>
</template>

<style scoped>
.vault, .vault-access {
  background: #fff;
  padding: 2rem;
  border-radius: 0.5rem;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}
</style>