<template>
  <ion-list>

    <ion-item>
      <ion-range min="-200" max="200" color="danger" pin="true">
        <ion-label slot="start">-200</ion-label>
        <ion-label slot="end">200</ion-label>
      </ion-range>
    </ion-item>

  </ion-list>
</template>

<script>
import { IonItem, IonLabel, IonList, IonRange } from '@ionic/vue';
import { defineComponent } from 'vue';
import { addIcons } from 'ionicons';
import { sunny} from 'ionicons/icons';

export default defineComponent({
  components: {  IonItem, IonLabel, IonList, IonRange },
  mounted() {
    // Sets initial value for dual-knob ion-range



    addIcons({
      sunny
    });
    
  }
});
</script>