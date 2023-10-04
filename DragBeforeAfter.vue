<template>
   <div class="images" ondragstart="return false;" ondrop="return false;">
      <div class="wrapper" ref="wrapper">
         <div class="before" :class="{animation: startAnimation}" ref="before">
            <img :src="beforePhoto" ref="beforeImg" draggable="false"/>
            <span class="images__label">Было</span>
         </div>
         <div class="after">
            <img :src="afterPhoto" draggable="false"/>
            <span class="images__label images__label_after">Стало</span>
         </div>
         <div class="scroller" :class="{animation: startAnimation}" ref="scroller">
            <q-img src="icons/actions/drag.svg" style="width: 20px"></q-img>
         </div>
      </div>
      <button v-if="hasPopup" type="button" class="images__full" @click="togglePopup">
         <q-img src="icons/actions/full.svg" style="fill:#676f73;color:#676f73"></q-img>
      </button>
      <q-dialog v-model="showPopup">

         <div class="popup__content">
            <DragBeforeAfter :beforePhoto="beforePhoto" :afterPhoto="afterPhoto" :startAnimation="true"/>
         </div>
      </q-dialog>
   </div>
</template>

<script>

    export default {
        name: 'DragBeforeAfter',
        props: {
            beforePhoto: {
                type: String,
                required: true
            },
            afterPhoto: {
                type: String,
                required: true
            },
            hasPopup: {
                type: Boolean,
                default: false
            },
            startAnimation: {
                type: Boolean,
                default: false
            }
        },
        data: () => ({
            active: false,
            scroller: null,
            wrapper: null,
            before: null,
            showPopup: false
        }),
        methods: {
            scrollIt(x) {
                const transform = Math.max(20, (Math.min(x, this.wrapper.offsetWidth - 20)));
                this.before.style.width = transform + 'px';
                this.scroller.style.left = transform + 'px';
            },
            mouseDown() {
                this.active = true;
            },
            mouseUp() {
                this.active = false;
            },
            mouseMove(e) {
                if (!this.active) return;
                let x = e.pageX;
                x -= this.wrapper.getBoundingClientRect().left;
                this.scrollIt(x);
            },
            touchStart() {
                this.active = true;
            },
            touchEnd() {
                this.active = false;
            },
            init() {
                this.$refs.beforeImg.style.width = this.wrapper.clientWidth + 'px';
            },
            onResize() {
                this.init();
                this.scrollIt(this.wrapper.offsetWidth / 2);
            },
            togglePopup() {
                this.showPopup = !this.showPopup;
            },
            closePopup() {
                //this.$emit('closePopup', this.$vnode.key?.slice(1) ?? null);
                this.togglePopup();
            }
        },
        mounted() {
            this.scroller = this.$refs.scroller;
            this.wrapper = this.$refs.wrapper;
            this.before = this.$refs.before;

            this.init();
            window.addEventListener('resize', this.onResize);

            this.scroller.addEventListener('mousedown', this.mouseDown);
            document.body.addEventListener('mouseup', this.mouseUp);
            document.body.addEventListener('mouseleave', this.mouseUp);
            document.body.addEventListener('mousemove', this.mouseMove);

            this.scroller.addEventListener('touchstart', this.touchStart);
            document.body.addEventListener('touchend', this.touchEnd);
            document.body.addEventListener('touchcancel', this.touchEnd);
        },
        beforeUnmount() {
            window.removeEventListener('resize', this.onResize);

            this.scroller.removeEventListener('mousedown', this.mouseDown);
            document.body.removeEventListener('mouseup', this.mouseUp);
            document.body.removeEventListener('mouseleave', this.mouseUp);
            document.body.removeEventListener('mousemove', this.mouseMove);

            this.scroller.removeEventListener('touchstart', this.touchStart);
            document.body.removeEventListener('touchend', this.touchEnd);
            document.body.removeEventListener('touchcancel', this.touchEnd);
        }
    };
</script>

<style scoped lang="scss">

   .images {
      width: 100%;
      height: 100%;
      position: relative;
      user-select: none;
      &__full {
         position: absolute;
         background: #3AEDE7;
         right: 1rem;
         top: 1rem;
         width: 2.125rem;
         height: 2.125rem;
         z-index: 5;
         padding: 0.25rem;
         transition: 0.5s;
         opacity: 0;
         border-radius: 50%;
         border: none;
         outline: none;
      }
      &__label {
         position: absolute;
         background: #3AEDE7;
         left: 1rem;
         bottom: 1rem;
         z-index: 2;
         padding: 0 0.25rem;
         font-size: 0.875rem;
         font-weight: bold;
         text-transform: uppercase;
         transition: 0.5s;
         opacity: 0;
         &_after {
            left: auto;
            right: 1rem;
         }
      }
      &:hover {
         .images__label, .images__full {
            opacity: 1;
         }
      }
   }
   .wrapper {
      width: 100%;
      height: 100%;
      position: absolute;
      left:50%;
      top:50%;
      transform:translate3d(-50%,-50%,0);
      overflow:hidden;
      pointer-events:none;
      & .scroller {
         width: 40px;
         height: 40px;
         position: absolute;
         left:50%;
         top:50%;
         transform:translate(-50%, -50%);
         border-radius:50%;
         pointer-events:auto;
         cursor: pointer;
         z-index: 5;
         background: #3AEDE7;
         display: flex;
         justify-content: center;
         align-items: center;
         &:before, &:after {
            content:" ";
            display: block;
            width: 1px;
            height: 9999px;
            position: absolute;
            left: 50%;
            z-index: 30;
            transition:0.1s;
            background: #3AEDE7;
         }
         &:before {
            top:100%;
         }
         &:after {
            bottom:100%;
         }
      }
      & .before, .after {
         width:100%;
         height:100%;
         background-repeat:no-repeat;
         background-color: white;
         background-size: cover;
         background-position: center;
         position: absolute;
         top:0;
         left:0;
         pointer-events:none;
         overflow: hidden;
         z-index: 1;
         & img {
            width: 100%;
            height: 100%;
            object-fit: cover;
         }
      }
      & .before{
         width:50%;
         z-index: 2;
         & img {
            width: 200%
         }
      }
      & .animation {
         &.before {
            width: 20px;
            animation: animBefore 5s ease-in-out backwards;
         }
         &.scroller {
            left: 20px;
            animation: animScroller 5s ease-in-out backwards;
         }
      }
      @keyframes animBefore {
         0% {
            width: calc(20px);
         }
         50% {
            width: calc(100% - 20px);
         }
         100% {
            width: calc(20px);
         }
      }
      @keyframes animScroller {
         0% {
            left: calc(20px);
         }
         50% {
            left: calc(100% - 20px);
         }
         100% {
            left: calc(20px);
         }
      }
   }

   .popup {
      position: fixed;
      z-index: 20;
      left: 0;
      top: 0;
      width: 100%;
      height: 100%;
      display: flex;
      justify-content: center;
      align-items: center;
      padding: 0;
      background: none;
      &__bg {
         position: absolute;
         width: 100%;
         height: 100%;
         border: none;
      }
      &__content {
         z-index: 21;
         width: 70vw;
         max-width: 70vw;
         height: 70vh;
         padding: 0.375rem;
         background: #FFFFFF;
      }
   }
</style>
