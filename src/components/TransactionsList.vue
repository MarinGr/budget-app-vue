<template>
  <section class="transactions-list">
    <p class="list__title">Trasnsactions</p>
    <div class="list__container">
      <template
        class="transaction-card"
        v-for="transaction in sortedByDate"
        :key="transaction.createdAt"
      >
        <TransactionItem
          :transaction="transaction"
          @delete-transaction="deleteTransaction"
        />
      </template>
    </div>
  </section>
</template>

<script setup>
import { computed, toRefs } from "vue";
import TransactionItem from "./TransactionItem.vue";

const props = defineProps({
  transactions: Array,
});

const { transactions: transactions } = toRefs(props);

const emit = defineEmits(["delete-transaction"]);

emits: ["delete-transaction"];

const sortedByDate = computed(() => {
  return transactions.value.sort((a, b) => {
    return b.createdAt - a.createdAt;
  });
});

const deleteTransaction = (transaction) => {
  emit("delete-transaction", transaction);
};
</script>

<style scoped>
.transactions-list {
  margin-top: 24px;
}

.list__title {
  font-weight: 600;
}
.list__container {
  margin-top: 12px;
  max-height: 400px;
  overflow: auto;
}

.list__container::-webkit-scrollbar {
  width: 12px;
}

.list__container::-webkit-scrollbar-track {
  box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.1);
  border-radius: 8px;
}

.list__container::-webkit-scrollbar-thumb {
  background-color: #dadff5;
  border-radius: 8px;
}
</style>
