<template>
  <div>
    <GlobalEvents
      @mousemove="mouseMove"
      @mouseover="mouseOverElement"
      @mouseout="mouseOut"
      @mousedown="reduceSize"
      @mouseup="upSize"
    />

    <div id="cool-cursor" v-bind:style="{ left:xPos, top:yPos }">
      <div v-if="!hover" id="cool-circle"
           v-bind:style="{ height:currentHeight, width:currentWidth,'border-radius':bRadius, transform: 'rotate('+angle+')', borderColor: color }">
        <div id="pointer" v-bind:style="{color: pointerColor}"></div>
      </div>
      <div v-else="!hover" id="cool-circle"
           v-bind:style="{ height:currentHeight, width:currentWidth,'border-radius':bRadius, borderColor: hoverColor }">
        <div id="pointer" v-bind:style="{color: pointerColor}"></div>
      </div>
    </div>

  </div>
</template>

<script>
  import GlobalEvents from 'vue-global-events';

  export default {
    name: 'CoolCursor',
    components: {
      GlobalEvents
    },
    props: {
      color: {
        type: String,
        required: false,
        default: "black"
      },
      defaultHeight: {
        required: false,
        default: 30
      },
      defaultWidth: {
        required: false,
        default: 30
      },
      minHeight: {
        required: false,
        default: '22px'
      },
      minWidth: {
        required: false,
        default: '22px'
      },
      initialX: {
        required: false,
        default: 0
      },
      initialY: {
        required: false,
        default: 0
      },
      defaultRadius: {
        required: false,
        default: '50%'
      },
      pointerColor: {
        required: false,
        default: 'green'
      },
      hoverColor: {
        required: false,
        default: 'yellow'
      },

    },

    data: () => {
      return {
        // USED VALUES
        bRadius: '50%',
        angle: '0turn',
        currentHeight: "30",
        currentWidth: "30",
        hightValue: 30,
        widthValue: 30,
        // POSITION
        dx: 1,
        dy: 1,
        xPos: 0,
        yPos:0,
        lastX: 0,
        lastY: 0,
        // COMPUTED
        cX: 0,
        cY: 0,
        // CURSOR STATE
        hover: false,
        clicked: false
      };
    },

    mounted() {
      //* Init values
      this.currentHeight = this.defaultHeight;
      this.currentWidth = this.defaultWidth;
      this.hightValue = this.defaultHeight;
      this.widthValue = this.defaultWidth;
      this.bRadius = this.defaultRadius;
      this.xPos = this.initialX;
      this.yPos = this.initialY;
    },

    methods: {
      mouseMove: function (event) {
        if (!this.hover) {
          this.xPos = event.clientX + "px";
          this.yPos = event.clientY + "px";
          this.cX = event.clientX;
          this.cY = event.clientY;
          if (!this.hover) {

            let rawDeplacementX = this.cX - this.lastX;
            let deplacementX = Math.abs(rawDeplacementX);

            let rawDeplacementY = this.cY - this.lastY;
            let deplacementY = Math.abs(rawDeplacementY);

            let deplacementTotal = Math.abs(deplacementX + deplacementY);
            let deplacementDiago = (deplacementX * 100) / deplacementTotal;

            if (deplacementDiago > 35 && deplacementDiago < 80) {
              if (deplacementX > deplacementY) {
                this.dx = deplacementX / 16;
              } else {
                this.dy = deplacementX / 16;
              }

              this.dy = 1;
              if (rawDeplacementX < 0) { // left
                if (rawDeplacementY > 0) { // down
                  this.angle = "-0.65turn";
                } else { // up
                  this.angle = "0.65turn";
                }
              } else { // right
                if (rawDeplacementY > 0) { // down
                  this.angle = "0.65turn";
                } else { // up
                  this.angle = "-0.65turn";
                }
              }


            } else {
              this.angle = '0turn';
              this.dx = deplacementX / 16;
              this.dy = deplacementY / 16;

            }


            if (!this.clicked) {
              if (this.dx < 1) this.dx += 1;
              if (this.dy < 1) this.dy += 1;
              if (this.dx > 3) this.dx = 2.5;
              if (this.dy > 3) this.dy = 2.5;
              this.currentHeight = (this.hightValue * this.dy) + "px";
              this.currentWidth = (this.widthValue * this.dx) + "px";
            }
            this.lastX = this.cX;
            this.lastY = this.cY;
          }


        }


      },
      mouseOverElement: function (event) {
        if (event.target.classList.contains("selectionnable")) {
          // select current element
          let elem = event.target;
          // notify hover
          this.hover = true;
          // change borderRadius to have the same as the element
          this.bRadius = getComputedStyle(elem).borderRadius;

          // Get bounding rect
          let rect = elem.getBoundingClientRect();
          // set new pos to fit the center of the current element
          this.xPos = (rect.right - (elem.offsetWidth / 2)) + "px";
          this.yPos = (rect.top + (elem.offsetHeight / 2)) + "px";
          //elem.style.transition = ''
          // fit the size of the current element
          this.currentHeight = (elem.offsetHeight + 20) + "px";
          this.currentWidth = (elem.offsetWidth + 20) + "px";


        }
      },
      mouseOut: function (event) {
        this.hover = false;
        this.bRadius = this.defaultRadius;
        this.currentHeight = this.defaultHeight + "px";
        this.currentWidth = this.defaultWidth + "px";
      },
      reduceSize: function (event) {
        if (!this.hover) {
          this.clicked = true;
          this.currentHeight = this.minHeight;
          this.currentWidth = this.minWidth;
        }
      },
      upSize: function (event) {
        if (!this.hover) {
          this.clicked = false;
          this.currentHeight = this.defaultHeight + "px";
          this.currentWidth = this.defaultWidth + "px";
        }

      }
    }
  }
  ;
</script>

<style>

  #cool-cursor {
    position: absolute;
    z-index: -1;

  }

  #pointer {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-60%, -60%);


  }

  .selectionnable:hover {
    cursor: pointer;
  }

  #pointer:before {
    content: ' \25CF';
    font-size: 10px;


  }

  #cool-circle {
    position: relative;
    border-style: solid;
    border-radius: 50%;
    transition: all 0.4s ease;
    transform: translate(-50%, -50%);
  }
  * {
    cursor: none;
    margin: 0;
  }


</style>
