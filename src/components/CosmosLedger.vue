<template>
    <div class="cosmosLedger">
        <h3>Detect device</h3>
        <button v-on:click="tryConnect">Connect</button>
        <ul id="ledger-status">
            <li v-for="item in ledgerStatus" v-bind:key="item.index">
                {{ item.msg }}
            </li>
        </ul>
        <hr>
        <h3>Retrieve balance + delegations</h3>
        <button v-on:click="getBalance">Get Balance and delegations</button>
        <ul id="account-status">
            <li v-for="item in accountStatus" v-bind:key="item.index">
                {{ item.msg }}
            </li>
        </ul>
        <hr>
        <h3>Delegate</h3>
        <button v-on:click="delegate">Delegate</button>
        <ul id="delegation-status">
            <li v-for="item in delegationStatus" v-bind:key="item.index">
                {{ item.msg }}
            </li>
        </ul>
    </div>
</template>

<script>
    import CosmosDelegateTool from "cosmos-delegation-js";
    import TransportU2F from '@ledgerhq/hw-transport-u2f';

    const transport = new TransportU2F();
    const cdt = new CosmosDelegateTool(transport);
    cdt.setNodeURL('https://stargate.cosmos.network');

    export default {
        name: 'CosmosLedger',
        props: {
            restUrl: String,
        },
        data() {
            return {
                deviceLog: [],
                accountLog: [],
                delegationLog: [],
                myAddr: ''
            }
        },
        computed: {
            ledgerStatus() {
                return this.deviceLog;
            },
            accountStatus() {
                return this.accountLog;
            },
            delegationStatus() {
                return this.delegationLog;
            }
        },
        methods: {
            log: function (list, msg) {
                list.push({
                    index: list.length,
                    msg: msg
                })
            },
            tryConnect: async function () {
                this.deviceLog = [];
                this.accountLog = [];
                this.delegationLog = [];
                this.myAddr = null;

                this.log(this.deviceLog, "Trying to connect...");

                await cdt.connect();
                if (!cdt.connected) {
                    this.log(this.deviceLog, cdt.lastError);
                    return;
                }
                this.log(this.deviceLog, "Connected!");

                this.myAddr = await cdt.retrieveAddress(0, 0);
                this.log(this.deviceLog, `Address  : ${this.myAddr.bech32}`);
                this.log(this.deviceLog, `PublicKey: ${this.myAddr.pk}`);
            },
            getBalance: async function () {
                if (!cdt.connected) {
                    this.log(this.accountLog, "Try connecting first..");
                    return;
                }

                if (this.myAddr == null) {
                    this.log(this.accountLog, "Retrieve the device address first");
                    return;
                }

                this.log(this.accountLog, `Query ${this.myAddr.bech32}`);

                let accInfo = await cdt.getAccountInfo(this.myAddr);
                this.log(this.accountLog, accInfo);

                const reply = await cdt.retrieveBalances([this.myAddr]);
                this.log(this.accountLog, reply);
            },
            delegate: async function () {
                if (!cdt.connected) {
                    this.log(this.delegationLog, "Try connecting first..");
                    return;
                }
                if (this.myAddr === null) {
                    this.log(this.delegationLog, "Retrieve the device address first");
                    return;
                }
                const txContext = {
                    chainId: 'cosmoshub-2',
                    path: this.myAddr.path,
                    bech32: this.myAddr.bech32,
                    pk: this.myAddr.pk,
                };

                const dummyTx = await cdt.txCreateDelegate(
                    txContext,
                    'validatorAddress',
                    1000,   // TODO change this
                    'delegation Example',
                );

                this.log(this.delegationLog, "Waiting for device to sign");

                const signedTx = await cdt.sign(dummyTx, txContext);

                this.log(this.delegationLog, "Broadcasting signed tx");
                const response = await  cdt.txSubmit(signedTx);

                this.log(this.delegationLog, response);
            }
        }
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    h3 {
        margin: 40px 0 0;
    }

    button {
        padding: 5px;
        font-weight: bold;
        font-size: medium;
    }

    ul {
        padding: 10px;
        text-align: left;
        alignment: left;
        list-style-type: none;
        background: black;
        font-weight: bold;
        color: greenyellow;
    }
</style>
