<template>
  <b-card
    no-body
    class="selected-item-card"
    :border-variant="isItemSelected ? 'primary' : 'light'"
  >
    <b-container class="my-3">
      <b-row align-v="center">
        <b-col sm="1">
          <b-form-checkbox
            :disabled="!item.availability"
            v-model="isItemSelected"
            @change="itemSelected"
          />
        </b-col>
        <b-col sm="1" v-if="showAvailability">
          <b-icon
            icon="circle-fill"
            :variant="!!item.availability ? 'success' : 'danger'"
          />
        </b-col>
        <b-col :sm="titleSize">
          <b-row>
            <span class="title-text">{{ item.product_name }}</span>
          </b-row>
          <b-row>
            <span class="subtitle-text">{{ item.product_sku }}</span>
          </b-row>
        </b-col>
        <b-col sm="2">
          <div class="right-border border-success">
            <b-row no-gutters>
              <span class="subtitle-text">Price ($)</span>
            </b-row>
            <b-row no-gutters>
              <span class="title-text">{{
                getFormattedNumber(item.selling_price)
              }}</span>
            </b-row>
          </div>
        </b-col>
        <b-col sm="2">
          <div class="right-border border-success">
            <b-row no-gutters>
              <span class="subtitle-text">Availability</span>
            </b-row>
            <b-row no-gutters>
              <span class="title-text">{{ item.availability }}</span>
            </b-row>
          </div>
        </b-col>
        <b-col v-if="isSet">
          <b-icon v-b-toggle="collapseTitle" icon="chevron-down"></b-icon>
        </b-col>
      </b-row>
      <b-collapse :id="collapseTitle">
        <b-row align-h="center" align-v="center">
          <b-container class="mt-2 pl-5" fluid>
            <hr />
            <b-row class="my-3 gray-text"><h5>Included Items</h5></b-row>
            <b-row>
              <b-table
                striped
                :items="item.elements"
                :fields="fields"
              ></b-table>
            </b-row>
          </b-container>
        </b-row>
      </b-collapse>
    </b-container>
  </b-card>
</template>

<script>
import numeral from "numeral";
export default {
  name: "selectable-item",
  props: {
    item: {
      type: Object,
      default: () => ({}),
    },
    showAvailability: {
      type: Boolean,
      default: true,
    },
    isSelected: {
      type: Boolean,
      default: false,
    },
  },
  computed: {
    isSet() {
      return !!this.item.elements.length;
    },
    titleSize() {
      return this.showAvailability ? "5" : "6";
    },
    collapseTitle() {
      return this.item.product_sku + "search_detail";
    },
  },
  data() {
    return {
      isItemSelected: this.isSelected,
      fields: [
        { key: "product_sku", label: "SKU" },
        { key: "product_name", label: "Name" },
        { key: "quantity", sortable: true },
      ],
    };
  },
  methods: {
    itemSelected(newVal) {
      this.$emit("itemSelected", { isSelected: newVal, item: this.item });
    },
    getFormattedNumber(n) {
      return numeral(n).format("0,0.00");
    },
  },
  watch: {
    "item.isSelected"(newVal) {
      this.isItemSelected = newVal;
    },
  },
};
</script>

<style scoped>
.selected-item-card {
  width: 100% !important;
}

.title-text {
  font-size: 18px !important;
  font-weight: 500;
  line-height: 2rem;
}

.subtitle-text {
  font-size: 12px !important;
  font-weight: 300;
  color: #8e9093;
  line-height: 1.5rem;
}
.right-border {
  border-right: 3px solid;
  align-content: right;
  text-align: right !important;
}
.gray-text {
  color: #757c82;
}
</style>
