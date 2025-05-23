<template>
  <div class="transaction-list">
    <div class="table-responsive">
      <table class="table">
        <thead>
          <tr class="table-header">
            <th scope="col" class="text-secondary">날짜</th>
            <th scope="col" class="text-secondary">카테고리</th>
            <th scope="col" class="text-secondary">내용</th>
            <th scope="col" class="text-secondary">금액</th>
            <th scope="col" class="text-secondary">메모</th>
            <th scope="col" class="text-center text-secondary">작업</th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="displayedTransactions.length === 0">
            <td colspan="6" class="text-center py-4 text-muted">
              거래 내역이 없습니다.
            </td>
          </tr>
          <TransactionListItem
            v-for="(transaction, index) in displayedTransactions"
            :key="`${transaction.type}-${transaction.date}-${transaction.amount}`"
            :transaction="transaction"
            @edit-transaction="editTransaction"
            @delete-transaction="deleteTransaction"
          />
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { computed, onMounted, ref, watch } from "vue";
import TransactionListItem from "./TransactionListItem.vue";
import { useTransactionStore } from "@/stores/transaction-store";

import "bootstrap/dist/css/bootstrap.css";
import "bootstrap/dist/js/bootstrap.bundle.min.js";

// Props & Emits
const props = defineProps({
  currentPage: {
    type: Number,
    default: 1,
  },
  itemsPerPage: {
    type: Number,
    default: 10,
  },
  isCompactView: {
    type: Boolean,
    default: false,
  },
  compactItemCount: {
    type: Number,
    default: 3,
  },
});

const emit = defineEmits([
  "edit-transaction",
  "delete-transaction",
  "update:totalItems",
]);

const transactionStore = useTransactionStore();
const filteredTransactions_2 = computed(() => transactionStore.transactionInfo);
const filteredTransactions = ref([]);
onMounted(async () => {
  await transactionStore.fetchTransaction();
  filteredTransactions.value = [...transactionStore.transactionInfo];
});

console.log("slice:", filteredTransactions.value.slice());

const sortedFilteredTransactions = computed(() => {
  return [...filteredTransactions.value]
    .slice()
    .sort((a, b) => new Date(b.date) - new Date(a.date));
});

// 현재 보여줄 거래 항목
const displayedTransactions = computed(() => {
  if (props.isCompactView) {
    return sortedFilteredTransactions.value.slice(0, props.compactItemCount);
  }
  const start = (props.currentPage - 1) * props.itemsPerPage;
  const end = start + props.itemsPerPage;
  return sortedFilteredTransactions.value.slice(start, end);
});

const totalItems = computed(() => filteredTransactions.value.length);

watch(
  totalItems,
  (value) => {
    emit("update:totalItems", value);
  },
  { immediate: true }
);

// 거래 수정/삭제 emit
const editTransaction = (transactionData) => {
  emit("edit-transaction", transactionData);
};

const deleteTransaction = (transactionData) => {
  emit("delete-transaction", transactionData);
};

// 외부에서 필터링된 데이터 수신
const handleFilterTransactions = (newFilteredTransactions) => {
  filteredTransactions.value = newFilteredTransactions;
};
// 전체 거래 가져오기
const getAllTransactions = () => transactionStore.transactionInfo;

defineExpose({
  handleFilterTransactions,
});
</script>

<style scoped>
.transaction-list {
  margin-bottom: 1.5rem;
  width: 100%;
}
.text-primary {
  color: #4f48dc !important;
}
.text-secondary {
  color: #6c757d !important;
  background-color: #f5f5f5 !important;
}
.table {
  border-collapse: separate;
  border-spacing: 0;
  width: 100%;
}
.table-responsive {
  width: 100%;
  overflow-x: auto;
}
.table-header {
  border-top: 1px solid #dee2e6;
  border-bottom: 1px solid #dee2e6;
  background-color: #f8f9fa;
}
.table th {
  padding: 0.75rem;
  font-weight: 500;
  color: #6c757d;
}
.table td {
  padding: 0.75rem;
  vertical-align: middle;
  border-bottom: 1px solid #dee2e6;
}
.border-bottom {
  border-bottom: 1px solid #dee2e6;
}
tr:hover {
  background-color: rgba(0, 0, 0, 0.01);
}
tr.selected {
  background-color: rgba(13, 110, 253, 0.08);
}
</style>
