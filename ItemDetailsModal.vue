<template>
  <b-container fluid>
    <b-row class="detail-title-text mt-1 mb-3 ml-2">
      <h4>Pricing Details</h4>
    </b-row>
    <b-row class="mb-3">
      <b-container fluid>
        <b-row class="my-2">
          <b-col class="text-right detail-title-text" sm="4"
            >Unit Price ($):</b-col
          >
          <b-col class="text-left detail-value-text" sm="2">
            {{ getFormattedNumber(itemUpdateModel.selling_price) }}
          </b-col>
        </b-row>
        <b-row class="my-2">
          <b-col class="text-right detail-title-text" sm="4">Quantity:</b-col>
          <b-col class="text-left detail-value-text" sm="2">
            <b-input
              size="sm"
              v-model="itemUpdateModel.quantity"
              type="number"
              :state="quantityState"
            ></b-input>
          </b-col>
        </b-row>
        <b-row class="my-2">
          <b-col class="text-right detail-title-text" sm="4"
            >Subtotal ($):</b-col
          >
          <b-col class="text-left detail-value-text" sm="2">
            {{
              getFormattedNumber(
                itemUpdateModel.selling_price * itemUpdateModel.quantity
              )
            }}
          </b-col>
        </b-row>
        <b-row class="mt-5 mb-2">
          <b-col class="text-right detail-title-text" sm="4"
            >Discount ($):</b-col
          >
          <b-col class="text-left detail-value-text" sm="2">
            <b-input
              size="sm"
              v-model="itemUpdateModel.discount_offered"
              type="number"
            ></b-input>
          </b-col>
        </b-row>
        <b-row class="my-2 align-bottom">
          <b-col class="text-right detail-title-text" sm="4">Total ($):</b-col>
          <b-col class="text-left total-value-text" sm="2">
            {{
              getFormattedNumber(
                itemUpdateModel.selling_price * itemUpdateModel.quantity -
                  itemUpdateModel.discount_offered
              )
            }}
          </b-col>
        </b-row>
      </b-container>
    </b-row>
    <hr />
    <b-row class="detail-title-text my-3 ml-2">
      <h4>Order Type</h4>
    </b-row>
    <b-row class="ml-5">
      <b-form-group>
        <b-form-radio
          size="lg"
          class="mb-3 mt-2"
          v-model="itemUpdateModel.order_type"
          :value="orderTypes.customerTakes.value"
        >
          {{ orderTypes.customerTakes.text }}
        </b-form-radio>
        <b-form-radio
          size="lg"
          class="mt-4 mb-2"
          v-model="itemUpdateModel.order_type"
          :value="orderTypes.storePickup.value"
        >
          {{ orderTypes.storePickup.text }}
        </b-form-radio>
        <b-form-datepicker
          placeholder="Pickup Date"
          :disabled="!isItemForPickup"
          :state="pickupDateState"
          v-model="itemUpdateModel.pickup_date"
          class="mb-3"
        ></b-form-datepicker>
        <b-form-radio
          size="lg"
          class="mt-4 mb-1"
          v-model="itemUpdateModel.order_type"
          :value="orderTypes.homeDelivery.value"
        >
          {{ orderTypes.homeDelivery.text }}
        </b-form-radio>
        <b-form-datepicker
          placeholder="Delivery Date"
          :disabled="!isItemForDelivery"
          :state="deliveryDateState"
          v-model="itemUpdateModel.delivery_date"
          class="mb-3"
        ></b-form-datepicker>
      </b-form-group>
    </b-row>
  </b-container>
</template>

<script>
import numeral from "numeral";
export default {
  name: "item-details-modal",
  props: {
    itemUpdateModel: Object,
  },
  computed: {
    isItemForDelivery() {
      return this.itemUpdateModel.order_type === "home_delivery";
    },
    isItemForPickup() {
      return this.itemUpdateModel.order_type === "store_pickup";
    },
    deliveryDateState() {
      return this.isItemForDelivery
        ? !!this.itemUpdateModel.delivery_date
        : null;
    },
    pickupDateState() {
      return this.isItemForPickup ? !!this.itemUpdateModel.pickup_date : null;
    },
    quantityState() {
      return !!this.itemUpdateModel.quantity ? null : false;
    },
  },
  data() {
    return {
      orderTypes: {
        customerTakes: { value: "customer_takes", text: "Take Home" },
        storePickup: { value: "store_pickup", text: "Store Pickup" },
        homeDelivery: { value: "home_delivery", text: "Home Delivery" },
      },
    };
  },
  methods: {
    getFormattedNumber(n) {
      return numeral(n).format("0,0.00");
    },
  },
};
</script>

<style>
.detail-title-text {
  color: rgb(126, 138, 148);
  font-size: 1.08rem;
  font-weight: 400;
  line-height: 2rem;
}

.detail-value-text {
  font-size: 1.3rem;
  font-weight: 500;
}

.total-value-text {
  font-size: 1.3rem;
  font-weight: 550;
}
</style>
