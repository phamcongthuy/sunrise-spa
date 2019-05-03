<template>
  <div v-if="product"
       class="row select-row">
    <ul class="list-inline">
      <li v-for="(values, name) in attributes"
          :key="name">
        <p class="text-uppercase">
          {{name}}
        </p>
        <AttributeSelect :product="product"
                         :values="values"
                         :sku="sku"
                         :attributeCombination2Sku="attributeCombination2Sku" />
      </li>
    </ul>
  </div>
</template>

<script>
import gql from 'graphql-tag';
import AttributeSelect from './AttributeSelect.vue';

export default {
  components: { AttributeSelect },

  props: {
    sku: {
      type: String,
      required: true,
    },
  },

  data: () => ({
    product: null,
  }),

  methods: {
    groupValuesByAttribute(acc, currentItem) {
      if (!acc[currentItem.name]) {
        acc[currentItem.name] = [];
      }
      acc[currentItem.name].push(currentItem.value || currentItem.label);
      return acc;
    },

    generateCombination(attributes) {
      return Object.values(attributes)
        .map(attribute => attribute.label || attribute.value)
        .filter(value => typeof value === 'string')
        .join('-');
    },
  },

  computed: {
    attributes() {
      return this.product.masterData.current.allVariants
        .flatMap(variant => Object.values(variant.attributes))
        .filter(attr => typeof attr === 'object')
        .reduce(this.groupValuesByAttribute, {});
    },

    attributeCombination2Sku() {
      return this.product.masterData.current.allVariants
        .map((variant) => {
          const combi = this.generateCombination(variant.attributes);
          const { sku } = variant;
          return { combi, sku };
        })
        .reduce((acc, value) => {
          acc[value.combi] = value.sku;
          return acc;
        });
    },
  },

  apollo: {
    product: {
      query: gql`
        query VariantSelector($locale: Locale!, $sku: String!) {
          product(sku: $sku) {
            id
            masterData {
              current {
                allVariants {
                  sku
                  attributes {
                    ...on mainProductType {
                      color {
                        key
                        label(locale: $locale)
                        name
                      }
                      size {
                        value
                        name
                      }
                    }
                  }
                }
                variant(sku: $sku) {
                  attributes {
                    ...on mainProductType {
                      color {
                        key
                        label(locale: $locale)
                        name
                      }
                      size {
                        value
                        name
                      }
                    }
                  }
                }
              }
            }
          }
        }`,
      variables() {
        return {
          locale: this.$i18n.locale,
          sku: this.sku,
        };
      },
    },
  },
};
</script>
