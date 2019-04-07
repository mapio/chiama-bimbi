<template>
  <v-tabs
    show-arrows
    centered
    color="cyan"
    dark
    icons-and-text
    @change="doTab"
  >
    <v-tabs-slider color="yellow"></v-tabs-slider>

    <v-tab href="#inserisci">
      Inserisci i nomi <v-icon>group_add</v-icon>
    </v-tab>

    <v-tab href="#chiama">
      Chiama! <v-icon>record_voice_over</v-icon>
    </v-tab>

    <v-tab href="#info">
      Informazioni <v-icon>info</v-icon>
    </v-tab>

    <v-tab-item value="inserisci">
      <v-container>
        <v-textarea v-model="names" placeholder="Scrivi qui i nomi dei bambini, uno per linea" autofocus clearable auto-grow></v-textarea>
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
        <v-btn @click="doCall" :disabled="disabled" alig>Chiama!</v-btn>
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
  data() {return {
    names: 'Aldo\nGiovanni\nGiacomo',
    toBeCalled: [],
    called: [],
    voices: [],
    disabled: false
  }},
  methods: {
    doTab(name) {
      if (name == 'chiama') {
        this.toBeCalled = []
        this.names.split(/\r?\n/).forEach((item, idx) => {
          if (item) this.toBeCalled.push({id: idx, name: item})
        })
        this.called = []
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
      this.disabled = this.toBeCalled.length == 0
    },
    getChildPayload(groupIndex, itemIndex) {
      return groupIndex == 0 ? this.toBeCalled[itemIndex] : this.called[itemIndex]
    },
  }
}
</script>

<style scoped>
  span.name {padding: 1em; border: 1pt dotted black;}
</style>
