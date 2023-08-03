<template>
  <main class="app">
    <h1 class="app-title">Budget App</h1>
    <section class="balance">
      <div class="balance__header">
        <p class="balance__title">Total balance:</p>
        <i
          v-show="!editingMode"
          @click="editBalance"
          title="Edit"
          class="fa-regular fa-pen-to-square balance__edit-icon"
        ></i>
      </div>
      <div v-show="editingMode" class="balance__info">
        <span>$</span>
        <input
          type="number"
          placeholder="Enter your current balance..."
          class="balance__input"
          v-model="balance"
          @blur="setBalance"
          @keyup.enter="setBalance"
          @input="limitNumericInputs"
          ref="balanceInputField"
        />
      </div>
      <div v-show="!editingMode" class="balance__info">
        <p class="balance__total">$ {{ balance }}</p>
      </div>
    </section>
    <AddTransactionForm @add-transaction="addTransaction" />
    <TransactionList
      v-show="transactions.length"
      :transactions="transactions"
      @delete-transaction="deleteTransaction"
    />
  </main>
</template>

<script setup>
import { ref, onMounted, watch, nextTick } from "vue";
import AddTransactionForm from "./components/AddTransactionForm.vue";
import TransactionList from "./components/TransactionsList.vue";

emits: ["delete-transaction"];

const balance = ref("");
const transactions = ref([]);
const editingMode = ref(false);
const balanceInputField = ref();

const setBalance = () => {
  if (balance.value !== "") {
    editingMode.value = false;
  }
};

const editBalance = async () => {
  editingMode.value = true;
  await nextTick();
  balanceInputField.value.focus();
};

function limitNumericInputs() {
  if (balance.value > 99999999) {
    balance.value = 99999999;
  }
  if (balance.value < 0) {
    balance.value = 0;
  }
}

const addTransaction = (transaction) => {
  transactions.value.push(transaction);
  if (transaction.type == "expense") {
    balance.value -= transaction.amount;
  } else {
    balance.value = +balance.value + transaction.amount;
  }
  editingMode.value = false;
};

const deleteTransaction = (transaction) => {
  transactions.value = transactions.value.filter(
    (item) => item.id !== transaction.value.id
  );
  if (transaction.value.type == "expense") {
    balance.value = +balance.value + transaction.value.amount;
  } else {
    balance.value -= transaction.value.amount;
  }
};

watch(balance, (newVal) => {
  localStorage.setItem("balance", newVal);
});

watch(
  transactions,
  (newVal) => {
    localStorage.setItem("transactions", JSON.stringify(newVal));
  },
  { deep: true }
);

onMounted(() => {
  balance.value = localStorage.getItem("balance") || "";
  editingMode.value = balance.value === "" ? true : false;
  transactions.value = JSON.parse(localStorage.getItem("transactions")) || [];
});
</script>

<style>
#app {
  position: absolute;
  top: 10%;
  padding: 40px;
  border-radius: 10px;
  color: var(--text-color-primary);
  background-color: var(--app-bg);
  box-shadow: var(--app-bg-shadow);
  width: 600px;

  @media (max-width: 990px) {
    padding: 20px;
    max-width: 400px;
  }

  @media (max-width: 480px) {
    max-width: 300px;
  }
}

.app-title {
  text-align: center;
  font-size: 30px;
}

.balance {
  background: var(--gradient-bg);
  color: var(--white);
  border-radius: 8px;
  margin-top: 22px;
  padding: 28px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  box-shadow: var(--shadow);
}

.balance__header {
  display: flex;
  justify-content: space-between;
}

.balance__title {
  font-size: 14px;
}

.balance__info {
  margin-top: 12px;
  display: flex;
  align-items: center;
}

.balance__info span {
  font-size: 14px;
}

.balance__input {
  margin-left: 12px;
  padding: 6px 8px;
  width: 100%;
  border-radius: 5px;
  color: var(--text-color-primary);
}

.balance__total {
  font-size: 24px;
  font-weight: 600;
  margin-top: 4px;
}

.balance__edit-icon {
  transition: 0.2s ease-in-out;
}

.balance__edit-icon:hover {
  transform: scale(1.2);
}
</style>
