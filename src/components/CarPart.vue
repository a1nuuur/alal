<template>
  <tr>
    <td>{{ partNameWithParent }}</td>
    <td>{{ calculatedPrice }}</td>
    <td>{{ part.quantity }}</td>
    <td>{{ totalCost }}</td>
    <td>
      <button @click="addPart">Добавить</button>
      <button @click="removePart">Удалить</button>
    </td>
  </tr>

  <template v-if="part.subparts.length">
    <car-part
        v-for="(subpart, index) in part.subparts"
        :key="index"
        :part="subpart"
        :parent="part.name"
    />
  </template>
</template>

<script>
export default {
  name: "CarPart",
  props: {
    part: Object,
    parent: String
  },
  computed: {
    // Если у элемента есть дочерние элементы, цена считается как сумма цен дочерних элементов
    calculatedPrice() {
      if (this.part.subparts.length > 0) {
        return this.part.subparts.reduce(
            (acc, subpart) => acc + subpart.price * subpart.quantity,
            0
        );
      } else {
        return this.part.price;
      }
    },
    totalCost() {
      // Общая стоимость: цена элемента (или его дочерних элементов) умноженная на количество
      return this.calculatedPrice * this.part.quantity;
    },
    partNameWithParent() {
      return this.parent ? `${this.parent} → ${this.part.name}` : this.part.name;
    }
  },
  methods: {
    addPart() {
      this.part.quantity++;
    },
    removePart() {
      if (this.part.quantity > 0) {
        this.part.quantity--;
      }
    }
  }
};
</script>

<style scoped>
button {
  margin: 0 5px;
}
</style>
