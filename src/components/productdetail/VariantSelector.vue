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
      const blah = this.product.masterData.current.allVariants
        .flatMap(variant => Object.values(variant.attributes))
        .filter(attr => typeof attr === 'object');
      return blah.reduce(this.assemble, {});
    },

    attributeCombination2Sku() {
      return {
        'black-34': 'M0E20000000DLPQ',
        'white-34': 'M0E20000000DLPA',
        'green-34': 'M0E20000000ED0A',
      };
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
