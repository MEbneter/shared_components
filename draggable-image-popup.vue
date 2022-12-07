<template>
  <DxPopup
    :dragEnabled="false"
    :visible="true"
    :width="'98%'"
    :height="'80%'"
    :closeOnOutsideClick="true"
    :showCloseButton="false"
    @hidden="closePopup"
    @showing="openPopup"
    @shown="backgroundUpdates"
  >
    <DxToolbarItem
      widget="dxButton"
      :options="{
        text: this.buttonText,
        icon: 'icons/arrow_back.svg',
        onClick: clickButton
      }"
      location="center">
    </DxToolbarItem>
    <img
      :src="usedsrc"
      :class="'popupImage undraggable'"
      @mouseup="this.changeView(false)"
      @dragstart="fireFoxDraggableCancel"
      />
  </DxPopup>
</template>

<script>

import DxPopup, { DxToolbarItem } from 'devextreme-vue/popup'

export default {
  components: {
    DxPopup,
    DxToolbarItem
  },

  emits: ['hidden'],

  props: {
    imagesrc: {
      type: String,
      default: null
    }
  },
  methods: {
    // prevents image from being draggable in firefox browser ...
    fireFoxDraggableCancel (e) {
      e.preventDefault()
    },
    closePopup () {
      // remove all event listeners
      document.removeEventListener('pointerdown', this.mouseDown)
      document.removeEventListener('pointerup', this.mouseUp)
      document.removeEventListener('pointermove', this.mouseMove)
      this.changeView(true)
      this.$emit('hidden')
    },
    backgroundUpdates () {
      this.popupWindowWidth = document.getElementsByClassName('dx-popup-content')[0].offsetWidth
      this.popupWindowHeight = document.getElementsByClassName('dx-popup-content')[0].offsetHeight
      this.compareImageSize()
      this.checksrc()
    },
    openPopup () {
      // get image data
      const imageData = new Image()
      imageData.src = this.imagesrc
      this.imageWidth = imageData.width
      this.imageHeight = imageData.height
      // get popup window data
      this.popupWindowWidth = document.getElementsByClassName('dx-popup-content')[0].offsetWidth
      this.popupWindowHeight = document.getElementsByClassName('dx-popup-content')[0].offsetHeight

      this.compareImageSize()
      this.checksrc()

      // set all eventListeners
      document.getElementsByClassName('popupImage')[0].setAttribute('style', 'object-position:' + Number(this.offsetX) + 'px ' + Number(this.offsetY) + 'px;')

      document.addEventListener('pointerdown', this.mouseDown)
      document.addEventListener('pointerup', this.mouseUp)
      document.addEventListener('pointermove', this.mouseMove)
      // starts the fullView of with true
      this.changeView(true)
    },
    mouseDown (e) {
      if (this.fullView === false) {
        this.mousePressed = true
        this.lastMX = e.clientX
        this.lastMY = e.clientY
      }
    },
    mouseUp (e) {
      if (this.fullView === false) {
        this.mousePressed = false
        this.offsetX += this.lastEventPosX
        this.offsetY += this.lastEventPosY
      }
    },
    mouseMove (e) {
      if (this.fullView === false) {
        if (this.mousePressed === true) {
          this.updateStyle(document.getElementsByClassName('popupImage')[0], e)
        }
      }
    },
    // updates the content if image is being dragged
    updateStyle (elementImage, event) {
      this.checkOutOfBounds(event)
      if (elementImage !== undefined) {
        elementImage.setAttribute(
          'style',
          'object-position:' + String(Number(this.offsetX) + Number(this.getDiffX(event))) + 'px ' +
                               String(Number(this.offsetY) + Number(this.getDiffY(event))) + 'px;'
        )
      }
    },
    // gets the difference between where the user clicked and how far he dragged it
    getDiffX (event) {
      this.lastEventPosX = event.clientX - this.lastMX
      return this.lastEventPosX
    },
    getDiffY (event) {
      this.lastEventPosY = event.clientY - this.lastMY
      return this.lastEventPosY
    },
    // checks if the user drags the image too far out
    checkOutOfBounds (event) {
      // "border" on the left
      if (this.offsetX + this.getDiffX(event) > 0) {
        this.lastMX += this.offsetX + this.getDiffX(event)
      }
      // upper "border"
      if (this.offsetY + this.getDiffY(event) > 0) {
        this.lastMY += this.offsetY + this.getDiffY(event)
      }
      // "border" on the right
      if (this.offsetX + this.getDiffX(event) < -this.imageWidth + this.popupWindowWidth) {
        this.lastMX += ((this.offsetX + this.getDiffX(event)) - (-this.imageWidth + this.popupWindowWidth))
      }
      // bottom "border"
      if (this.offsetY + this.getDiffY(event) < -this.imageHeight + this.popupWindowHeight) {
        this.lastMY += ((this.offsetY + this.getDiffY(event)) - (-this.imageHeight + this.popupWindowHeight))
      }
    },
    // checks if the newly opened image is another one as the one that is already stored
    checksrc () {
      if (this.usedsrc === this.imagesrc) {
        this.usedsrc = this.imagesrc
      } else {
        this.usedsrc = this.imagesrc
        this.offsetX = 0
        this.offsetY = 0
      }
    },
    // compares the image size to the window size
    // if the window is larger than the image => permanently out of bounds
    // so if this is the case => it sets the image size to the size of the window
    compareImageSize () {
      if (this.popupWindowWidth >= this.imageWidth) {
        this.imageWidth = this.popupWindowWidth
      }
      if (this.popupWindowHeight >= this.imageHeight) {
        this.imageHeight = this.popupWindowHeight
      }
    },
    changeView (newMode) {
      const popupImage = document.getElementsByClassName('popupImage')[0]
      if (newMode === false) {
        if (popupImage.classList.contains('originalSize')) {
          popupImage.classList.remove('originalSize')
        }
        popupImage.classList.add('zoomedSize')
        this.buttonText = this.$t('imagepopup.fullview')
      } else {
        if (popupImage.classList.contains('zoomedSize')) {
          popupImage.classList.remove('zoomedSize')
        }
        popupImage.classList.add('originalSize')
        popupImage.setAttribute('style', '')
        this.offsetX = 0
        this.offsetY = 0
        this.lastEventPosX = 0
        this.lastEventPosY = 0
        this.buttonText = this.$t('imagepopup.back')
      }
      this.fullView = newMode
    },
    clickButton () {
      if (this.fullView === false) {
        this.changeView(true)
      } else {
        this.closePopup()
      }
    }
  },

  data () {
    return {
      mousePressed: false,

      // if fullView is true, the whole image is shown in popup
      // else => image is shown in its original size ("zoomed in") (also in popup)
      fullView: true,

      // i18n test displayed in title-button
      buttonText: '',

      // the actual used src is in here so it can be compared with imagesrc
      usedsrc: '',

      // offset from upper left corner of picture (basically position of image)
      offsetX: 0,
      offsetY: 0,

      // where on the image did the user click => image is moved by difference of {lastMX/lastMY} and current mouse pos (event.clientX/event.clientY)
      lastMX: 0,
      lastMY: 0,

      // lastEventPosX is always updated after mousemove / touchmove event
      lastEventPosX: 0,
      lastEventPosY: 0,

      // image size
      imageWidth: 0,
      imageHeight: 0,

      // popup window height
      popupWindowWidth: 0,
      popupWindowHeight: 0
    }
  }
}
</script>

<style>

  .dx-popup-content {
    background-color: #DDDDDD;
  }

  .originalSize {
    cursor: pointer;
    object-fit: cover;
    object-position: 0px 0px;
    width: 107%;
    height: auto;
  }

  .zoomedSize {
    cursor: grab;
    overflow: hidden;
    object-fit: none;
    background-size: cover;
  }

  .popupImage {
    margin-left: -20px;
    margin-top: -20px;
    margin-right: 20px;
  }

  .undraggable {
    -webkit-user-drag: none;
    user-select: none;
    -webkit-touch-callout: none;
    -webkit-user-select: none;
    -khtml-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    -moz-window-dragging: none;
    -moz-user-input: none;
  }

  .dx-widget {
    margin-top: 2px;
    margin-bottom: 15px;
  }

</style>
