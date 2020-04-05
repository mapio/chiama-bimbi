<template>
  <v-tabs
    show-arrows
    centered
    background-color="cyan"
    dark
    icons-and-text
    v-model="curTab"
    @change="doTab"
  >
    <v-tabs-slider color="yellow"></v-tabs-slider>

    <v-tab href="#inserisci">
      Inserisci i nomi <v-icon>mdi-account-cog</v-icon>
    </v-tab>

    <v-tab href="#chiama">
      Chiama! <v-icon>mdi-account-voice</v-icon>
    </v-tab>

    <v-tab href="#info">
      Informazioni <v-icon>mdi-alert-circle</v-icon>
    </v-tab>

    <v-tab-item value="inserisci">
      <v-container>
        <v-textarea @change="handleNames" v-model="names" placeholder="Scrivi qui i nomi dei bambini, uno per linea" autofocus clearable auto-grow></v-textarea>
      </v-container>
    </v-tab-item>

    <v-tab-item value="chiama">
      <v-container>
      <v-layout justify-space-around align-space-around>
      <v-flex xs4>
      <v-card><v-card-title>Da chiamare</v-card-title><v-card-text>
        <Container group-name="names" @drop="rap => onDrop(0, rap)" :get-child-payload="itemIndex => getChildPayload(0, itemIndex)">
          <Draggable v-for="item in toBeCalled" :key="item.id"><v-chip color="green">{{item.name}}</v-chip></Draggable>
        </Container>
      </v-card-text></v-card>
      </v-flex>
      <v-flex xs2>
        <v-btn @click="doCall" color="info" block :disabled="disabled">Chiama</v-btn>
        <v-btn @click="handleNames" color="info" block outlined>Ricomincia</v-btn>
      </v-flex>
      <v-flex xs4>
      <v-card><v-card-title>Chiamati</v-card-title><v-card-text>
        <Container group-name="names" @drop="rap => onDrop(1, rap)" :get-child-payload="itemIndex => getChildPayload(1, itemIndex)">
          <Draggable v-for="item in called" :key="item.id"><v-chip color="red">{{item.name}}</v-chip></Draggable>
        </Container>
      </v-card-text></v-card>
      </v-flex>
      </v-layout>
      </v-container>
    </v-tab-item>

    <v-tab-item value="info">
      <v-container>
        Info…
      </v-container>
    </v-tab-item>

  </v-tabs>
</template>

<script>
import {Container, Draggable} from 'vue-smooth-dnd'

const getVoices = () => {
	return new Promise(resolve => {
		let voices = speechSynthesis.getVoices()
		if (voices.length) resolve(voices)
		speechSynthesis.onvoiceschanged = () => resolve(speechSynthesis.getVoices())
	})
}

function speak(voice, text, next) {
  window.speechSynthesis.cancel()
  let ssu = new SpeechSynthesisUtterance(text)
  ssu.voice = voice
  window.speechSynthesis.speak(ssu)
  function waitssu() {
    if (!(window.speechSynthesis.speaking || window.speechSynthesis.pending)) {
      next()
      return
    }
    window.setTimeout(waitssu, 200)
  }
  waitssu();
}

export default {
  components: {Container, Draggable},
  name: 'CallMe',
  created() {
    getVoices().then(found => {
      this.voices = found.filter(voice => voice.lang == 'it-IT')
    })
  },
  mounted() {
    if (localStorage.getItem('cb_satus')) {
       try {
        const status = JSON.parse(localStorage.getItem('cb_satus'))
        this.names = status.names
        this.toBeCalled = status.toBeCalled
        this.called = status.called
      } catch(e) {
        localStorage.removeItem('cb_satus')
      }
    }
    if (this.names == '') {
      this.names = 'Aldo\nGiovanni\nGiacomo'
      this.handleNames()
    }
  },
  data() {return {
    names: '',
    toBeCalled: [],
    called: [],
    voices: [],
    disabled: false,
    curTab: 'chiama'
  }},
  methods: {
    saveStatus() {
      const parsed = JSON.stringify({
        names: this.names,
        toBeCalled: this.toBeCalled,
        called: this.called
      })
      localStorage.setItem('cb_satus', parsed)
    },
    handleNames() {
      this.toBeCalled = []
      this.names.split(/\r?\n/).forEach((item, idx) => {
        if (item) this.toBeCalled.push({id: idx, name: item})
      })
      this.called = []
      this.saveStatus()
      this.disabled = this.toBeCalled.length == 0
    },
    doTab(name) {
      if (name == 'chiama') {
        this.disabled = this.toBeCalled.length == 0
      }
    },
    doCall() {
      this.disabled = true
      const n = this.toBeCalled.length
      if (n > 0) {
        const idx = Math.floor(Math.random() * n)
        const item = this.toBeCalled[idx]
        speak(this.voices[0], item.name, () => {
          this.called.push(item)
          this.toBeCalled.splice(idx, 1)
          this.saveStatus()
          if (this.toBeCalled.length > 0) this.disabled = false
        })
      }
    },
    onDrop(idx, {removedIndex, addedIndex, payload}) {
      if (removedIndex !== null) {
        const src = idx == 0 ? this.toBeCalled : this.called
        const res = src.filter(item => item.id != payload.id)
        if (idx == 0)
          this.toBeCalled = res
        else
          this.called = res
      }
      if (addedIndex !== null) {
        if (idx == 0)
          this.toBeCalled.splice(addedIndex, 0, payload)
        else
          this.called.splice(addedIndex, 0, payload)
      }
      this.saveStatus()
      this.disabled = this.toBeCalled.length == 0
    },
    getChildPayload(groupIndex, itemIndex) {
      return groupIndex == 0 ? this.toBeCalled[itemIndex] : this.called[itemIndex]
    },
  }
}
</script>

<style scoped>
</style>
