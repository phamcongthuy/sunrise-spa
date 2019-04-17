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
    assemble(acc, currentItem) {
      (acc[currentItem.name] || (acc[currentItem.name] = []))
        .push(currentItem.value || currentItem.label);
      return acc;
    },
  },

  computed: {
    attributes() {
      return this.product.masterData.current.allVariants
        .flatMap(variant => Object.values(variant.attributes))
        .filter(attr => typeof attr === 'object')
        .reduce(this.assemble, {});
    },

    attributeSku() {
      return this.product.masterData.current.allVariants
        .map(variant => variant.sku);
    },

    attributeCombination2Sku() {
      const input = {};
      Object.values(this.attributes.color)
        .forEach((value, i) => {
          const combo = value.concat('-', this.attributes.size[i]);
          input[combo] = this.attributeSku[i];
        });
      return input;
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
                      size {
                        value
                        name
                      }
                       color {
                        key
                        label(locale: $locale)
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
