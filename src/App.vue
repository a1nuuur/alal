<template>
  <div id="app" class="container">
    <h1>Авто Закуп</h1>

    <button @click="addDetail">➕ Добавить деталь</button>
    <button @click="exportToExcel">📤 Экспорт в Excel</button>

    <div v-if="details.length" class="details-grid">
      <div v-for="(detail, index) in details" :key="detail.id" class="item">
        <input v-model="detail.title" placeholder="Название детали" />
        <input v-model.number="detail.price" type="number" placeholder="Цена" readonly />

        <button @click="addSubDetail(detail)">➕ Поддеталь</button>
        <button @click="removeDetail(index)" class="remove-btn">❌ Удалить</button>

        <SubDetailList
          :subDetails="detail.subDetails"
          @add-sub-detail="addSubDetail"
          @remove-sub-detail="removeSubDetail"
          @update-price="updateParentPrice(detail)"
        />
      </div>
    </div>
  </div>
</template>

<script>
import * as XLSX from "xlsx";
import SubDetailList from "./components/SubDetailList.vue";

export default {
  components: {
    SubDetailList
  },
  data() {
    return {
      details: []
    };
  },
  methods: {
    addDetail() {
      this.details.push({
        id: Date.now(),
        title: `Деталь ${this.details.length + 1}`,
        price: 0,
        subDetails: []
      });
    },
    addSubDetail(parent) {
      if (!parent.subDetails) {
        parent.subDetails = [];
      }
      parent.subDetails.push({
        id: Date.now(),
        name: `Поддеталь ${parent.subDetails.length + 1}`,
        price: 0,
        quantity: 1,
        subDetails: []
      });
      this.updateParentPrice(parent);
    },
    removeSubDetail(parentArray, subIndex) {
      parentArray.splice(subIndex, 1);
      this.recalculateAllPrices();
    },
    removeDetail(index) {
      this.details.splice(index, 1);
    },
    updateParentPrice(parent) {
      parent.price = this.calculateTotalPrice(parent.subDetails);
    },
    calculateTotalPrice(subDetails) {
      return subDetails.reduce((sum, sub) => {
        return sum + (sub.price * sub.quantity) + this.calculateTotalPrice(sub.subDetails);
      }, 0);
    },
    recalculateAllPrices() {
      this.details.forEach(detail => this.updateParentPrice(detail));
    },
    exportToExcel() {
      const data = [];

      function flattenDetails(details, parent = "") {
        details.forEach(detail => {
          const row = {
            "Название": detail.name || detail.title,
            "Цена": detail.price || "",
            "Количество": detail.quantity || "",
            "Родитель": parent
          };
          data.push(row);
          if (detail.subDetails && detail.subDetails.length > 0) {
            flattenDetails(detail.subDetails, detail.name || detail.title);
          }
        });
      }

      flattenDetails(this.details);

      const ws = XLSX.utils.json_to_sheet(data);
      const wb = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(wb, ws, "Автозапчасти");
      XLSX.writeFile(wb, "Автозапчасти.xlsx");
    }
  }
};
</script>

<style>
.container {
  max-width: 100%;
  margin: auto;
}
.details-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
}
.item {
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  background-color: #f9f9f9;
  text-align: center;
}
.remove-btn {
  background-color: #e74c3c;
  color: white;
}
</style>
