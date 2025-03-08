<template>
  <ul>
    <li v-for="(sub, subIndex) in subDetails" :key="sub.id">
      <input v-model="sub.name" placeholder="Название" />
      <input v-model.number="sub.price" type="number" placeholder="Цена" @input="updateParentPrice" />
      <input v-model.number="sub.quantity" type="number" placeholder="Кол-во" @input="updateParentPrice" />

      <button @click="$emit('add-sub-detail', sub)">➕ Поддеталь</button>
      <button @click="$emit('remove-sub-detail', subDetails, subIndex)">❌ Удалить</button>

      <ul v-if="sub.subDetails.length">
        <SubDetailList
          :subDetails="sub.subDetails"
          @add-sub-detail="$emit('add-sub-detail', $event)"
          @remove-sub-detail="$emit('remove-sub-detail', $event)"
          @update-price="$emit('update-price')"
        />
      </ul>
    </li>
  </ul>
</template>

<script>
export default {
  props: {
    subDetails: {
      type: Array,
      required: true
    }
  },
  methods: {
    updateParentPrice() {
      this.$emit("update-price"); // Сообщаем родителю, что нужно обновить цену
    }
  }
};
</script>
