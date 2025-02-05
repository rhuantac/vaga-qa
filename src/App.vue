<script setup lang="ts">
import { ref, onMounted } from 'vue'
import BankVault from './components/BankVault.vue'

interface Account {
  id: number
  name: string
  accountNumber: string
  balance: number
}

interface Transaction {
  id: number
  type: 'deposito' | 'saque' | 'transferencia'
  amount: number
  date: string
  description: string
}

const accounts: Account[] = [
  { id: 1, name: 'João Silva', accountNumber: '1234-5', balance: 1000 },
  { id: 2, name: 'Maria Santos', accountNumber: '5678-9', balance: 5000 },
  { id: 3, name: 'Pedro Oliveira', accountNumber: '9012-3', balance: 100 }
]

const selectedAccount = ref<Account | null>(null)
const transactions = ref<Transaction[]>([])
const amount = ref('')
const description = ref('')
const destinatario = ref('')
const showModal = ref(false)
const dontShowAgain = ref(false)
const showVault = ref(false)


const revealVaultButton = () => {
  showVault.value = true
}

onMounted(() => {
  const shouldShow = localStorage.getItem('showWelcomeModal') !== 'false'
  if (shouldShow) {
    showModal.value = true
  }
})

const openModal = () => {
  showModal.value = true
  dontShowAgain.value = false // Reset a opção quando abrir manualmente
}

const closeModal = () => {
  if (dontShowAgain.value) {
    localStorage.setItem('showWelcomeModal', 'false')
  }
  showModal.value = false
}

const selectAccount = (account: Account) => {
  selectedAccount.value = account
}

const logout = () => {
  selectedAccount.value = null
  transactions.value = []
  amount.value = ''
  description.value = ''
  destinatario.value = ''
}

const checkBalanceForVault = () => {
  if (selectedAccount.value && (selectedAccount.value.balance > 1000000 || selectedAccount.value.balance < -1000000)) {
    showVault.value = true
  }
}

// Bug intencional: não verifica saldo negativo em transferências
const transfer = () => {
  if (!amount.value || !destinatario.value || !selectedAccount.value) return
  
  const valor = parseFloat(amount.value)
  selectedAccount.value.balance -= valor
  
  transactions.value.push({
    id: Date.now(),
    type: 'transferencia',
    amount: valor,
    date: new Date().toLocaleDateString(),
    description: `Transferência para ${destinatario.value}`
  })
  
  amount.value = ''
  destinatario.value = ''
  checkBalanceForVault()
}

// Bug intencional: permite saques com valor maior que o saldo
const withdraw = () => {
  if (!amount.value || !selectedAccount.value) return
  
  const valor = parseFloat(amount.value)
  selectedAccount.value.balance -= valor
  
  transactions.value.push({
    id: Date.now(),
    type: 'saque',
    amount: valor,
    date: new Date().toLocaleDateString(),
    description: 'Saque'
  })
  
  amount.value = ''
  checkBalanceForVault()
}

const deposit = () => {
  if (!amount.value || !selectedAccount.value) return
  
  const valor = parseFloat(amount.value)
  // Bug intencional: às vezes adiciona valor errado
  selectedAccount.value.balance += Math.random() > 0.8 ? valor * 1.1 : valor
  
  transactions.value.push({
    id: Date.now(),
    type: 'deposito',
    amount: valor,
    date: new Date().toLocaleDateString(),
    description: 'Depósito'
  })
  
  amount.value = ''
  checkBalanceForVault()
}
</script>

