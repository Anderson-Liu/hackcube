<template>
  <div class="board">
    <cube-nav/>
    <div v-for="latest_crf_item in latest_crf_items" >
      <b-alert show variant="primary" dismissible>
        <h4 class="alert-heading">RF</h4>
        <p>
          In frequency {{latest_crf_item.freq}} found prot: {{latest_crf_item.prot}}, data: {{latest_crf_item.data}} signal.<u @click="clickRF">Learn More.</u>
        </p>
      </b-alert>
    </div>
    <b-alert :show="showNFCAlert" variant="success" dismissible>
      <h4 class="alert-heading">NFC</h4>
      <p>
        Found NFC card ID: {{latest_nfc_item.ID}}, <a href="#content">Learn More.</a>
      </p>
    </b-alert>
    <h1 class="text-center">Cube NFC Manage</h1>
    <h3 class="text-center">Safety Risk Detection for Cards Working at 125Khz, 13.5Mhz.</h3>
    <br/>
    <!-- Read List -->
    <b-container>
      <b-row align-h="between">
        <b-col cols="4">
          <h5>Read</h5>
        </b-col>
        <b-col cols="2">
          <van-switch v-model="readSwitch" @change="onSwitchRead" size="25px"/>
        </b-col>
      </b-row>
    </b-container>

    <b-table id="content" :items="items" :fields="fields">
      <div slot="WRITE" slot-scope="data">
        <van-switch v-model="items[data.index].WRITE" @change="onSwitchAction('nw', items[data.index].WRITE, items[data.index].VID, items[data.index].ID)" size="25px"/>
      </div>
      <div slot="SIMULATE" slot-scope="data">
        <van-switch v-model="items[data.index].SIMULATE" @change="onSwitchAction('ns', items[data.index].SIMULATE, items[data.index].VID, items[data.index].ID)" size="25px"/>
      </div>
    </b-table>

    <!-- Write List -->
    <b-container style="margin-top: 15px">
      <b-row align-h="between">
        <b-col cols="4">
          <h5>Write</h5>
        </b-col>
        <b-col cols="3">
          <van-switch v-model="writeSwitch" @change="onSwitchAction('nw', writeSwitch)" size="25px"/>
        </b-col>
      </b-row>
    </b-container>

    <b-container>
      <b-row>
        <b-col cols="4">
          <b-form-input v-model="writeVid"
                        type="text"
                        placeholder="VID">
          </b-form-input>
        </b-col>
        <b-col cols="8">
          <b-form-input v-model="writeId"
                        type="text"
                        placeholder="ID">
          </b-form-input>
        </b-col>
      </b-row>
    </b-container>

    <!-- Simulate List -->
    <b-container style="margin-top: 15px">
      <b-row align-h="between">
        <b-col cols="4">
          <h5>Simulate</h5>
        </b-col>
        <b-col cols="3">
          <van-switch v-model="simulateSwitch" @change="onSwitchAction('ns', simulateSwitch)" size="25px"/>
        </b-col>
      </b-row>
    </b-container>

    <b-container fluid>
      <b-row>
        <b-col cols="4">
          <b-form-input v-model="simulateVid"
                        type="text"
                        placeholder="VID">
          </b-form-input>
        </b-col>
        <b-col cols="8">
          <b-form-input v-model="simulateId"
                        type="text"
                        placeholder="ID">
          </b-form-input>
        </b-col>
      </b-row>
    </b-container>

    <br/>
    <Input readonly v-model="updateLog" type="textarea" :autosize="{minRows: 6,maxRows: 14}" placeholder="NFC card information"></Input>
    <br/><br/>
  </div>
</template>

