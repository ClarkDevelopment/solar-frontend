<template>
    <solar-modal>
        <template v-slot:header>
            Add New Customer
        </template>
        <template v-slot:body>
            <ul class="newCustomer">
                <li>
                    <label for="firstName">First Name</label>
                    <input type="text" id="firstName" v-model="newCustomer.firstName">
                </li>
                <li>
                    <label for="lastName">Last Name</label>
                    <input type="text" id="lastName" v-model="newCustomer.lastName">
                </li>
                <li>
                    <label for="address1">Address Line 1</label>
                    <input type="text" id="address1" v-model="newCustomer.primaryAddress.addressLine1">
                </li>
                <li>
                    <label for="address2">Address Line 2</label>
                    <input type="text" id="address2" v-model="newCustomer.primaryAddress.addressLine2">
                </li>
                <li>
                    <label for="city">City</label>
                    <input type="text" id="city" v-model="newCustomer.primaryAddress.city">
                </li>
                <li>
                    <label for="state">State</label>
                    <input type="text" id="state" v-model="newCustomer.primaryAddress.state">
                </li>
                <li>
                    <label for="postalCode">Postal Code</label>
                    <input type="text" id="postalCode" v-model="newCustomer.primaryAddress.postalCode">
                </li>
                <li>
                    <label for="country">Country</label>
                    <input type="text" id="country" v-model="newCustomer.primaryAddress.country">
                </li>
            </ul>
        </template>
        <template v-slot:footer>
            <solar-button
                    type="button"
                    @click.native="save"
                    aria-label="Save new customer"
            >
                Save Customer
            </solar-button>
            <solar-button
                    type="button"
                    @click.native="close"
                    aria-label="Close the modal"
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
    import {ICustomer} from "../../../types/Customer";

    @Component({
        name: "NewCustomerModal",
        components: { SolarButton, SolarModal }
    })
    export default class NewCustomerModal extends Vue {
        newCustomer: ICustomer = {
            id: 0,
            createdOn: new Date,
            updatedOn: new Date,
            firstName: "",
            lastName: "",
            primaryAddress: {
                id: 0,
                createdOn: new Date,
                updatedOn: new Date,
                addressLine1: "",
                addressLine2: "",
                city: "",
                state: "",
                postalCode: "",
                country: ""
            }
        };

        close() {
            this.$emit("close");
        }

        save() {
            this.$emit("save:customer", this.newCustomer);
        }
    }
</script>

<style scoped lang="scss">
    .newCustomer {
        display: flex;
        flex-wrap: wrap;
        list-style: none;
        padding: 0;
        margin: 0;

        input {
            width: 80%;
            height: 1.8rem;
            margin: 0.8rem 2rem 0.8rem 0.8rem;
            font-size: 1.1rem;
            line-height: 1.3rem;
            padding: 0.2rem;
            color: #555;
        }

        label {
            font-weight: bold;
            display: block;
            margin: 0.8rem;
        }
    }
</style>