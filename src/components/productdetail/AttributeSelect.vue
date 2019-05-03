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
    sku: {
      type: String,
      required: true,
    },
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

  created() {
    this.selected = this.sku;
  },

  watch: {
    sku(newSku) {
      this.selected = newSku;
    },
  },

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

    generateAttributeCombination(attribute) {
      return Object.values(this.productVariant)
        .map(attr => attr.label)
        .filter(value => typeof value === 'string').toString()
        .concat('-', attribute);
    },

    attribute2sku(attr) {
      return this.attributeCombination2Sku[this.generateAttributeCombination(attr)];
    },
  },

};
</script>
