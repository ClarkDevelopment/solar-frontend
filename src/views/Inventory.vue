<template>
  <div class="inventory-container">
    <h1 id="inventoryTitle">
      Inventory Dashboard
    </h1>
    <hr />

    <inventory-chart />

    <div class="inventory-actions">
      <solar-button @click.native="showNewProductModal" id="addNewItemBtn">
        Add New Item
      </solar-button>
      <solar-button @click.native="showShipmentModal" id="receiveShipmentBtn">
        Receive Shipment
      </solar-button>
    </div>
    <table id="inventoryTable" class="table">
      <tr>
        <th>Item</th>
        <th>Quantity On-hand</th>
        <th>Unit Price</th>
        <th>Taxable</th>
        <th>Delete</th>
      </tr>
      <tr v-for="item in inventory" :key="item.id">
        <td>{{ item.product.name }}</td>
        <td v-bind:class="applyColor(item.quantityOnHand, item.idealQuantity)">{{ item.quantityOnHand }}</td>
        <td>{{ item.product.price | price }}</td>
        <td>
          <span v-if="item.product.isTaxable">
            Yes
          </span>
          <span v-else>
            No
          </span>
        </td>
        <td>
          <div class="lni-cross-circle product-archive" @click="archiveProduct(item.id)">

          </div>
        </td>
      </tr>
    </table>

    <new-product-modal
      v-if="isNewProductVisible"
      @save:product="saveNewProduct"
      @close="closeModals"
    />
    <shipment-modal
      v-if="isShipmentVisible"
      @save:shipment="saveNewShipment"
      :inventory="inventory"
      @close="closeModals"
    />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { IProduct, IProductInventory } from "../../types/Product";
import { IShipment } from "../../types/Shipment";
import {InventoryService} from "@/services/inventory-service";
import {ProductService} from "@/services/product-service";
import SolarButton from "@/components/SolarButton.vue";
import NewProductModal from "@/components/modals/NewProductModal.vue";
import ShipmentModal from "@/components/modals/ShipmentModal.vue";
import InventoryChart from "@/components/charts/InventoryChart.vue";

const inventoryService = new InventoryService();
const productService = new ProductService();

@Component({
  name: "Inventory",
  components: {SolarButton, NewProductModal, ShipmentModal, InventoryChart }
})
export default class Inventory extends Vue {
  isNewProductVisible = false;
  isShipmentVisible = false;

  inventory: IProductInventory[] = [
    {
      id: 1,
      product: {
        id: 2,
        createdOn: new Date(),
        updatedOn: new Date(),
        name: "Some Product",
        description: "stuff",
        price: 100,
        isTaxable: true,
        isArchived: false
      },
      quantityOnHand: 100,
      idealQuantity: 102
    },
    {
      id: 2,
      product: {
        id: 4,
        createdOn: new Date(),
        updatedOn: new Date(),
        name: "Some More Product",
        description: "stuff again",
        price: 500,
        isTaxable: false,
        isArchived: true
      },
      quantityOnHand: 500,
      idealQuantity: 500
    }
  ];

  applyColor(current: number, target: number) {
    if ( current <= 0 )
      return "red";
    if ( Math.abs(target - current) > 8 )
      return "yellow";

    return "green";
  }

  closeModals() {
    this.isShipmentVisible = false;
    this.isNewProductVisible = false;
  }

  showNewProductModal() {
    this.isNewProductVisible = true;
  }

  showShipmentModal() {
    this.isShipmentVisible = true;
  }

  async saveNewProduct(newProduct: IProduct) {
    await productService.addNewProduct(newProduct);
    this.isNewProductVisible = false;
    await this.initialize();
  }

  async saveNewShipment(shipment: IShipment) {
    await inventoryService.updateInventoryQuantity(shipment);
    this.isShipmentVisible = false;
    await this.initialize();
  }

  async archiveProduct(productId: number) {
    await productService.archiveProduct(productId);
    await this.initialize();
  }

  async initialize() {
    this.inventory = await inventoryService.getInventory();
    await this.$store.dispatch("assignSnapshots");

  }

  async created() {
    await this.initialize();
  }
}
</script>

<style scoped lang="scss">
  @import "@/scss/global.scss";

  .green {
    font-weight: bold;
    color: $solar-green;
  }

  .yellow {
    font-weight: bold;
    color: $solar-yellow;
  }

  .red {
    font-weight: bold;
    color: $solar-red;
  }

  .inventory-actions {
    display: flex;
    margin-bottom: 0.8rem;
  }

  .product-archive {
    cursor: pointer;
    font-weight: bold;font-size: 1.2rem;
    color: $solar-red;
  }
</style>