<template>
  <div class="container mx-auto p-4">
    <!-- Botão de Ajuda -->
    <button
      @click="openModal"
      class="fixed top-4 right-4 bg-blue-500 text-white p-2 rounded-full w-10 h-10 flex items-center justify-center hover:bg-blue-600 shadow-lg z-40"
      title="Abrir informações"
    >
      ?
    </button>

    <!-- Modal de Boas-vindas -->
    <div
      v-if="showModal"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50"
    >
      <div class="bg-white rounded-lg p-8 max-w-md w-full mx-4">
        <h2 class="text-2xl font-bold mb-4">Bem-vindo ao Banco Teste!</h2>
        <p class="mb-4">
          Este site simula um app de banco com algumas funcionalidades básicas.
          Porém ele contém alguns bugs espalhados de forma intencional.
        </p>
        <p class="mb-6">
          Seu desafio é encontrar o máximo de bugs possível e enviar para o nosso e-mail uma lista dos que você identificou, junto com seu currículo.           
        </p>
        <p class="mb-6">
          Além disso, gostaríamos que você também fizesse sugestões de melhoria de usabilidade, que podem ser incluídas no mesmo e-mail.
        </p>
        <p class="mb-6 font-semibold">Boa sorte! :)</p>
        <div class="flex items-center mb-4">
          <input
            type="checkbox"
            id="dontShowAgain"
            v-model="dontShowAgain"
            class="mr-2"
          />
          <label for="dontShowAgain">Não mostrar novamente</label>
        </div>
        <button
          @click="closeModal"
          class="w-full bg-blue-500 text-white py-2 rounded hover:bg-blue-600"
        >
          Entendi
        </button>
      </div>
    </div>

    <div class="max-w-4xl mx-auto">
      <!-- Tela de Seleção de Conta -->
      <template v-if="!selectedAccount">
        <h1 class="text-3xl font-bold mb-8 text-center">Banco Teste</h1>
        <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
          <div
            v-for="account in accounts"
            :key="account.id"
            @click="selectAccount(account)"
            class="bg-white shadow-lg rounded-lg p-6 cursor-pointer hover:shadow-xl transition-shadow"
          >
            <h3 class="text-xl font-semibold mb-2">{{ account.name }}</h3>
            <p class="text-gray-600 mb-2">Conta: {{ account.accountNumber }}</p>
            <p class="text-lg font-semibold text-green-600">
              R$ {{ account.balance.toFixed(2) }}
            </p>
          </div>
        </div>
      </template>

      <!-- Sistema Bancário -->
      <template v-else>
        <div class="flex justify-between items-center mb-8">
          <h1 class="text-3xl font-bold">Banco Teste</h1>
          <div>
            <p class="text-gray-600 mb-1">{{ selectedAccount.name }}</p>
            <p class="text-sm text-gray-500">Conta: {{ selectedAccount.accountNumber }}</p>
            <button
              @click="logout"
              class="text-red-500 hover:text-red-700 text-sm mt-2"
            >
              Sair da conta
            </button>
          </div>
        </div>
        
        <div class="bg-white shadow-lg rounded-lg p-6 mb-8">
          <h2 class="text-2xl font-semibold mb-4">Saldo Atual</h2>
          <p class="text-4xl font-bold text-green-600">
            R$ {{ selectedAccount.balance.toFixed(2) }}
          </p>
        </div>
        
        <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8">
          <div class="bg-white shadow-lg rounded-lg p-6">
            <h3 class="text-xl font-semibold mb-4">Depósito</h3>
            <input
              v-model="amount"
              type="number"
              placeholder="Valor"
              class="w-full p-2 border rounded mb-4"
            />
            <button
              @click="deposit"
              class="w-full bg-green-500 text-white py-2 rounded hover:bg-green-600"
            >
              Depositar
            </button>
          </div>
          
          <div class="bg-white shadow-lg rounded-lg p-6">
            <h3 class="text-xl font-semibold mb-4">Saque</h3>
            <input
              v-model="amount"
              type="number"
              placeholder="Valor"
              class="w-full p-2 border rounded mb-4"
            />
            <button
              @click="withdraw"
              class="w-full bg-red-500 text-white py-2 rounded hover:bg-red-600"
            >
              Sacar
            </button>
          </div>
          
          <div class="bg-white shadow-lg rounded-lg p-6">
            <h3 class="text-xl font-semibold mb-4">Transferência</h3>
            <input
              v-model="destinatario"
              type="text"
              placeholder="Destinatário"
              class="w-full p-2 border rounded mb-4"
            />
            <input
              v-model="amount"
              type="number"
              placeholder="Valor"
              class="w-full p-2 border rounded mb-4"
            />
            <button
              @click="transfer"
              class="w-full bg-blue-500 text-white py-2 rounded hover:bg-blue-600"
            >
              Transferir
            </button>
          </div>
        </div>
        
        <div class="bg-white shadow-lg rounded-lg p-6">
          <h3 class="text-xl font-semibold mb-4">Histórico de Transações</h3>
          <div class="space-y-4">
            <div
              v-for="transaction in transactions"
              :key="transaction.id"
              class="border-b pb-4"
            >
              <div class="flex justify-between items-center">
                <div>
                  <p class="font-semibold">
                    {{ transaction.type.charAt(0).toUpperCase() + transaction.type.slice(1) }}
                  </p>
                  <p class="text-sm text-gray-600">{{ transaction.description }}</p>
                  <p class="text-sm text-gray-500">{{ transaction.date }}</p>
                </div>
                <p
                  :class="{
                    'text-green-600': transaction.type === 'deposito',
                    'text-red-600': transaction.type === 'saque' || transaction.type === 'transferencia'
                  }"
                  class="font-semibold"
                >
                  {{ transaction.type === 'deposito' ? '+' : '-' }}R$ {{ transaction.amount.toFixed(2) }}
                </p>
              </div>
            </div>
          </div>
        </div>

        <!-- Botão oculto para acessar o cofre -->
        <div
          @click="revealVaultButton"
          class="fixed bottom-0 right-0 w-10 h-10 cursor-pointer"
          title="Clique para revelar o cofre"
        ></div>

        <!-- Componente do Cofre -->
        <BankVault v-if="showVault" />
      </template>
    </div>
  </div>
</template>

<style scoped>


.hidden-button {
  position: fixed;
  bottom: 10px;
  right: 10px;
  width: 20px;
  height: 20px;
  background-color: transparent;
  cursor: pointer;
}
</style>