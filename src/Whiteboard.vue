<script>

import note from './components/note.vue'


export default {
  components: {
    note
  },

  data() {
    return {
      index: 0,
      notes: {},                // data about "post-it" notes; key being the id of the note
      texts: {},                // list of texts in the notes; key being the id of the note
      arrows: {},               // arrows represented as an oriented graph; key being id from 
                                // which the arrow is pointing and the value an array of ids 
                                // to which it is pointing
      arrowFrom: null,          // id of the note where an arrow has started being dragged from
      arrowFromWhileOut: null,  // same as above, but here the value is stored if the mouse 
                                // leaves the whiteboard while still pressed down
      creating: false,          // true if creation mode is on
      deleting: false,          // true if deleting mode is on
      offsetX: 0,               // value of offset from point of origin of the note while dragged
      offsetY: 0,               // value of offset from point of origin of the note while dragged
      defaultSize: 150          // defalut note size
    }
  },

  computed: {
    edges() { // gives coordinates for arrows
      // computes the shortest possible arrow between the notes
      // chooses the best snap points

      // not as good a result as was hoped for while the notes are overlaping


      let distance = (ax, ay, bx, by) => { // compute euclidean distance of two points
        return Math.sqrt((ax - bx) ** 2 + (ay - by) ** 2);
      };
      let giveSnapPoints = (id, offset=0) => { // list coordinated of snap points
        let height = parseInt(this.notes[id].height.trim("px"));
        let width = parseInt(this.notes[id].width.trim("px"));
        return [[this.notes[id].x           - offset,     this.notes[id].y + height/2             ], 
                [this.notes[id].x + width/2,              this.notes[id].y            - offset    ],
                [this.notes[id].x + width/2,              this.notes[id].y + height   + 1.5*offset],
                [this.notes[id].x + width   + 1.5*offset, this.notes[id].y + height/2             ]];
      };

      var result = [];
      for (const pointingFrom in this.arrows) {
        for (const pointingTo of this.arrows[pointingFrom]) {
          var minDist = Infinity;
          var bestA = null;
          var bestB = null;
          for (const aPoint of giveSnapPoints(parseInt(pointingFrom))) {
            for (const bPoint of giveSnapPoints(parseInt(pointingTo), 10)) {
              var dist = distance(aPoint[0], aPoint[1], bPoint[0], bPoint[1])
              // console.log('evaluating distance of ' + aPoint[0], aPoint[1], bPoint[0], bPoint[1], dist);
              if (dist < minDist) {
                minDist = dist;
                bestA = aPoint;
                bestB = bPoint;
              };
            }
          };
          result.push([bestA[0], bestA[1], bestB[0], bestB[1]]);
        };
      };
      return result;
    }
  },

  methods: {
    onClick({ clientX: x, clientY: y }) { // create note
      if (this.creating) {
        this.notes[this.index++] = {x: x, 
                                    y: y,
                                    width: this.defaultSize + 'px', 
                                    height: this.defaultSize + 'px'};
        // console.log('note created at ' + x, y);
      };
    },

    updateText(msg) { // save updated text of a note
      const {id, text} = msg;
      this.texts[id] = text;
    },

    updateNoteWidth(msg) { // save width if a note has been resized
      const {id, newWidth} = msg;
      this.notes[id].width = newWidth;
    },

    updateNoteHeight(msg) { // save height if a note has been resized
      const {id, newHeight} = msg;
      this.notes[id].height = newHeight;
    },

    onDrop() { // move note that has been dragged
      event.preventDefault();
      this.notes[event.dataTransfer.getData("text")].x = event.clientX - this.offsetX;
      this.notes[event.dataTransfer.getData("text")].y = event.clientY - this.offsetY;
    },

    arrowStart(id) { // save id of the note from which the arrow has been initiated
      // console.log('arrow start attempt by ' + id);
      this.arrowFrom = id;
    },

    arrowEnd(id) { // check if a dragged arrow has been correctly placed
      // if the arrow doesn't exist, creates it
      // if the arrow does exist, removes it

      // console.log('arrow attempt form ' + this.arrowFrom + ' to ' + id);
      if (this.arrowFrom != null && id != this.arrowFrom) {
        if (this.arrowFrom in this.arrows) {
          if (this.arrows[this.arrowFrom].includes(id)) {
            this.arrows[this.arrowFrom].pop(id);
            // console.log('arrow sucessfully removed');
          } else {
            this.arrows[this.arrowFrom].push(id);
            // console.log('arrow sucessfully added');
          };
        } else {
          this.arrows[this.arrowFrom] = [id];
          // console.log('arrow sucessfully added');
        };
      };
    },

    onMouseLeave() { // save id of a note that initiated an arrow when the mouse moved away from the whiteboard
      if (this.arrowFrom != null) {
        this.arrowFromWhileOut = this.arrowFrom;
        this.arrowFrom = null;
      };
    },

    onMouseEnter() { // if the mouse has returned while still being pressed, resume as if nothing happened
      if (this.arrowFromWhileOut != null && event.buttons == 2) {
        this.arrowFrom = this.arrowFromWhileOut;
        this.arrowFromWhileOut = null;
      };
    },

    movingMouse() { // not finished; intended for moving a arrow that is being created
      if (this.arrowFrom != null && event.buttons == 2) {
        // console.log(event.clientX, event.clientY); 
        // control a special arrow and move so that it points where the mouse is pointing
      };
    },

    deleteNote(id) { // delete a note
      // doesn't work as intended
      // BUG !!!!!
      //   when a note is deleted the contents migrate to the note that has been creaded after the deleted one
      //   a fix could be iterating through texts object (it still contains correct contents) and replace the contents manually, 
      //   but that is not addressing the cause
      //   I'm not entirelly sure about the casuse, but I suspect the internal workings of the browser for handling forms 


      this.deleting = false; // after each deletion the deleting mode is switched off
      delete this.notes[id];
      delete this.arrows[id];
      delete this.texts[id];
      for (const key in this.arrows) {
        this.arrows[key] = this.arrows[key].filter(element => element != id);
      };
    }
  }
}
</script>



