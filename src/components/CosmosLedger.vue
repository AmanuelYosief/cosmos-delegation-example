<template>
    <div class="cosmosLedger">
        <h3>Detect device</h3>
        <button v-on:click="tryConnect">TryConnect</button>
        {{status}}

        <h3>Retrieve balance + delegations</h3>
        <button v-on:click="getBalance">Get Balance</button>

        <h3>Delegate</h3>
        <button v-on:click="delegate">Delegate</button>
    </div>
</template>

<script>
    import CosmosDelegateTool from "cosmos-delegation-js";
    import TransportU2F from '@ledgerhq/hw-transport-u2f';
    const transport = new TransportU2F();
    const cdt = new CosmosDelegateTool(transport);

    export default {
        name: 'CosmosLedger',
        props: {
            restUrl: String,
        },
        data() {
            return {
                ledgerStatus: ""
            }
        },
        computed: {
            status() {
                return this.ledgerStatus;
            }
        },
        methods: {
            tryConnect: async function () {
                this.ledgerStatus = "Trying to connect...";
                await cdt.connect();
                if (!cdt.connected) {
                    this.ledgerStatus = cdt.lastError;
                    return;
                }
                this.ledgerStatus = "Connected!";
            },
            getBalance: async function () {
            },
            delegate: async function () {
            }
        }
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    h3 {
        margin: 40px 0 0;
    }
</style>
