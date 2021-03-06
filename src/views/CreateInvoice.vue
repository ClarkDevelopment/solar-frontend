<template>
    <div>
        <h1 id="invoiceTitle">
            Create Invoice
        </h1>
        <hr/>
        <div class="invoice-step" v-if="invoiceStep === 1">
            <h2>Step 1: Select Customer</h2>
            <div v-if="customers.length" class="invoice-step-detail">
                <label for="customer">Customer:</label>
                <select v-model="selectedCustomerId" class="invoice-customers" id="customer">
                    <option disabled value="">Please select a Customer</option>
                    <option v-for="customer in customers" :value="customer.id">
                        {{ customer.firstName + " " + customer.lastName }}
                    </option>
                </select>
            </div>
        </div>
        <div class="invoice-step" v-if="invoiceStep === 2">
            <h2 class="invoice-step">Step 2: Create Order</h2>
            <div v-if="inventory.length" class="invoice-step-detail">
                <label for="product">Product:</label>
                <select v-model="newItem.product" class="invoice-line-item" id="product">
                    <option disabled value="">Please select a Product</option>
                    <option v-for="i in inventory" :value="i.product" :key="i.product.id">
                        {{ i.product.name }}
                    </option>
                </select>
                <label for="quantity">Quantity:</label>
                <input v-model="newItem.quantity" id="quantity" type="number" min="0"/>
            </div>

            <div class="invoice-item-actions">
                <solar-button
                        :disabled="!newItem.product || !newItem.quantity"
                        @click.native="addLineItem">
                    Add Line Item
                </solar-button>
                <solar-button
                        :disabled="!lineItems.length"
                        @click.native="finalizeOrder">
                    Finalize Order
                </solar-button>
            </div>
            <div class="invoice-order-list" v-if="lineItems.length">
                <div class="runningTotal">
                    <h3>Running Total:</h3>
                    {{ runningTotal | price }}
                </div>
                <hr/>
                <table class="table">
                    <thead>
                        <tr>
                            <th>Product</th>
                            <th>Description</th>
                            <th>Quantity</th>
                            <th>Price</th>
                            <th>Subtotal</th>
                        </tr>
                    </thead>
                    <tr v-for="lineItem in lineItems" :key="'index_'+lineItem.product.id">
                        <td> {{lineItem.product.name}}</td>
                        <td> {{lineItem.product.description}}</td>
                        <td> {{lineItem.quantity}}</td>
                        <td> {{lineItem.product.price}}</td>
                        <td> {{lineItem.product.price * lineItem.quantity | price}}</td>
                    </tr>
                </table>
            </div>
        </div>
        <div class="invoice-step" v-if="invoiceStep === 3">
            <h2>Step 3: Review and Submit</h2>
            <solar-button @click.native="submitInvoice">Submit Invoice</solar-button>
            <hr />

            <div class="invoice-step-detail" id="invoice" ref="invoice">
                <div class="invoice-logo">
                    <img id="imgLogo" alt="Solar Coffee Logo" src="../assets/images/solar_coffee_logo.png">
                    <h3>1337 Solar Lane</h3>
                    <h3>San Somewhere, CA 90000</h3>
                    <h3>USA</h3>

                    <div class="invoice-order-list" v-if="lineItems.length">
                        <div class="invoice-header">
                            <h3>Incoice: {{ new Date() | humanizeDate}}</h3>
                            <h3> Customer: {{this.selectedCustomer.firstName + " " + this.selectedCustomer.lastName}}</h3>
                            <h3>Address: {{this.selectedCustomer.primaryAddress.addressLine1}}</h3>
                            <h3 v-if="this.selectedCustomer.primaryAddress.addressLine2" >
                                {{this.selectedCustomer.primaryAddress.addressLine2}}
                            </h3>
                            <h3>
                                {{this.selectedCustomer.primaryAddress.city}}
                                {{this.selectedCustomer.primaryAddress.state}}
                                {{this.selectedCustomer.primaryAddress.postalCode}}
                            </h3>
                            <h3>{{this.selectedCustomer.primaryAddress.country}}</h3>
                        </div>
                        <table class="table">
                            <thead>
                            <tr>
                                <th>Product</th>
                                <th>Description</th>
                                <th>Quantity</th>
                                <th>Price</th>
                                <th>Subtotal</th>
                            </tr>
                            </thead>
                            <tr v-for="lineItem in lineItems" :key="'index_'+lineItem.product.id">
                                <td> {{lineItem.product.name}}</td>
                                <td> {{lineItem.product.description}}</td>
                                <td> {{lineItem.quantity}}</td>
                                <td> {{lineItem.product.price}}</td>
                                <td> {{lineItem.product.price * lineItem.quantity | price}}</td>
                            </tr>
                            <tr>
                                <th colspan="4"></th>
                                <th>Grand Total</th>
                            </tr>
                            <tfoot>
                                <tr>
                                    <td colspan="4" class="due">Balance due upon receipt:</td>
                                    <td class="price-final"> {{runningTotal | price}}</td>
                                </tr>
                            </tfoot>
                        </table>
                    </div>
                </div>
            </div>
        </div>

        <hr/>
        <div class="invoice-ste-actions">
            <solar-button @click.native="prev" :disabled="!canGoPrev">Previous</solar-button>
            <solar-button @click.native="next" :disabled="!canGoNext">Next</solar-button>
            <solar-button @click.native="startOver">Start Over</solar-button>
        </div>
    </div>
