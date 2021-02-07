<template>
  <base-layout page-title="Plane Control Panel">
    <ion-content>
      <ion-grid>
        <ion-row>
          <ion-col size="12">
            <ion-button
              expand="block"
              color="secondary"
              v-if="!isConnected"
              @click="OpenWebDealio"
              >Connect</ion-button
            >
            <ion-button
              expand="block"
              color="danger"
              v-else
              @click="CloseWebDealio"
              >Disconnect</ion-button
            >
          </ion-col>
        </ion-row>

        <ion-row class="ion-text-center ion-no-padding">
          <ion-col>
            <h4 class="ion-padding-horizontal">{{ webSocketStatus }}</h4>
          </ion-col>
        </ion-row>
        <ion-row>
          <ion-col>
            <p class="ion-float-left">battery status</p>
            <p class="ion-float-right">signal status</p>
          </ion-col>
        </ion-row>

        <ion-row>
          <ion-col>
            <ion-item>
              <ion-range v-model="sliderValue" @ionchange="YeetSliderInfo" min="-200" max="200" color="danger" pin="true">
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
            <ion-button @click="SendMessage" expand="block">Send</ion-button>
          </ion-col>
        </ion-row>
      </ion-grid>

      <ion-list lines="none">
        <ion-card v-for="message in connectionHistory" :key="message.body"
          ><ion-item
            >{{ message.sender }}: {{ message.body }}</ion-item
          ></ion-card
        >
      </ion-list>
    </ion-content>
  </base-layout>
</template>

<script>
import { add } from "ionicons/icons";

// import RangeTest from "../components/control/range-test.vue";
// import ButtonTest from "../components/control/button-test.vue";
// import ScrollControl from "../components/control/ScrollControl.vue";

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
  IonList,
  IonCard,
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
    IonList,
    IonCard,
  },
  data() {
    return {
      add,
      isConnected: false,
      connectionHistory: [],
      sendBarInfo: "faf",
      planeSocket: null,
      sliderValue: 40
    };
  },
  computed: {
    webSocketStatus() {
      if (this.isConnected) {
        return "Connected";
      }
      return "Not Connected";
    },
  },
  methods: {
    OpenWebDealio() {
      if (this.planeSocket === null || this.planeSocket.readystate !== 1) {
        this.LogMessage("system", "trying to connect via websockets");
        this.planeSocket = new WebSocket("ws://192.168.4.1");
        this.planeSocket.onopen = () => {
          this.isConnected = true;
        };
        this.planeSocket.onclose = () => {
          this.isConnected = false;
        };

        this.planeSocket.onmessage = (event) => {
          this.LogMessage("plane", event.data);
        };
      }
    },
    SendMessage() {
      const messageInfo = {
        body: this.sendBarInfo,
        sender: "client",
      };

      if (this.planeSocket === null || this.planeSocket.readyState !== 1) {
        this.LogMessage("System", "send failed");
        return;
      }
      this.LogMessage("client", this.sendBarInfo);

      this.planeSocket.send(JSON.stringify(messageInfo));
      this.sendBarInfo = "";


    },
    YeetSliderInfo(){
      const messageInfo = {
        body: this.sliderValue,
        sender: "slider",
      };

      if (this.planeSocket === null || this.planeSocket.readyState !== 1) {
        this.LogMessage("System", "send failed");
        return;
      }
      this.LogMessage("slider", this.sliderValue);

      this.planeSocket.send(JSON.stringify(messageInfo));
      this.sendBarInfo = "";

    },

    LogMessage(sender, body) {
      const messageInfo = {
        body: body,
        sender: sender,
      };
      this.connectionHistory.unshift(messageInfo);
    },
    CloseWebDealio(){
      this.planeSocket.close()
    }
  },
};
</script>