<script>
  /* eslint-disable no-restricted-syntax */

  import CubeNav from '@/components/CubeNav';
  import axios from 'axios';
  import AxiosMixin from '../axios-mixin';

  const arrayMove = require('array-move');

  function isValidIds(Vid, Id) { return /^\w+$/.test(Vid) && /^\w+$/.test(Id); }
  export default {
    name: 'NFC',
    components: {
      CubeNav,
    },
    mixins: [AxiosMixin],
    data() {
      return {
        updateLog: '',
        switchSize: '25px',
        readSwitch: true,
        // TODO: Confirm if necessary to disable input when false
        writeSwitch: false,
        simulateSwitch: false,
        writeVid: null,
        writeId: null,
        simulateVid: null,
        simulateId: null,
        fields: ['VID', 'ID', 'WRITE', 'SIMULATE'],
        // items: [
        //   { VID: '050', ID: '000279080', WRITE: false, SIMULATE: false },
        // ],
        items: [],
        latest_crf_items: [],
        showNFCAlert: false,
        latest_nfc_item: '',
      };
    },
    methods: {
      fetchNFCLog() {
        axios
          .get(`${process.env.BACKEND_HOST}/nfc_log`, {
            validateStatus(status) {
              return status < 400;
            },
          })
          .then((response) => {
            const result = response.data;
            if (response.status === 304) {
              return;
            }
            this.updateLog = result[result.data_key];
          })
          .catch((err) => {
            if (err.response) {
              this.$Message.error(err.response.data.message);
            } else {
              this.$Message.error('Request fail');
            }
          });
      },
      fetchAllNFCData() {
        return axios
          .get(`${process.env.BACKEND_HOST}/all_nfc_items`, {
            validateStatus(status) {
              return status < 400; // Reject only if the status code is greater than or equal to 400
            },
          })
          .then((response) => {
            const result = response.data;
            if (response.status === 304) {
              return;
            }
            this.items = result[result.data_key];
            this.$Message.info(result.message);
          })
          .catch((err) => {
            if (err.response) {
              this.$Message.error(err.response.data.message);
            } else {
              this.$Message.error('Request fail');
            }
          });
      },
      fetchNFCData() {
        axios
          .get(`${process.env.BACKEND_HOST}/nfc_item`, {
            validateStatus(status) {
              return status < 400; // Reject only if the status code is greater than or equal to 400
            },
          })
          .then((response) => {
            const result = response.data;
            if (response.status === 304) {
              return;
            }
            const item = result[result.data_key];
            const index = this.items.findIndex(x => x.ID === item.ID);
            this.latest_nfc_item = item;
            if (index === -1) {
              this.items.unshift(item);
            } else {
              this.items = arrayMove(this.items, index, 0);
            }
            this.showNFCAlert = true;
            this.$Message.info('Detect new nfc data.');
          })
          .catch((err) => {
            if (err.response) {
              this.$Message.error(err.response.data.message);
            } else {
              this.$Message.error('Request fail');
            }
          });
      },
      serialSend(parameter) {
        axios
          .get(`${process.env.BACKEND_HOST}/serial_send/${parameter}`)
          .then((response) => {
            const message = response.data.message;
            this.$Message.success(message);
          })
          .catch((err) => {
            if (err.response) {
              this.$Message.error(err.response.data.message);
            } else {
              this.$Message.error('Request fail');
            }
          });
      },
      onSwitchRead(checked) {
        if (checked) {
          this.$timer.start('fetchNFCData');
        } else {
          this.$timer.stop('fetchNFCData');
        }
      },
      onSwitchAction(actionType, isOpen, VID, ID) {
        if (!isOpen) {
          return;
        }

        if (VID && ID) {
          const parameter = `${actionType}${VID}${ID}`;
          this.serialSend(parameter);
          return;
        }
        switch (actionType) {
          case 'nw':
            if (!(this.writeVid && this.writeId)) {
              this.$Message.error('Please input ID and VID');
              this.writeSwitch = !this.writeSwitch;
            } else if (!(isValidIds(this.writeVid, this.writeId))) {
              this.$Message.error('Please input the right ID and VID');
              this.writeSwitch = !this.writeSwitch;
            } else {
              const parameter = `${actionType}${this.writeVid}${this.writeId}`;
              this.serialSend(parameter);
            }
            break;
          case 'ns':
            if (!(this.simulateVid && this.simulateId)) {
              this.$Message.error('Please input the right ID and VID');
              this.simulateSwitch = !this.simulateSwitch;
            } else if (!(isValidIds(this.simulateVid, this.simulateId))) {
              this.$Message.error('Please input the right ID and VID');
              this.simulateSwitch = !this.simulateSwitch;
            } else {
              const parameter = `${actionType}${this.simulateVid}${this.simulateId}`;
              this.serialSend(parameter);
            }
            break;
          default:
            this.$Message.error('Please input the right API');
            break;
        }
      },
    },
    created() {
      let needFetchAll = true;
      if (this.$route.params.latest_nfc_item) {
        needFetchAll = false;
        this.fetchAllNFCData().then(() => {
          const item = this.$route.params.latest_nfc_item;
          const index = this.items.findIndex(x => x.ID === item.ID);
          if (index === -1) {
            this.items.unshift(item);
          } else {
            this.items = arrayMove(this.items, index, 0);
          }
        });
      }
      if (this.readSwitch) {
        this.$timer.start('fetchNFCData');
      } else if (this.readSwitch === false) {
        this.$timer.stop('fetchNFCData');
      }
      if (needFetchAll) {
        this.fetchAllNFCData();
      }
    },
    timers: {
      fetchNFCData: { time: 3000, autostart: false, repeat: true },
      fetchNFCLog: { time: 5000, autostart: true, repeat: true },
      fetchCRFItems: { time: 3400, autostart: true, repeat: true },
    },
  };
</script>

<style scoped>

</style>