<template>
<head>
  <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined:opsz,wght,FILL,GRAD@48,400,0,0" />
</head>
  <svg>
    <defs>
      <marker id="arrowhead" markerWidth="10" markerHeight="7" 
              refX="0" refY="3.5" orient="auto">
        <polygon points="0 0, 10 3.5, 0 7" />
      </marker>
    </defs>
    <line v-for="(coords, id) in edges" 
            :x1="coords[0]" :y1="coords[1]" 
            :x2="coords[2]" :y2="coords[3]" 
            stroke="#000" stroke-width="1" 
            marker-end="url(#arrowhead)" />    
  </svg>
  <div id="whiteboard"
       @click="onClick" 
       @mouseup.right="this.arrowFrom = null"
       @mouseleave="onMouseLeave"
       @mouseenter="onMouseEnter"
       @mousemove="movingMouse"
       @drop="onDrop"
       @dragover.prevent
       @dragenter.prevent>
    <note v-for="(noteData, id) in notes"
            :x="noteData.x" 
            :y="noteData.y" 
            :id="id"
            :height="noteData.height"
            :width="noteData.width"
            :deleting="deleting"
            @myText="msg => updateText(msg)"  
            @offsetX="pos => this.offsetX = pos"
            @offsetY="pos => this.offsetY = pos" 
            @newWidth="msg => updateNoteWidth(msg)"
            @newHeight="msg => updateNoteHeight(msg)"
            @arrowStart="id => arrowStart(id)"
            @arrowEnd="id => arrowEnd(id)"
            @deleteMe="id => deleteNote(id)"
    />  
  </div>
  <div class="controls">
    <button @click="creating = !creating">
      <span class="material-symbols-outlined">
        {{creating ? 'disabled_by_default' : 'add_box' }}
      </span>
    </button>
    <button @click="deleting = !deleting; creating = false">
      <span class="material-symbols-outlined" 
            :style="{color: (deleting ? 'purple' : 'black')}">
        delete
      </span>
    </button>
  </div>
</template>

<style>
  body {
    margin: 0;
    padding: 0;
  }

  #app {
    margin: 0;
    padding: 0 !important;
  }

  svg {
    width: 100%;
    height: 100%;
    position: absolute;
    background-color: #0000;
  }

  .controls {
    position: fixed;
    top: 10px;
    left: 0;
    right: 0;
    text-align: center;
  }

  .controls button + button {
    margin-left: 6px;
  }

  html, body, #app, #whiteboard {
    height: 100%;
  }

  #whiteboard {
    position: relative;
  }
</style>



<!-- 

<template>
  <header>
    <div class="wrapper">
      Help please!
    </div>
  </header>

  <main>
  </main>
</template> -->