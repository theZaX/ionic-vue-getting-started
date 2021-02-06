<template>
  <!-- Default -->

  <!-- Colors -->
  <ion-button  expand="block" color="secondary">Connect</ion-button>


</template>

<script>
import { IonButton } from "@ionic/vue";
import { defineComponent } from "vue";

import { addIcons } from "ionicons";
import { star } from "ionicons/icons";

export default defineComponent({
  components: { IonButton },

  mounted() {
    addIcons({
      star,
    });
  },
});
</script>