</template>

<script lang="ts">
    import {Component, Vue} from "vue-property-decorator";
    import {IInvoice, ILineItem} from "../../types/Invoice";
    import {ICustomer} from "../../types/Customer";
    import {IProductInventory} from "../../types/Product";
    import {CustomerService} from "@/services/customer-service";
    import {InvoiceService} from "@/services/invoice-service";
    import {InventoryService} from "@/services/inventory-service";
    import SolarButton from "@/components/SolarButton.vue";
    import jsPDF from "jspdf";
    import html2canvas from 'html2canvas';

    const customerService = new CustomerService();
    const inventoryService = new InventoryService();
    const invoiceService = new InvoiceService();

    @Component({
        name: "CreateInvoice",
        components: {SolarButton}
    })
    export default class CreateInvoice extends Vue {
        invoiceStep: number = 1;
        invoice: IInvoice = {
            createdOn: undefined,
            customerId: 0,
            lineItems: [],
            updatedOn: undefined
        }

        customers: ICustomer[] = [];
        selectedCustomerId: number = 0;
        inventory: IProductInventory[] = [];
        lineItems: ILineItem[] = [];
        newItem: ILineItem = {
            product: undefined,
            quantity: 0
        };

        get canGoPrev() {
            return this.invoiceStep !== 1;
        }

        get canGoNext() {
            if (this.invoiceStep ===1) {
                return this.selectedCustomerId !== 0;
            }
            if (this.invoiceStep === 2) {
                return this.lineItems.length;
            }
            if ( this.invoiceStep ===3) {
                return false;
            }
            return false;
        }

        get runningTotal() {
            return this.lineItems.reduce((a, b) => a + (b['product']['price'] * b["quantity"]), 0);
        }

        get selectedCustomer() {
            return this.customers.find(c => c.id == this.selectedCustomerId);
        }

        async submitInvoice(): Promise<void> {
            this.invoice = {
                createdOn: new Date,
                updatedOn: new Date,
                customerId: this.selectedCustomerId,
                lineItems: this.lineItems
            };

            await invoiceService.makeNewInvoice(this.invoice);
            this.downloadPdf();
            await this.$router.push("/orders");
        }

        downloadPdf() {
            let pdf = new jsPDF("p", "pt", "a4");
            let invoice = document.getElementById("invoice");
            let width = invoice.clientWidth;
            let height = invoice.clientHeight;

            html2canvas(invoice).then(canvas => {
               let image = canvas.toDataURL("image/png");
               pdf.addImage(image, "PNG", 0, 0, width * 0.55, height * 0.55);
               pdf.save("invoice");
            });
        }

        addLineItem() {
            let newItem: ILineItem = {
                product: this.newItem.product,
                quantity: Number(this.newItem.quantity)
            }

            console.log(newItem);
            console.log(this.newItem);

            let existingItems = this.lineItems.map(item => item.product.id);
            console.log(existingItems);
            if (existingItems.includes(newItem.product.id)) {
                let lineItem = this.lineItems.find(
                    item => item.product.id === newItem.product.id
                );

                let currentQuantity = Number(lineItem.quantity);
                let updatedQuantity = (currentQuantity + newItem.quantity);
                lineItem.quantity = updatedQuantity;
            } else {
                this.lineItems.push(this.newItem);
            }

            this.newItem = {
                product: undefined,
                quantity: 0
            };
        }

        finalizeOrder() {
            this.invoiceStep = 3;
        }

        prev(): void {
            if (this.invoiceStep === 1) return;
            this.invoiceStep--;
        }

        next(): void {
            if (this.invoiceStep === 3) return;
            this.invoiceStep++;
        }

        startOver(): void {
            this.invoiceStep = 1;
        }

        async initialize() {
            this.customers = await customerService.getCustomers();
            this.inventory = await inventoryService.getInventory();
        }

        async created() {
            await this.initialize();
        }

    }
</script>

<style scoped lang="scss">
    @import "@/scss/global.scss";
    .invoice-steps-action {
        display: flex;
        width: 100%;
    }

    .invoice-step {

    }

    .invoice-step-detail {
        margin: 1.2rem;
    }

    .invoice-order-list {
        margin-top: 1.2rem;
        padding: 0.8rem;

        .line-item {
            display: flex;
            border-bottom: 1px dashed #ccc;
            padding: 0.8rem;
        }

        .item-col {
            flex-grow: 1;
        }
    }

    .invoice-item-actions {
        display: flex;
    }

    .price-pre-tax {
        font-weight: bold;
    }

    .price-final {
        font-weight: bold;
        color: $solar-green;
    }

    .due {
        font-weight: bold;
    }

    .invoice-header {
        width: 100%;
        margin-bottom: 1.2rem;

        img {
            width: 280px;
        }
    }

    #imgLogo {
        width: 280px;
    }
</style>