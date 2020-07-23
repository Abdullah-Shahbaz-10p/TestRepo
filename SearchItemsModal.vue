<template>
  <b-container>
    <b-row align-h="center">
      <b-col sm="8">
        <b-input-group class="mt-1">
          <template v-slot:prepend>
            <b-input-group-text class="bg-transparent text-primary">
              <b-icon icon="search"></b-icon>
            </b-input-group-text>
          </template>
          <b-form-input
            placeholder="Search"
            v-model="searchModel.keyword"
          ></b-form-input>
        </b-input-group>
      </b-col>
      <b-col sm="2">
        <b-button class="mt-1" variant="primary" @click="searchAction">
          Search
        </b-button>
      </b-col>
    </b-row>
    <b-row align-h="center" class="my-3">
      <b-form-checkbox v-model="showAvailableItemsOnly" switch>
        <b>Show Only Available Items</b>
      </b-form-checkbox>
    </b-row>
    <b-row class="my-2">
      <b-col
        sm="3"
        class="mt-3"
        v-for="item of selectedItems"
        :key="item.product_sku"
      >
        <b-button
          size="sm"
          :variant="item.isOfInvoiceList ? 'outline-primary' : 'outline-danger'"
          :disabled="item.isOfInvoiceList"
          @click="removeItem(item)"
        >
          {{ item.product_name }}
          <b-icon v-if="!item.isOfInvoiceList" icon="x" />
        </b-button>
      </b-col>
    </b-row>
    <template v-if="searchModel.result.length">
      <b-row class="mt-2" v-if="searchedSets.length">
        <h4>Sets</h4>
        <b-container fluid>
          <b-row
            class="my-3"
            v-for="set of searchedSets"
            :key="set.product_sku"
          >
            <selectable-item
              :item="set"
              :isSelected="isItemSelected(set)"
              :showAvailability="!showAvailableItemsOnly"
              @itemSelected="addItemInCheckedList"
            ></selectable-item>
          </b-row>
        </b-container>
      </b-row>
      <b-row class="mt-3" v-if="searchedItems.length">
        <h4>Items</h4>
        <b-container fluid>
          <b-row
            class="my-3"
            v-for="item of searchedItems"
            :key="item.product_sku"
          >
            <selectable-item
              :item="item"
              :isSelected="isItemSelected(item)"
              :showAvailability="!showAvailableItemsOnly"
              @itemSelected="addItemInCheckedList"
            ></selectable-item>
          </b-row>
        </b-container>
      </b-row>
    </template>
    <template v-else>
      <b-row class="mt-3"><h6>Please search to see a list of Items</h6></b-row>
    </template>
  </b-container>
</template>

<script>
import SelectableItem from "./SelectableItem";
export default {
  name: "search-items-modal",
  props: {
    searchModel: {
      type: Object,
      default: () => ({}),
    },
    searchAction: {
      default: () => {},
    },
    invoiceSelectedItems: {
      type: Array,
      default: () => [],
    },
    selectedItems: {
      type: Array,
      default: () => [],
    },
    selectedItemsSkuSet: {
      type: Set,
      default: () => new Set(),
    },
  },
  components: { SelectableItem },
  computed: {
    searchResult() {
      return this.showAvailableItemsOnly
        ? this.searchModel.result.filter((item) => !!item.availability)
        : this.searchModel.result;
    },
    searchedSets() {
      return this.searchResult.filter((item) => !!item.elements.length);
    },
    searchedItems() {
      return this.searchResult.filter((item) => !item.elements.length);
    },
  },
  data() {
    return {
      showAvailableItemsOnly: true,
    };
  },
  methods: {
    addItemInCheckedList({ item, isSelected }) {
      if (isSelected) {
        this.selectedItemsSkuSet.add(item.product_sku);
        this.selectedItems.push(item);
        this.$set(
          this.selectedItems[this.selectedItems.length - 1],
          "isSelected",
          true
        );
        return;
      }

      const itemIndex = this.selectedItems.findIndex(
        ({ product_sku }) => item.product_sku === product_sku
      );
      this.selectedItems.splice(itemIndex, 1);
      this.selectedItemsSkuSet.delete(item.product_sku);
      item.isSelected = false;
    },
    removeItem(item) {
      this.addItemInCheckedList({ item, isSelected: false });
      if (item.isOfInvoiceList) {
        this.$emit("deleteInvoiceItem", item);
      }
    },
    isItemSelected(item) {
      return this.selectedItemsSkuSet.has(item.product_sku);
    },
  },
};
</script>
