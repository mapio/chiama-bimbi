<template>
  <v-app id="callme">
    <v-content>
      <v-container fill-height>
        <v-layout>
          <v-flex>
            <v-card class="elevation-8" style="height: 100%">
              <CallMe/>
            </v-card>
          </v-flex>
        </v-layout>
      </v-container>
    </v-content>
  </v-app>
</template>

<script>
import CallMe from './components/CallMe'

export default {
  name: 'App',
  components: { CallMe },
  data() { return {}}
}
</script>
