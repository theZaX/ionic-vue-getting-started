<template>
  <base-layout page-title="Plane Control Panel">
    <ion-content>
      <ion-grid>
        <ion-row class="ion-text-center ion-no-padding">
          <ion-col>
            <h4 class="ion-padding-horizontal">Status Bar</h4>
          </ion-col>
        </ion-row>
        <ion-row>
          <ion-col>
            <p class="ion-float-left">battery status</p>
            <p class="ion-float-right">signal status</p>
          </ion-col>
        </ion-row>

        <ion-row>
        {{sliderValues[0]}}
        </ion-row>

        <ion-row>

          <ion-col>
            <ion-item>
              <ion-range
                v-model="sliderValues[0]"
                min="0"
                max="1024"
                color="danger"
                pin="true"
              >
                <ion-label slot="start">0</ion-label>
                <ion-label slot="end">100%</ion-label>
              </ion-range>
            </ion-item>
          </ion-col>
        </ion-row>

        <ion-row>
        {{sliderValues[1]}}
        </ion-row>


        <ion-row>

          <ion-col>
            <ion-item>
              <ion-range
                v-model="sliderValues[1]"
                min="-200"
                max="200"
                color="danger"
                pin="true"
              >
                <ion-label slot="start">-200</ion-label>
                <ion-label slot="end">200</ion-label>
              </ion-range>
            </ion-item>
          </ion-col>
        </ion-row>

        <ion-row class="ion-align-items-end">
          <ion-col size="8">
            <ion-item>
              <ion-label position="floating">Message</ion-label>
              <ion-input v-model="sendBarInfo"></ion-input>
            </ion-item>
          </ion-col>

          <ion-col size="4">
            <ion-button @click="SendFromBar" expand="block">Send</ion-button>
          </ion-col>
        </ion-row>
      </ion-grid>

      <!-- <ion-list lines="none">
        <ion-card v-for="message in connectionHistory" :key="message.body"
          ><ion-item
            >{{ message.sender }}: {{ message.body }}</ion-item
          ></ion-card
        >
      </ion-list> -->
    </ion-content>
  </base-layout>
</template>

<script>
import { add } from "ionicons/icons";

// import RangeTest from "../components/control/range-test.vue";
// import ButtonTest from "../components/control/button-test.vue";
// import ScrollControl from "../components/control/ScrollControl.vue";

const IP_ADDRESS = "192.168.1.10";
const UDP_PORT = 1234;

import { Plugins } from "@capacitor/core";
const { UdpPlugin } = Plugins;
//import {UdpPluginUtils} from "capacitor-udp";

import {
  IonCol,
  IonGrid,
  IonRow,
  IonButton,
  IonItem,
  IonLabel,
  IonInput,
  IonRange,
  IonContent,
  //IonList,
  //IonCard,
} from "@ionic/vue";

export default {
  components: {
    IonCol,
    IonGrid,
    IonRow,
    IonButton,
    IonItem,
    IonLabel,
    IonInput,
    IonRange,
    IonContent,
    //IonList,
    //IonCard,
  },
  data() {
    return {
      add,
      isConnected: false,
      connectionHistory: [],
      sendBarInfo: "faf",
      planeSocket: null,
      sliderValues: [0, 0],
      socketId: null,
      packetNumber: 0
    };
  },
  computed: {},
  methods: {
    SendFromBar() {
      this.SendMessage("Client", this.sendBarInfo);
      this.sendBarInfo = "";
    },
    SendMessage(sender, body) {
      const messageInfo = {
        body: body,
        sender: sender,
      };

      this.LogMessage(sender, body);

      UdpPlugin.send({
        socketId: this.socketId,
        address: IP_ADDRESS,
        port: UDP_PORT,
        buffer: btoa(messageInfo),
      });
    },
    YeetSliderInfo() {
      this.SendMessage("Slider", this.sliderValue);
    },

    LogMessage(sender, body) {
      const messageInfo = {
        body: body,
        sender: sender,
      };
      this.connectionHistory.unshift(messageInfo);
    },
    CreateUdpSocket() {
      this.LogMessage("System", "about to create dealio");
      UdpPlugin.create({
        properties: { name: "yourSocketName", bufferSize: 2048 },
      }).then((res) => {
        this.socketId = res.socketId;
        this.LogMessage("System", "dealio created");
        UdpPlugin.bind({ socketId: res.socketId, port: 5000 });
      });
    },
    InboundMessageHandler(data) {
      this.LogMessage("Plane", atob(data.buffer));
      const recievedObject = JSON.parse(atob(data.buffer));
      console.log(recievedObject);
    },
    ConnectionHandler() {
      this.packetNumber++
      
      const controlPacket = {
        type: "control",
        SliderValues: this.sliderValues,
        PacketNumber: this.packetNumber
      }

      UdpPlugin.send({
        socketId: this.socketId,
        address: IP_ADDRESS,
        port: UDP_PORT,
        buffer: btoa(JSON.stringify(controlPacket)),
      });

      this.LogMessage("System", this.packetNumber);
    },
  },
  mounted() {
    this.CreateUdpSocket();

    // set up message handler
    UdpPlugin.addListener("receive", (data) => {
      this.InboundMessageHandler(data);
    });

    setInterval(this.ConnectionHandler, 10);
    // set up send sequence
  },
};
</script>
