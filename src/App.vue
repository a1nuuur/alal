<template>
  <div id="app" class="container">
    <h1>Авто Закуп</h1>

    <div class="form">
      <button @click="openAddDetailModal">Добавить деталь</button>
      <button @click="exportToExcel">Экспортировать в Эксель</button>
    </div>

    <div v-if="details.length > 0" class="details-grid">
      <div v-for="(detail, index) in details" :key="index" class="item">
        <img :src="detail.image" alt="Добавить изображение" />
        <span>{{ detail.title }}</span>
        <span><strong>Цена:</strong> {{ calculateTotalPrice(index) }} тг.</span>
        <div>
          <button @click="addQuantity(index)">+</button>
          <span>{{ detail.quantity }}</span>
          <button @click="removeQuantity(index)" :disabled="detail.quantity <= 0">-</button>
        </div>
        <div>
          <h3>Детали</h3>
          <ul>
            <li v-for="(option, optionIndex) in detail.options" :key="optionIndex">
              {{ option.name }} - Кол-во: {{ option.quantity }} - Цена за единицу: {{ option.price }} тг.
              <button @click="addOptionQuantity(index, optionIndex)">+</button>
              <button @click="removeOptionQuantity(index, optionIndex)" :disabled="option.quantity <= 0">-</button>
              <button @click="removeOption(index, optionIndex)">Удалить</button>
            </li>
          </ul>
          <input type="text" v-model="detail.newOption.name" placeholder="Добавить деталь" />
          <input type="number" v-model.number="detail.newOption.price" placeholder="Цена" />
          <button @click="addOption(index)">Добавить</button>
        </div>
        <button @click="removeDetail(index)" class="remove-btn">Удалить</button>
      </div>
    </div>
    <div v-else>
      <p>No details added yet.</p>
    </div>

    <div v-if="isModalOpen" class="modal">
      <div class="modal-content">
        <span class="close-btn" @click="closeModal">&times;</span>
        <h2>Добавить деталь</h2>
        <input type="text" v-model="newDetail.title" placeholder="Название детали" />
        <input type="file" @change="handleFileChange" />
        <button @click="addDetail">Добавить</button>
      </div>
    </div>
  </div>
</template>

<script>
import * as XLSX from 'xlsx';

export default {
  data() {
    return {
      newDetail: {
        title: '',
        image: '',
        quantity: 0,
        options: [],
        newOption: { name: '', price: 0, quantity: 1 }
      },
      details: [
        {
          title: 'Кузов',
          image: 'https://png.pngtree.com/png-clipart/20231020/original/pngtree-3d-car-white-blank-template-auto-body-car-photo-png-image_13389021.png',
          quantity: 10,
          options: [
            { name: 'Стекла', quantity: 2, price: 5000 },
            { name: 'Двери', quantity: 1, price: 11000 }
          ],
          newOption: { name: '', price: 0, quantity: 1 }
        },
        {
          title: 'Двигатель',
          image: 'https://png.pngtree.com/png-clipart/20231020/original/pngtree-3d-car-white-blank-template-auto-body-car-photo-png-image_13389021.png',
          quantity: 5,
          options: [
            { name: 'Поршни', quantity: 3, price: 3000 },
            { name: 'Чугунные трубки', quantity: 1, price: 2000 }
          ],
          newOption: { name: '', price: 0, quantity: 1 }
        }
      ],
      isModalOpen: false
    };
  },
  methods: {
    handleFileChange(event) {
      const file = event.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = (e) => {
          this.newDetail.image = e.target.result;
        };
        reader.readAsDataURL(file);
      }
    },
    openAddDetailModal() {
      this.isModalOpen = true;
    },
    closeModal() {
      this.isModalOpen = false;
    },
    addDetail() {
      if (this.newDetail.title && this.newDetail.image) {
        this.details.push({ ...this.newDetail });
        this.newDetail.title = '';
        this.newDetail.image = '';
        this.newDetail.quantity = 0;
        this.newDetail.options = [];
        this.newDetail.newOption = { name: '', price: 0, quantity: 1 };
        this.closeModal();
      } else {
        alert('ошибка');
      }
    },
    removeDetail(index) {
      this.details.splice(index, 1);
    },
    addQuantity(index) {
      this.details[index].quantity++;
    },
    removeQuantity(index) {
      if (this.details[index].quantity > 0) {
        this.details[index].quantity--;
      }
    },
    addOption(index) {
      const newOption = this.details[index].newOption;
      if (newOption.name && newOption.price > 0 && newOption.quantity > 0) {
        this.details[index].options.push({ ...newOption });
        this.details[index].newOption = { name: '', price: 0, quantity: 1 };
      }
    },
    removeOption(detailIndex, optionIndex) {
      this.details[detailIndex].options.splice(optionIndex, 1);
    },
    addOptionQuantity(detailIndex, optionIndex) {
      this.details[detailIndex].options[optionIndex].quantity++;
    },
    removeOptionQuantity(detailIndex, optionIndex) {
      if (this.details[detailIndex].options[optionIndex].quantity > 0) {
        this.details[detailIndex].options[optionIndex].quantity--;
      }
    },
    calculateTotalPrice(index) {
      const detail = this.details[index];
      return detail.options.reduce(
          (total, option) => total + option.price * option.quantity,
          0
      );
    },
    exportToExcel() {
      const wsData = this.details.map((detail) => ({
        Title: detail.title,
        Quantity: detail.quantity,
        Price: this.calculateTotalPrice(this.details.indexOf(detail)), // Добавим цену
        Options: detail.options
            .map((option) => `${option.name} (x${option.quantity})`)
            .join(', '),
      }));

      const ws = XLSX.utils.json_to_sheet(wsData);
      const wb = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(wb, ws, 'Details');

      // Проверка на название файла
      try {
        XLSX.writeFile(wb, 'details.xlsx');
      } catch (error) {
        console.error('Ошибка экспорта в Excel:', error);
      }
    }

  }
    };
</script>

<style>
body {
  font-family: Arial, sans-serif;
  margin: 20px;
}
.container {
  max-width: 100%;
  margin: auto;
}

.form {
  margin-bottom: 20px;
}

.details-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
}

.item {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  background-color: #f9f9f9;
  text-align: center;
}

.item img {
  width: 100%;
  height: 200px;
  object-fit: cover;
  border-radius: 5px;
  margin-bottom: 10px;
}

.remove-btn {
  margin-top: 10px;
  background-color: #e74c3c;
  color: white;
  border: none;
  padding: 5px 10px;
  cursor: pointer;
  border-radius: 3px;
}

.remove-btn:hover {
  background-color: #c0392b;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 5px;
  width: 400px;
  position: relative;
}

.close-btn {
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 20px;
  cursor: pointer;
}

input[type="text"],
input[type="number"],
button {
  margin: 5px 0;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
}

button {
  background-color: #3498db;
  color: white;
  cursor: pointer;
}

button:hover {
  background-color: #2980b9;
}
</style>
