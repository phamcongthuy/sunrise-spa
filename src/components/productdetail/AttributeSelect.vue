<template>
  <div>
    <select v-model="selected"
            @change="updateSku"
            class="select-product-detail">
      <option v-for="value in distinctValues"
              :key="value"
              :value="attribute2sku(value)">
        {{value}}
      </option>
    </select>
  </div>
</template>

<script>
export default {
  props: {
    values: {
      type: Array,
      required: true,
    },
    attributeCombination2Sku: {
      type: Object,
      required: true,
    },
    product: {
      type: Object,
      required: true,
    },
  },

  data: () => ({
    selected: '',
  }),

  computed: {
    productVariant() {
      return this.product.masterData.current.variant.attributes;
    },

    distinctValues() {
      return new Set(this.values);
    },
  },

  methods: {
    updateSku() {
      this.$router.push({ path: this.selected });
    },

    generateAttributeCombination(attr) {
      return attr.concat('-', this.productVariant.size.value);
    },

    attribute2sku(attr) {
      return this.attributeCombination2Sku[this.generateAttributeCombination(attr)];
    },
  },

};
</script>
