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
    name: {
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
    distinctValues() {
      return new Set(this.values);
    },
  },

  methods: {
    updateSku() {
      this.$router.push({ path: this.selected });
    },

    generateAttributeCombination(optionAttrValue) {
      return Object.values(this.product.masterData.current.variant.attributes)
        .map((attr) => {
          if (attr.name === this.name) {
            return optionAttrValue;
          }
          return attr.value || attr.label;
        })
        .filter(value => value)
        .join('-');
    },

    attribute2sku(optionAttrValue) {
      const attributeCombination = this.generateAttributeCombination(optionAttrValue);
      return this.attributeCombination2Sku[attributeCombination];
    },
  },
};
</script>
