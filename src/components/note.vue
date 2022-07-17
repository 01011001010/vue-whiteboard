<script>

export default {
  props: {
    id: String,
    x: Number,
    y: Number,
    width: String,
    height: String,
    deleting: Boolean
  },

  emits: ['myText', 'offsetX', 'offsetY', 
          'newHeight', 'newWidth', 'arrowStart', 
          'arrowEnd', 'deleteMe'],

  data() {
    return {
      noteText: '',
      absolute: 'absolute'
    }
  },

  methods: {
      onMouseUp() { // for resizing, relay the info about the new size to the whiteboard
        this.$emit('newWidth', {id: this.id, 
                                newWidth: document.getElementById(this.id).style.getPropertyValue("width")});
        this.$emit('newHeight', {id: this.id, 
                                 newHeight: document.getElementById(this.id).style.getPropertyValue("height")});
      },
      
      onDrag(event) { // start moving the note
        this.$emit('offsetX', event.clientX - this.x);
        this.$emit('offsetY', event.clientY - this.y);
        event.dataTransfer.setData("text", event.target.id);
      },

      startArrow() { // initiate an arrow
        // console.log(this.id + ' started an arrow');
        this.$emit('arrowStart', this.id);
      },

      endArrow() { // end an arrow
        // console.log(this.id + ' ended an arrow');
        this.$emit('arrowEnd', this.id);
      },

      onClick({ clientX: x, clientY: y }) { // check if the delete mode is on and if yes, then ask the whiteboard to delete this note
        if (this.deleting) {
          this.$emit('deleteMe', this.id);
        }
      }
    },

    watch: {
      noteText(newText, oldText) { // observe input and relay change to the whiteboard
        // console.log('text changed to ' + newText);
        this.$emit('myText', {id: this.id, text: newText});
      }
    }
}
</script>

<template>
    <textarea @mouseup.right="endArrow"
              @mouseup.left="onMouseUp" 
              @mousedown.right="startArrow"
              v-model="noteText" 
              :style="{left: x + 'px', top: y + 'px', 
                       position: absolute, width: width, 
                       height: height}" 
              :id="id"
              @click="onClick" 
              draggable="true" 
              @dragstart='onDrag($event)'>
    </textarea>
</template>

<style>
  textarea {
    background-color: #f0ff4a;
    min-height: 110px;
    min-width: 110px;
  }
</style>