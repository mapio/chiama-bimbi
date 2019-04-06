<template>
  <div class="callme">
    <textarea v-model="names">
    </textarea>
    <button @click="doStart">Inizia!</button>
    <Container group-name="names" @drop="rap => onDrop(0, rap)" :get-child-payload="itemIndex => getChildPayload(0, itemIndex)">
        <Draggable v-for="item in toBeCalled" :key="item.id"><span class="name">{{item.name}}</span></Draggable>
    </Container>
    <button @click="doCall">Chiama!</button>
    <Container group-name="names" @drop="rap => onDrop(1, rap)" :get-child-payload="itemIndex => getChildPayload(1, itemIndex)">
        <Draggable v-for="item in called" :key="item.id"><span class="name">{{item.name}}</span></Draggable>
    </Container>
  </div>
</template>

<script>
import {Container, Draggable} from 'vue-smooth-dnd'

export default {
  components: {Container, Draggable},
  name: 'CallMe',
  data() {return {
    names: 'uno\ndue\ntre\nquattto\ncinque',
    toBeCalled: [],
    called: []
  }},
  methods: {
    doCall() {
      if (this.toBeCalled.length > 0) {
        const toCall = this.toBeCalled.pop()
        this.called.push(toCall)
      }
    },
    doStart() {
      this.toBeCalled = []
      this.names.split(/\r?\n/).forEach((item, idx) => {
        if (item) this.toBeCalled.push({id: idx, name: item})
      })
      this.called = []
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
      // console.log(idx, removedIndex, addedIndex, payload.id)
    },
    getChildPayload(groupIndex, itemIndex) {
      console .log('get-child-payload', groupIndex, itemIndex)
      if (groupIndex == 0)
        return this.toBeCalled[itemIndex]
      else
        return this.called[itemIndex]
    },
  }
}
</script>

<style scoped>
  span.name {padding: 1em; border: 1pt dotted black;}
</style>
