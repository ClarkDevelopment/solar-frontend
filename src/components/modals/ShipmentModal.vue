<template>
  <solar-modal>
    <template v-slot:header>
      Receive Shipment
    </template>
    <template v-slot:body>
      <label for="product">Product Received</label>
      <!-- Selected item gets bound to the selectedProduct model -->
      <select v-model="selectedProduct" class="shipmentItems" id="product">
        <option disabled value="">Please select an Item</option>
        <option v-for="item in inventory" :key="item.product.id" :value="item">
          {{ item.product.name }}
        </option>
      </select>
      <label for="qtyReceived">Quantity Received:</label>
      <input type="number" id="qtyReceived" v-model="qtyReceived" />
    </template>
    <template v-slot:footer>
      <solar-button
        type="button"
        @click.native="save"
        aria-label="Save new shipment"
      >
        Save Received Shipment
      </solar-button>
      <solar-button
        type="button"
        @click.native="close"
        aria-label="Close modal"
      >
        Close
      </solar-button>
    </template>
  </solar-modal>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";
import SolarButton from "@/components/SolarButton.vue";
import SolarModal from "@/components/modals/SolarModal.vue";
import { IProduct, IProductInventory } from "../../../types/Product";
import { IShipment } from "../../../types/Shipment";

@Component({
  name: "ShipmentModal",
  components: { SolarButton, SolarModal }
})
export default class ShipmentModal extends Vue {
  @Prop({ required: true, type: Array as () => IProductInventory[] })
  inventory!: IProductInventory[];

  selectedProduct: IProduct = {
    id: 0,
    createdOn: new Date(),
    updatedOn: new Date(),
    name: "",
    description: "",
    price: 0,
    isTaxable: false,
    isArchived: false
  };

  qtyReceived = 0;

  close() {
    this.$emit("close");
  }
  save() {
    const shipment: IShipment = {
      productId: this.selectedProduct.id,
      adjustment: this.qtyReceived
    };

    this.$emit("save:shipment", shipment);
  }
}
</script>

<style scoped></style>
