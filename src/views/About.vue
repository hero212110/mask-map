<template>
  <v-card id="create">
    <v-container fluid>
      <v-row>
        <v-col cols="12" sm="6" md="4">
          <v-subheader>Options</v-subheader>
        </v-col>
      
      </v-row>
    </v-container>
    <v-speed-dial
      v-model="fab"
      right
    >
      <template v-slot:activator>
        <v-btn v-model="fab" color="blue darken-2" dark fab>
          <v-icon v-if="fab">mdi-close</v-icon>
          <v-icon v-else>mdi-account-circle</v-icon>
        </v-btn>
      </template>
      <v-btn fab dark small color="green">
        <v-icon>mdi-pencil</v-icon>
      </v-btn>
      <v-btn fab dark small color="indigo">
        <v-icon>mdi-plus</v-icon>
      </v-btn>
      <v-btn fab dark small color="red">
        <v-icon>mdi-delete</v-icon>
      </v-btn>
    </v-speed-dial>
  </v-card>
</template>

<script>
export default {
  data: () => ({
    fab: false,
  }),
};
</script>

<style>
#create .v-speed-dial {
  position: absolute;
}
#create .v-btn--floating {
  position: relative;
}
</style>
