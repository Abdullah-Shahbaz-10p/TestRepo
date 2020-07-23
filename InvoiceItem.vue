<template>
  <b-card
    class="invoice-item-card my-2"
    no-body
    header-bg-variant="transparent"
  >
    <template v-slot:header>
      <b-container fluid>
        <b-row align-v="center">
          <b-col sm="6">
            <span class="title-text">{{ item.product_name }}</span>
          </b-col>
          <b-col sm="2" class="text-right">
            <span class="value-text"
              >${{ getFormattedNumber(item.selling_price) }}</span
            >
          </b-col>
          <b-col sm="3" class="text-center">
            <b-icon
              icon="dash"
              font-scale="1.4"
              class="rounded-circle bg-danger"
              variant="white"
              @click="decreaseQuantity"
            ></b-icon>
            <span class="px-3 value-text">{{ item.quantity }}</span>
            <b-icon
              icon="plus"
              font-scale="1.4"
              class="rounded-circle bg-primary"
              variant="white"
              @click="item.quantity++"
            ></b-icon>
          </b-col>
          <b-col sm="1">
            <b-button variant="link" href="#" tabindex="0">
              <b-icon
                :id="item.product_sku"
                font-scale="1.5"
                class="float-right"
                variant="dark"
                icon="three-dots-vertical"
              ></b-icon>
            </b-button>
          </b-col>
        </b-row>
      </b-container>
    </template>
    <b-card-body>
      <b-container>
        <b-row no-gutters>
          <b-col sm="6">
            <b-container fluid>
              <InvoiceDetailAttribute
                v-for="attr of itemDetailAttributes.left"
                :key="attr.key"
                :title="attr.key"
                :value="attr.valueFunc()"
              />
            </b-container>
          </b-col>
          <b-col sm="6">
            <b-container fluid>
              <InvoiceDetailAttribute
                v-for="attr of itemDetailAttributes.right"
                :key="attr.key"
                :title="attr.key"
                :value="attr.valueFunc()"
              />
            </b-container>
          </b-col>
        </b-row>
      </b-container>
    </b-card-body>

    <b-popover :target="item.product_sku" triggers="focus" placement="left">
      <b-list-group flush>
        <b-list-group-item
          v-for="option of itemPopOverOptions"
          :key="option.title"
          @click="option.action"
        >
          {{ option.title }}
        </b-list-group-item>
      </b-list-group>
    </b-popover>

    <b-modal
      :id="item.product_sku + 'items-modal'"
      scrollable
      size="lg"
      hide-footer
      header-class="title-text"
      :title="item.product_name"
    >
      <p class="value-text text-gray">Items Selected</p>
      <b-table striped :items="item.children" :fields="fields"></b-table>
    </b-modal>

    <b-modal
      :id="item.product_sku + 'edit-modal'"
      scrollable
      size="lg"
      header-class="title-text"
      :title="item.product_name"
      @ok="updateItem"
    >
      <InvoiceDetailsModal :itemUpdateModel="modalUpdateItem" />
    </b-modal>
  </b-card>
</template>

<script>
import InvoiceDetailAttribute from "./InvoiceDetailAttribute";
import InvoiceDetailsModal from "./ItemDetailsModal";
import moment from "moment";
import numeral from "numeral";

export default {
  name: "invoice-item",
  components: { InvoiceDetailAttribute, InvoiceDetailsModal },
  props: {
    item: {
      type: Object,
    },
  },
  data() {
    let self = this;
    let itemPopOverOptions = [
      {
        title: "Edit Details",
        action: self.editItemDetails,
      },
      {
        title: "Delete",
        action: self.deleteItem,
      },
    ];
    return {
      orderTypeKeyValueMap: {
        customer_takes: "Take Home",
        store_pickup: "Store Pickup",
        home_delivery: "Home Delivery",
      },
      itemPopOverOptions: (!!self.item.children.length
        ? [{ title: "View Items", action: self.showItemsList }]
        : []
      ).concat(itemPopOverOptions),
      itemDetailAttributes: {
        left: [
          {
            key: "Product SKU:",
            valueFunc: () => self.item.product_sku,
          },
          {
            key: "Order Type:",
            valueFunc: () =>
              self.orderTypeKeyValueMap[self.item.order_type] ||
              "Please select Order Type from options",
          },
          {
            key: "Date:",
            valueFunc: () => self.getOrderDate(),
          },
        ],
        right: [
          {
            key: "Subtotal ($):",
            valueFunc: () =>
              self.getFormattedNumber(
                self.item.quantity * self.item.selling_price
              ),
          },
          {
            key: "Discount ($):",
            valueFunc: () =>
              self.item.discount_offered
                ? self.getFormattedNumber(self.item.discount_offered)
                : "Please add Discount from options",
          },
          {
            key: "Total ($):",
            valueFunc: () =>
              self.getFormattedNumber(
                self.item.quantity * self.item.selling_price -
                  self.item.discount_offered
              ),
          },
        ],
      },
      fields: [
        { key: "product_sku", label: "SKU" },
        { key: "product_name", label: "Name" },
        { key: "quantity", sortable: true },
      ],
      modalUpdateItem: self.item,
    };
  },
  methods: {
    decreaseQuantity() {
      if (this.item.quantity === 1) {
        return;
      }
      this.item.quantity--;
    },
    getOrderDate() {
      let date = "N/A",
        item = this.item;

      switch (item.order_type) {
        case "customer_takes":
          date = "Today";
          break;
        case "store_pickup":
          date = moment(item.pickup_date).format("MMMM Do YYYY");
          break;
        case "home_delivery":
          date = moment(item.delivery_date).format("MMMM Do YYYY");
          break;
      }
      return date;
    },
    editItemDetails() {
      this.modalUpdateItem = Object.assign({}, this.item);
      this.$bvModal.show(this.item.product_sku + "edit-modal");
    },
    deleteItem() {
      this.$emit("deleteInvoiceItem", this.item);
    },
    showItemsList() {
      this.$bvModal.show(this.item.product_sku + "items-modal");
    },
    getFormattedNumber(n) {
      return numeral(n).format("0,0.00");
    },
    updateItem(event) {
      let item = this.item,
        updateItem = this.modalUpdateItem;

      if (
        updateItem.quantity < 1 ||
        (updateItem.order_type === "store_pickup" && !updateItem.pickup_date) ||
        (updateItem.order_type === "home_delivery" && !updateItem.delivery_date)
      ) {
        event.preventDefault();
      }

      Object.keys(item).forEach((key) => {
        if (item[key] !== updateItem[key]) {
          item[key] = updateItem[key];
        }
      });

      switch (item.order_type) {
        case "customer_takes":
          item.pickup_date = undefined;
          item.delivery_date = undefined;
        case "store_pickup":
          item.delivery_date = undefined;
          break;
        case "home_delivery":
          item.pickup_date = undefined;
          break;
      }
    },
  },
};
</script>

<style>
.invoice-item-card {
  width: 100% !important;
}

.title-text {
  font-size: 1.15rem;
  font-weight: 550;
}

.value-text {
  font-size: 1.08rem;
  font-weight: 500;
}
</style>
